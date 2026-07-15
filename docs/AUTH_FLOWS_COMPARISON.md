# AMRIT Helpline 104 — Auth Flows: Local (NEXT) vs Live UAT Comparison

**Author:** Aarti Panchal (C4GT 2026 intern, Piramal Swasthya)
**Date:** 2026-06-25
**Local app:** `http://localhost:4200` (Helpline104-UI-NEXT, branch `feat/auth-flows`)
**Live UAT:** https://uatamrit.piramalswasthya.org/104/ (legacy Helpline104-UI, "Version 3.5")
**Method:** Playwright navigation + screenshots of both apps. UAT forgot-password triggered with username `104hao` (real account, returns real security questions). Local backend mocked at the browser to render every screen with the same question wording for a fair visual comparison.

---

## Scope & limitations (read first)

| Screen | UAT captured? | Local captured? | Note |
|---|---|---|---|
| Login (entry to forgot-pw) | ✅ | ✅ (existing) | |
| Reset — username | ✅ | ✅ | |
| Reset — security questions (Q1–Q3) | ✅ | ✅ | UAT returned 3 real questions for `104hao` |
| Reset — answer validation error | ✅ | ✅ (via dialog) | |
| Set password | ❌ **not reachable on UAT** | ✅ | UAT needs the *correct* security answers (unknown to me); documented from legacy source instead |
| Set security questions (first login) | ❌ **not reachable on UAT** | ✅ | UAT needs a `Status: "New"` account; `104hao` is already set up |

> I could not pass the UAT security questions (the real answers for `104hao` are unknown), so the UAT **set-password** and **first-login set-security-questions** screens could not be screenshotted live. Those comparisons fall back to the legacy source contract documented in `docs/AUTH_FLOWS_AUDIT.md`. Everything reachable is screenshotted.

### Screenshot index — `screenshots/auth-flows-comparison/`
**UAT:** `uat-00-login.png`, `uat-01-reset-username.png`, `uat-02-question-1.png`, `uat-03-question-2.png`, `uat-04-question-3.png`, `uat-05-answers-submit-result.png`
**Local:** `local-01-reset-username.png`, `local-02-reset-neutral.png`, `local-03-question-1.png`, `local-04-question-2.png`, `local-05-question-3.png`, `local-06-set-password.png`, `local-07-set-password-validation.png`, `local-08-set-password-success.png`, `local-09-set-security-questions-empty.png`, `local-10-set-security-questions-filled.png`

---

## Cross-cutting differences (apply to all reset screens)

| Aspect | UAT (legacy) | Local (NEXT) | Assessment |
|---|---|---|---|
| **Page chrome** | Reset screens are a bare centered white card on a light-gray page — **no top bar, no footer, no language selector**. (The login page has a dark footer with "Powered by: WIPRO / Feedback / Version 3.5" but the reset page drops even that.) | Full app shell: blue `AppHeader` (logo + "AMRIT" + centered "Account Support" title + globe/language selector) and dark `AppFooter` ("2026 © PSMRI" + "Version 1.0.1"). | **Intentional divergence.** NEXT standardises chrome across the app. Decide whether pre-auth screens *should* show the language selector & version — see P1. |
| **Branding** | "Piramal **Swasthya**" full logo, centered inside the card. | "Piramal Swasthya" logo + "AMRIT" wordmark in the header bar (white, inverted). | NEXT moved branding into the shared header. Acceptable. |
| **"Powered by: WIPRO"** | Present (login footer). | Removed (intentional, per dashboard review). | OK — already a deliberate product decision. |
| **Card style** | Square corners, flat 1px shadow, white. | Rounded-xl, border + soft shadow, white `bg-card`. | Cosmetic; NEXT is more modern. |
| **Input style** | Material **underline** inputs with a leading icon (account_box / lock) and floating-style label. | Bordered **box** inputs (`z-input`) with a label *above* the field; password fields keep a leading lock icon. | Cosmetic; consistent within NEXT. |
| **Error feedback** | Material modal dialog: **red "Error" header bar**, message, single **"Ok"** button, dimmed backdrop. | ZardUI dialog: "Error" title, message, "OK" button (no red header bar) for answer-validation; inline red text for set-password; success uses a **toast**. | Functionally equivalent; styling differs. |
| **Typography** | Question text is **red, bold, italic**. | Question text is **bold, foreground color**, with a "Question X of 3" progress label above it. | NEXT is clearer (adds progress, drops the alarming red). |

---

## Screen-by-screen comparison

### 1. Entry point — Login → "Forgot Password?"
*UAT: `uat-00-login.png` · Local: existing login screen*

- **UAT:** "Forgot Password?" link bottom-right of the login card → routes to `/104/resetPassword`.
- **Local:** "Forgot Password?" link bottom-right of the login card → routes to `/reset-password`.
- **Visual:** UAT login has the WIPRO/Feedback/v3.5 footer; local login has a minimal "© 2026 PSMRI · AMRIT" footer and no top bar. Field styling differs (underline vs box) as above.
- **Functional:** ✅ Equivalent — both expose forgot-password from the same place.

### 2. Reset — username step
*UAT: `uat-01-reset-username.png` · Local: `local-01-reset-username.png`*

| | UAT | Local |
|---|---|---|
| Title | "Account Support" / "Follow the steps to change/reset the password" | Same subtitle, plus card heading "Reset your password" |
| Field | `Enter User Name *`, underline + account_box icon | `User Name *` label above a boxed input, placeholder "Enter user name" |
| Buttons | **Cancel** (solid blue) + **Next** (gray, disabled) side by side | **Cancel** (outline) + **Next** (filled, disabled) as two equal columns |
| Validation | `required`, `minlength=2` | `required`, `minlength=2` (parity ✅) |

- **Functional:** ✅ Equivalent. **Visual:** Cancel emphasis is flipped (UAT makes Cancel the prominent blue button; local makes Next the primary). Minor — see P2.

### 3. Reset — security questions (one at a time)
*UAT: `uat-02/03/04-question-*.png` · Local: `local-03/04/05-question-*.png`*

| | UAT | Local |
|---|---|---|
| Layout | One question per screen | One question per screen (parity ✅) |
| Progress | None | **"Question X of 3"** counter (improvement) |
| Question text | Red, bold, italic | Bold, foreground |
| Answer field | Underline input **+ show/hide (eye) toggle** | Boxed input, **no show/hide toggle** |
| Buttons | **Both "Next" and "Submit" shown, stacked** (the inapplicable one disabled) | **Single button** whose label switches "Next" → "Submit" on the last question |
| Questions returned | 3 (real) | Driven by API response count (we render exactly what `forgetPassword` returns) |

- **Functional differences:**
  - UAT shows Next **and** Submit simultaneously; local shows one context-aware button. Local is cleaner; behavior is equivalent.
  - UAT lets you reveal the typed answer (eye toggle); local does not. Minor parity gap — see P1.
  - **Question count:** legacy always iterated exactly 3; local iterates `SecurityQuesAns.length`. For `104hao` both are 3. If any account has ≠3 stored questions, behavior could differ — verify (P0).

### 4. Reset — answer validation error
*UAT: `uat-05-answers-submit-result.png` · Local: error dialog via `ConfirmDialogService.alert`*

- **UAT:** Wrong answers → Material dialog, red "Error" bar, *"We couldn't verify your answers. Please try again"*, "Ok"; resets to question 1.
- **Local:** Wrong answers → ZardUI error dialog (title "Error", API message, "OK"); resets to question 1 (`restartQuestions()`).
- **Functional:** ✅ Equivalent (dialog + reset). **Visual:** UAT has the red header bar; local dialog is neutral-styled. See P2.

### 5. Set password
*UAT: not reachable (documented from legacy source) · Local: `local-06/07/08-set-password*.png`*

| | UAT (from source) | Local |
|---|---|---|
| Fields | New Password + Confirm Password (underline, lock icon, eye toggle) | New Password + Confirm Password (box, lock icon, eye toggle) |
| Policy | `8–12, ≥1 digit, ≥1 upper, ≥1 special`; hint always visible: *"Password is required(min:8,max:12,...)"* | Same regex; inline `z-form-message` shown **on error**: "Password must be 8–12 characters with at least 1 digit, 1 uppercase letter and 1 special character (!@#$%^&*)." |
| Confirm match | `===` check before submit, else alert "Passwords does not match" | Cross-field validator + inline "Passwords do not match." (live, before submit) |
| Submit | "Change Password" → `setForgetPassword` → success alert → logout → login | "Change Password" → `setForgetPassword` → **success toast** "Password changed successfully. Please sign in." → `/login` |

- **Functional:** ✅ Equivalent contract; local adds live confirm-match feedback (improvement). **Could not visually diff UAT live.**

### 6. Set security questions (first login)
*UAT: not reachable (no "New" account) · Local: `local-09/10-set-security-questions*.png`*

| | UAT (from source) | Local |
|---|---|---|
| Trigger | login `Status === "New"` → `/setQuestions` | login `Status === "New"` → `/set-security-questions` (parity ✅) |
| Questions | **3** dropdowns, must be distinct | **3** dropdowns, must be distinct (cascading exclusion) ✅ |
| Password | New + Confirm password section | New + Confirm password section ✅ |
| `mobileNumber` | Hardcoded `"<redacted>"` (audit finding #2) | Sent **empty `""`** (deliberate fix) ✅ improvement |
| API chain | `saveUserSecurityQuesAns` → `setForgetPassword` (transactionId chained) | Same two-call chain via `switchMap` ✅ |
| Outcome | success alert → logout → login | success toast → `/login` ✅ |

- **Functional:** ✅ Matches legacy contract (verified by code review). **Could not visually diff UAT live.**

---

## Priority list

### P0 — Must fix / verify before PR merges
1. **Verify the first-login contract against the live backend.** We could not exercise `/set-security-questions` on UAT (no `Status:"New"` account). The local flow assumes the legacy chain (3 questions → `saveUserSecurityQuesAns` → `setForgetPassword`). If the UAT backend has changed (e.g. no longer needs the chained password step, or expects a different question count), a real "New" user could be **locked out**. → Get a `New` test account or backend confirmation. *(Carryover of earlier review item H1/H2.)*
2. **Confirm variable question-count handling.** Local sends back exactly the questions `forgetPassword` returned (here 3); legacy hard-assumed 3. Confirm `validateSecurityQuestionAndAnswer` accepts whatever count is echoed, so accounts with ≠3 stored questions don't break.
3. **No remaining visual defects.** The earlier Next-button overflow is fixed; reset/set-password/set-security screens all render correctly and contained. ✅ (nothing to fix here — listed for sign-off).

### P1 — Fix before the inbound-call workflow ships
1. **Answer field show/hide toggle.** UAT lets agents reveal the security answer (eye icon); local does not. Agents on calls may want to verify what they typed — add a show/hide toggle to the answer input for parity.
2. **Decide pre-auth chrome.** Local reset screens show the full header (incl. **language selector**) and footer; UAT shows a bare card. Confirm with the team whether the language selector belongs on pre-login screens (useful for regional agents) or should be hidden for a focused recovery flow. Make it a deliberate choice, not incidental.
3. **Error-dialog parity.** Align the local error dialog's emphasis with UAT's recognizable red "Error" treatment (or formally standardize on the ZardUI style app-wide) so agents see a consistent error language.

### P2 — Nice to have
1. **Question text styling.** Local uses bold foreground + "Question X of 3"; UAT uses red italic. Local is arguably clearer — keep, but note the divergence for design sign-off.
2. **Username field icon.** Add a leading user icon to the reset username field to match UAT's `account_box` affordance (login screen already has one).
3. **Cancel-vs-Next emphasis.** UAT styles Cancel as the prominent blue button; local makes Next primary and Cancel outline. Local convention (primary = forward action) is standard — keep, but confirm with design.
4. **Single vs dual buttons on question screen.** Local's single context-aware button (Next→Submit) is cleaner than UAT's stacked Next+Submit; no change needed, documented for awareness.

---

## Summary

Functionally the local NEXT flows are **at parity with or ahead of** legacy UAT on every reachable screen: same routes, same API contract, same validations, plus genuine improvements (privacy-preserving neutral message for unknown users, live confirm-match feedback, progress counter, non-hardcoded `mobileNumber`, success toasts). The differences are mostly **intentional modernisation** (app chrome, box inputs, dialog styling). The only true risk items are **P0 #1/#2** — both are *backend-contract verifications* for the two flows that could not be exercised live, not local code defects.
