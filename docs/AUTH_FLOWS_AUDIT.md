# AMRIT Helpline 104 — Auth Flows Audit

**Author:** Aarti Panchal (C4GT 2026 intern, Piramal Swasthya)
**Date:** 2026-06-24
**UAT audited:** https://uatamrit.piramalswasthya.org/104/ (running the **legacy** Helpline104-UI / Angular app, "Version 3.5")
**Source of truth read:** `C:/Users/Aarti Panchal/AMRIT/Helpline104-UI/src/app/`
**Method:** Playwright live navigation + reading legacy Angular source. **No code changes, no commits.**

> ⚠️ **Scope limit:** I do not have valid UAT credentials, so I could **not** complete the success paths that require an authenticated session / a `transactionId` (the security-question answering screen, the actual set-password form, and the first-login set-security-questions screen). Those are documented from **source code** (exact contracts) plus what the live error paths revealed. Everything I could reach live is screenshotted; everything I could not is explicitly flagged below.

---

## 0. Key finding up front (backend ≠ legacy UI)

The UAT **backend has been hardened against username enumeration**, but the legacy UI was never updated to handle the new response. This is the single most important thing for the NEXT rebuild.

- **Old UI expectation** (`resetPassword.component.ts` → `handleSuccess`): a `200` with body `{ data: { forgetPassword, SecurityQuesAns: [...] } }`.
- **Actual UAT response today** for an unknown user:
  ```json
  {"statusCode":5002,"errorMessage":"If the username is registered, you will be asked a security question","status":"User login failed"}
  ```
  This body has **no `data` field**, so the legacy service's `extractData()` throws, the error is swallowed into `this.error`, and **the user sees nothing happen** (no alert, no navigation). Verified live — see screenshot 03.

→ **For NEXT:** treat `5002` as the "maybe-registered" privacy response and surface a neutral message; only advance to the question screen when the response actually carries `SecurityQuesAns`.

---

## Route & component map (legacy, from `app.module.ts`)

| Route | Component | Source dir | AuthGuard? |
|---|---|---|---|
| `/resetPassword` | `ResetComponent` | `resetPassword/` | No (public) |
| `/setQuestions` | `SetSecurityQuestionsComponent` | `set-security-questions/` | No (public) |
| `/setPassword` | `SetPasswordComponent` | `set-password/` | No (public) |

All three routes are technically public, but the components depend on **in-memory state** (`dataService.uname`, `dataService.transactionId`, `dataService.Userdata`) that only exists after the preceding step. Navigating to them directly falls back to the login screen (verified live — screenshot 04).

### API base URLs
- Config: `ConfigService.getOpenCommonBaseURL()` → `environment.commonAPI`.
- **On UAT this resolves to:** `https://uatamrit.piramalswasthya.org/common-api/` (confirmed from the live network call to `/common-api/user/forgetPassword`).
- 104 base (`get104BaseURL`) is used only for `version`.

### Password encryption (shared by both set-password paths)
Passwords are **AES-encrypted client-side** before sending (CryptoJS):
- Passphrase (`Key_IV`): `"<redacted>"` (hardcoded in source).
- `keySize` 256, `ivSize` 128, `iterationCount` 1989, hasher SHA-512 (PBKDF2).
- Wire format: `salt(hex) + iv(hex) + ciphertext(base64)`, all concatenated into one string sent as `password`.
- Random `salt`/`iv` per encryption (`CryptoJS.lib.WordArray.random`).

### Password validation rule (UI), identical in both set-password screens
- Regex: `^(?=.*[0-9])(?=.*[A-Z])(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{8,12}$`
- Plain English: **8–12 chars, ≥1 digit, ≥1 uppercase, ≥1 special char from `!@#$%^&*`**, only alphanumerics + those specials allowed.
- UI hint text shown on pattern error: *"Password is required(min:8,max:12,alphanumeric,1 special character,1 upper case)"*
- Also `minlength="8" maxlength="12"` on the inputs. "Change/Update Password" button stays disabled until the form is valid.

---

## 1. FORGOT PASSWORD flow

Entry: login screen → **"Forgot Password?"** link → routes to `/104/resetPassword`.

### Screen 1 — Enter username (`ResetComponent`, `hideOnGettingQuestions=true`)
*Screenshot: `02-forgot-password-username.png`*

| Element | Detail |
|---|---|
| Header | "Account Support" / "Follow the steps to change/reset the password" |
| Field | `Enter User Name` (text). Validators: `required`, `minlength=2`, `usernameValidator`, `autocomplete=off` |
| Button | **Cancel** → `routerLink="/"` (back to login) |
| Button | **Next** → disabled until username valid; calls `getQuestions(userID)` |

**API call on Next:**
- `POST /common-api/user/forgetPassword`
- Request body (verified live): `{ "userName": "<username>" }`
- Service: `loginService.getSecurityQuestions()`.

**Responses:**
- **Unknown / unregistered user (verified live):** `200` + `{"statusCode":5002,"errorMessage":"If the username is registered, you will be asked a security question","status":"User login failed"}`. Legacy UI: silently does nothing (see §0). Screenshot 03.
- **Expected success (from source, NOT reachable without a real account):** `{ data: { forgetPassword: <not "user Not Found">, SecurityQuesAns: [ { question, questionId }, ... ] } }`. `handleSuccess` then shows the questions section.
- **Source-coded edge cases:** `data.forgetPassword == "user Not Found"` → navigate `/` + alert "User not found". `SecurityQuesAns.length == 0` → navigate `/` + alert "Questions are not set for this user".

### Screen 2 — Answer security questions (`showQuestions=true`) — *NOT reachable live (needs valid account)*
From source (`resetPassword.html` + component):
- Shows **one question at a time** (`bufferQuestion`), an `Enter Answer` field (`type` toggles via the `visibility` 👁 icon, tooltip "Show Answer"), validators `required` + `answerValidator`.
- Buttons **Next** (shown while `counter<2`) and **Submit** (shown when `counter>=2`), both disabled until answer valid.
- Asks **3 questions** (`counter` 0→2). Each answer pushed as `{ questionId, answer }` into `userFinalAnswers`. On the 3rd, `checking()` fires.

**API call on final submit:**
- `POST /common-api/user/validateSecurityQuestionAndAnswer`
- Body: `{ "SecurityQuesAns": [ {questionId, answer}, x3 ], "userName": "<uname>" }`
- Service: `loginService.validateSecurityQuestionAndAnswer()`.

**Responses (from source):**
- Success `statusCode == 200`: store `response.data.transactionId` in `dataService`, navigate to **`/setPassword`**.
- Non-200: alert the response as error, re-fetch questions, stay on `/resetPassword`.
- HTTP error: alert `error.errorMessage`, stay on `/resetPassword`.

→ Then continues into the **SET PASSWORD flow (§3)**.

---

## 2. SET SECURITY QUESTIONS flow (first login) — *NOT reachable live (needs a "New" account)*

**Trigger (from `login.component.ts`):** after a successful `userAuthenticate`, if `response.isAuthenticated === true && response.Status === "New"` → `router.navigate(["/setQuestions"])`. (Active users go to `/MultiRoleScreenComponent`.)

Component: `SetSecurityQuestionsComponent`. Two sections in one screen: questions section first (`questionsection=true`), then password section (`switch()` flips to `passwordSection=true`).

**On init (`ngOnInit`):**
- `GET /common-api/user/getsecurityquetions`
- Service: `http_calls.getData(...)`. Response `{ data: [ { QuestionID, Question }, ... ] }` populates 3 dropdowns.

### Section A — choose 3 questions + answers (`set-security-questions.component.html`)
- 3× (`md-select` "Question N" + `Answer N` text input). Each question dropdown filters out already-picked questions (`filterArrayOne/Two/Three`) so all 3 must be unique; duplicate selection → alert *"This question is already selected. Choose unique question"*.
- Question 2 disabled until form1 valid; Question 3 disabled until form2 valid. All `required`.
- **Next** button (disabled until form3 valid) → `setSecurityQuestions()`:
  - Requires exactly 3 selected, else alert *"All 3 questions should be different..."*.
  - Builds `dataArray` of 3 objects: `{ userID, questionID, answers: <trimmed or null>, mobileNumber: "<redacted>", createdBy: <userName> }`.
  - ⚠️ **`mobileNumber` is hardcoded `"<redacted>"`** in source — flag for NEXT.
  - Then `switch()` → shows password section.

### Section B — set password (same validation as §0)
- New Password + Confirm Password fields, password regex hint, 👁 show-password toggle.
- **Update Password** (disabled until valid) → `updatePassword(newpwd)`:
  - If `newpwd === confirmpwd`: `POST /common-api/user/saveUserSecurityQuesAns` with body = `dataArray` (the 3 Q/A objects). Else alert "Password doesn't match".
  - Service: `http_calls.postDataForSecurity(...)`.

**On `saveUserSecurityQuesAns` success** (`handleQuestionSaveSuccess`, requires `statusCode==200` and a `data.transactionId`):
- Chains to `POST /common-api/user/setForgetPassword` with `{ userName, password: <AES-encrypted>, transactionId: response.data.transactionId }`.
- Else alert `response.errorMessage`.

**On `setForgetPassword` success** (`successCallback`):
- alert *"Password changed successfully"* → `loginService.userLogout()` (`POST /common-api/user/userLogout`) → `authService.removeToken()` → navigate to `''` (login).

---

## 3. SET PASSWORD flow (`SetPasswordComponent`, route `/setPassword`)

Reached after security-question validation in the forgot-password flow (§1) carries a `transactionId` into `dataService`.

*Live: navigating to `/104/setPassword` directly **redirects/falls back to the login screen** (screenshot 04) — confirms it's unusable without an active reset session. The set-password form itself was not reachable live.*

From source (`set-password.component.html` + `.ts`):

| Element | Detail |
|---|---|
| Header | "Account Support" / "Follow the steps to change/reset the password" |
| Field | **New Password** — `pattern`=password regex, `minlength=8`, `maxlength=12`, `required`, 👁 toggle (tooltip "Show Password") |
| Field | **Confirm Password** — same validators |
| Hint | *"Password is required(min:8,max:12,alphanumeric,1 special character,1 upper case)"* on pattern error |
| Button | **Change Password** — disabled until `passwordFields` form valid; calls `updatePassword(newpwd)` |

**API call on Change Password (`updatePassword`):**
- Pre-check: `newpwd === confirmpwd` (else alert *"Passwords does not match"* — no API call).
- `POST /common-api/user/setForgetPassword`
- Body: `{ "userName": <dataService.uname>, "password": <AES-encrypted newpwd>, "transactionId": <dataService.transactionId> }`
- Service: `http_calls.postData(...)`.

**Responses (from source):**
- Success (`successCallback`): alert *"Password changed successfully"* (success) → `loginService.userLogout()` (`POST /common-api/user/userLogout`) → on success `authService.removeToken()` → navigate to `''` (login). `transactionId` is cleared.
- Error: alert `error.errorMessage` (error) → navigate `/resetPassword`.

---

## Consolidated API endpoint table

| # | Flow / step | Method + endpoint | Request body | Service method |
|---|---|---|---|---|
| 1 | Forgot pw — fetch questions | `POST /common-api/user/forgetPassword` | `{ userName }` | `getSecurityQuestions` |
| 2 | Forgot pw — validate answers | `POST /common-api/user/validateSecurityQuestionAndAnswer` | `{ SecurityQuesAns:[{questionId,answer}×3], userName }` | `validateSecurityQuestionAndAnswer` |
| 3 | Set pw (forgot flow) | `POST /common-api/user/setForgetPassword` | `{ userName, password(AES), transactionId }` | `postData` |
| 4 | First login — list questions | `GET /common-api/user/getsecurityquetions` | — | `getData` |
| 5 | First login — save Q/A | `POST /common-api/user/saveUserSecurityQuesAns` | `[{userID,questionID,answers,mobileNumber,createdBy}×3]` | `postDataForSecurity` |
| 6 | First login — set pw | `POST /common-api/user/setForgetPassword` | `{ userName, password(AES), transactionId }` | `postDataForSecurity` |
| 7 | Post-success logout | `POST /common-api/user/userLogout` | `{}` | `userLogout` |

---

## Findings / flags for the NEXT rebuild

1. **`5002` enumeration-protection response is unhandled by the legacy UI** (§0). Highest priority — the forgot-password flow currently appears broken to end users on UAT.
2. **Hardcoded `mobileNumber: "<redacted>"`** in `setSecurityQuestions()` — the real number is never collected/sent.
3. **Hardcoded encryption passphrase** `"<redacted>"` in client source (and salt label `"RandomInitVector"`). Security smell; mirror backend expectations carefully but flag.
4. **Silent error swallowing** — `getQuestions` error handler just sets `this.error` with no user feedback; several callbacks `console.log` only.
5. **One-question-at-a-time** reset UX with `Next`/`Submit` driven by a `counter`; the answer field reuses the password show/hide toggle.
6. **Password rule (8–12 chars)** is unusually short as a *max*; confirm whether NEXT should keep `maxlength=12`.
7. **Two separate set-password implementations** (`set-password` vs the password section inside `set-security-questions`) duplicate the same crypto + regex — candidate for a shared component/service in NEXT.

## Screenshots (saved to project root)
- `01-login-screen.png`, `01-login-screen-clean.png` — login + "Forgot Password?" link
- `02-forgot-password-username.png` — reset username screen
- `03-forgot-password-unknown-user-result.png` — unknown-user result (silent, the 5002 case)
- `04-set-password-screen.png` — `/setPassword` direct nav falls back to login

## Not verified live (need valid UAT credentials)
- Security-question **answering** screen (forgot flow, screen 2) and the success → `/setPassword` transition.
- The actual **set-password form** rendering and a successful password change.
- **First-login** set-security-questions screen (requires an account with `Status === "New"`).
- Exact success-response **shapes** for endpoints #2–#6.
