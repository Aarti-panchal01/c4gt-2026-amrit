# Helpline104-UI Migration Plan (v2)

> **Author:** Aarti Panchal (C4GT 2026 intern, Piramal Swasthya / AMRIT)
> **Mentor:** Dr. Mithun James
> **Old repo:** https://github.com/PSMRI/Helpline104-UI (Angular 4.1.3)
> **New repo:** https://github.com/PSMRI/Helpline104-UI-NEXT (Angular 20.3, scaffolded)
> **Working branch:** `angular-zard-migration` → small PRs → merge to `main` only after QA
> **Timeline:** Migration **complete by mid-August 2026**; **September = cleanup, deployment, QA, testing**

This revision (v2) rewrites the original AI-drafted plan to address Dr. Mithun James's review of 12 Jun 2026:
(1) a concrete **validation / anti-stub checklist** per module, 
(2) folder structure **aligned to AMRIT conventions** (`src/app/app-modules/…`, studied from MMU-UI and Common-UI), 
(3) a **Common-UI shared-component strategy** (consume Gopi's Common-UI v2 branch when ready),
(4) a **mid-August** timeline with September reserved for cleanup/deploy/QA, and (5) an explicit **Claude Code subagent PR-review process**.

---

## 1. Why a fresh repo (not an upgrade)

The old app is **Angular 4.1.3** — six major generations behind Angular 20. Nearly every foundational API it relies on has been removed or replaced, so a clean rebuild is faster and safer than a chained upgrade.

| Old (Angular 4) | Angular 20 |
|---|---|
| `@angular/http` (`Http`, `InterceptedHttp extends Http`) | `HttpClient` + functional interceptors |
| One giant `NgModule` (`app.module.ts`) + flat role folders | Standalone components, no NgModule, lazy routes |
| `@angular/material` **2.0.0-beta.11** + `md2 0.0.18` (the `md-*` API) | Removed entirely → **ZardUI** |
| RxJS 5.4 (`.map()`, `rxjs/Rx`) + **Signals** | RxJS 7.8 (`.pipe(map())`) + **Signals** |
| Template-driven `ngModel` everywhere | Reactive Forms / Signals |
| Zone.js change detection | **Zoneless** (`provideZonelessChangeDetection`) |
| jQuery + Bootstrap carousel (`innerpage`) for screen switching | Angular Router |

**Business logic and API contracts stay valid** — only the UI/runtime layer is rebuilt. (Verified: 118 components, 60 services in the old app; see §3.)

---

## 2. What the old app does (functional map)

AMRIT Helpline 104 is a **health helpline call-center application**. Agents in different roles handle inbound/outbound calls, register beneficiaries, fill medical case sheets, and close calls. Supervisors run reports and configure the system.

### 2.1 Roles
A user can hold **multiple roles**; after login they pick a service + role on the **multi-role screen**. Privileges come from `response.Previlege[]` filtered to `Service == "104"` (confirmed against the API — see §4).

| Code | Role |
|---|---|
| **RO** | Registration Officer |
| **HAO** | Health Advice Officer |
| **MO** | Medical Officer |
| **CO** | Complaint Officer |
| **Counsellor** | Health Counsellor |
| **PD** | Program Data |
| **SIO** | Special Information Officer (blood-on-call, epidemic, food safety, grievance, organ donation, schemes) |
| **Supervisor** | Agent monitoring, quality audit, reports, config |
| **Surveyor** | Field survey, call-type reports |

### 2.2 Core workflows

**Inbound call pipeline:**
`Incoming call (Czentrix CTI)` → `Beneficiary registration (RO)` → `Case sheet (HAO/MO/Counsellor)` → `Closure (call type / sub-type / followup)`

**Outbound pipeline:**
`Worklist (pre-allocated by role)` → `Select record` → `Outbound call + case sheet` → `Closure` (with allocate / search / reallocate tooling)

### 2.3 Feature inventory (old repo: **118 components, 60 services**)
- **Auth:** login (PBKDF2-SHA512 + AES, optional captcha), set-password, set-security-questions, reset-password
- **Shell / dashboard:** multi-role-screen, service-role-selection, dashboard (+ navigation / row-header / user-id), innerpage (jQuery carousel host — to be replaced by Router)
- **Inbound:** beneficiary-registration-104, case-sheet (+ covid / cancer / general / mcts / mmu variants), cheif-complaint-snomed-search, cdss / algo-component, prescription, closure, insert-complaint, bp-screening, diabetic-screening, schedule-appointment, consent
- **Outbound:** outbond-worklist, agent-outbondcall, outbound-allocate-records, outbound-search-records, reallocate-calls, dial-beneficiary
- **SIO sub-services:** blood-on-call, epidemic-outbreak, food-safety, grievience, information, organ-donation, scheme, sio-services-history, sio-outbound-provider
- **Supervisor (~18):** call-summary / call-quality / calltype / complaint-detail / district-wise-call-volume / diseases-summary / quality / unblock-user reports; configurations, upload-schemes, alerts-notifications, notifications, emergency-contacts, training-resources, location-communication, blood-url, grievance; agent-status, block-unblock-number, force-logout, quality-audit
- **Other reports:** medical-advise-report, mental-health-report, blood-on-call-detailed-report, surveyor-calltype-reports, covid-19, bal-vivah
- **Shared:** loader, captcha, common-dialog, message-dialog, notifications-dialog, edit-notifications, rating, custom-pipe (search filter), order-by.pipe, utc-date.pipe, validator directives
- **Integrations:** Czentrix CTI (REST — see note below), Socket.io (real-time notifications), SNOMED-CT, CDSS, i18n (`assets/i18n`)

### 2.4 Services layer (key ones)

`loginService`, `dataService` (central RxJS-subject state store), `AuthService`, `ConfigService`, `HttpServices`, `SearchService`, `RegisterService`, `CaseSheetService`, `CallServices`, `PrescriptionService`, `CDSSService`, `SnomedService`, `LocationService`, `SocketService`, `SessionService`, `LoaderService`, the four `Outbound*` services, six `sio*` services, ~10 `supervisor*` services, plus the HTTP plumbing (`http.interceptor.ts`, `http.securityinterceptor.ts`, `http.factory.ts`, `http.security.factory.ts`).

> **Migration note:** `dataService` is a god-object using `Subject`/`BehaviorSubject` for cross-component state. In Angular 20 this becomes **small, focused signal stores** (e.g. `AuthStore`, `CallStore`, `RoleStore`), and the two HTTP interceptor classes collapse into **functional `HttpInterceptorFn`s** registered with `provideHttpClient(withInterceptors([...]))`.

> ⚠️ **CTI — flagged for mentor.** The confirmed decision says *"CTI: same iframe integration, don't touch."* The old code, however, integrates Czentrix as a **REST service** (`czentrix.service.ts` calling `/cti/doAgentLogin`, `/callBeneficiary`, `/setCallDisposition`, `/getLoginKey`, …) against the `telephoneServer` env URL — **no iframe was found** in the old repo. This plan therefore describes CTI as the existing **REST integration, preserved as-is**. *Action: confirm with Dr. Mithun James whether an iframe softphone exists elsewhere or the "iframe" wording was imprecise.*

---

## 3. Old-app structure vs AMRIT convention (the alignment problem the mentor raised)

**Old Helpline104-UI** is flat and role-based — no `core/`, no `app-modules/`:
```
src/app/
  104-ro/ 104-mo/ 104-hao/ 104-co/ 104-sio/ 104-supervisor/ 104-surveyor/ …  (role folders)
  login/ dashboard/ innerpage/ multi-role-screen/ …
  services/        (32+ subfolders: authentication, czentrix, dataService, socketService, …)
  material.module.ts  http.interceptor.ts  http.factory.ts  app.module.ts
```
It does **not** use Common-UI (self-contained, no `.gitmodules`).

**Canonical AMRIT layout** (studied from **MMU-UI**, Angular 16, the most-modernized AMRIT app, and **Common-UI**):
```
src/
  app/
    app-modules/
      core/        → core.module, material.module, components/, directives/, services/, mocks/
      login/
      <feature>/   (nurse-doctor, lab, pharmacist, data-sync, …)
      registrar/   (consumed from the Common-UI git submodule)
    app-routing.module.ts  app.module.ts  app.component.*
  environments/    → environment.{local,dev,test,prod}.ts + environment.ci.ts.template
  assets/  css/  styles.scss
```

**Decision (confirmed with mentor): adopt the AMRIT `src/app/app-modules/…` taxonomy, but keep Angular 20 standalone internals** (no NgModules; each feature folder owns a `*.routes.ts` and standalone components). This satisfies "align with AMRIT conventions" on the folder taxonomy a reviewer navigates, while still dropping NgModules. See §6 for the exact tree.

---

## 4. API contracts to preserve (Helpline104-API)

**Spring Boot 3.2.2 / Java 17**, single-module WAR, **port 8091**, root context path, Redis sessions (2h, auto-extend), **JWT** in `Jwttoken` cookie + `Authorization` header. **API testing uses the UAT base URL** (confirmed decision).

- **Login:** `POST /user/userAuthenticate` → `{ userID, isAuthenticated, userName, Status, Previlege[] }`, where `Previlege[].Service` + `.Role[]` drive role/service-104 filtering.
- **No-auth (exempt) endpoints:** `userAuthenticate*`, `forgetPassword`, `setForgetPassword`, `changePassword`, `saveUserSecurityQuesAns`, `version`, `health`, swagger paths.
- **Endpoint families (base paths):** `/user` (auth), `/beneficiary` (calls, case sheet, blood-on-call, organ-donation, feedback, SIO history, IMR/MMR), `/outbound`, `/location` (states / districts / taluks / city / village), `/crmReports` (supervisor Excel reports), plus disease / CDSS / balVivah.
- **Response envelope:** every endpoint returns `{ "response": <data>, "error": <msg|null> }`.
- **Common-API dependency:** SMS (`/sms/sendSMS`), email (`/emailController/sendEmailGeneral`), and feedback creation are delegated by the 104 API to **Common-API** (`common-url`). The new UI's `environment` therefore needs **both** `ip104` and `commonAPI` base URLs (the old env already carries both, plus `telephoneServer` for CTI).

**Rule:** request/response shapes, the `{response,error}` envelope, the privilege object, JWT-cookie + `Authorization` header behaviour, and the `5002` "already logged in" force-logout path are all **contracts — preserved exactly**. Each migrated call is diffed against the old service before its PR merges (see §8 checklist).

---

## 5. Common-UI strategy (mentor point #3)

### 5.1 What Common-UI actually is
`PSMRI/Common-UI` is a **git submodule** (not an npm package): pure source (125 files), **no `package.json`/`angular.json`/`tsconfig`**. Host apps embed it at `./Common-UI` and import with bare paths (`Common-UI/src/registrar/…`, `Common-UI/src/tracking`). It contains:
- **`registrar/`** — patient registration stepper, search, family-tagging, **ABHA/ABDM (13 components)**, and services incl. **`SessionStorageService`** (the de-facto shared session layer; MMU imports it in 20+ files).
- **`feedback/`** — public feedback page + dialog.
- **`tracking/`** — Matomo/GA/Amrit tracking; **the only piece exported via `public-api.ts`**.

⚠️ **Critical coupling:** Common-UI today is **NgModule + Angular Material** and imports *back into the host app* (`src/app/app-modules/core/material.module`, `…/shared.module`, `src/environments/environment`). It **cannot be consumed directly by an Angular 20 standalone + ZardUI app** without a compatibility path. This is the central constraint of the shared-component workstream.

### 5.2 The 104 ↔ 1097 overlap (evidence for extraction)
Helpline1097-UI (same Angular 4.4.4, same flat structure) shares **~42 same-named components and ~51 same-named services** with 104. The genuinely shared layer:
- **Auth/session/http:** `login` + `captcha` (near-identical), `auth.service`, `authGuardService`, `sessionStorageService`, `socket.service` (differs only by hardcoded IP → must become config-driven), `czentrix.service`, the http interceptor/factory.
- **Shell:** `dashboard` (+ navigation / row-header / user-id), `multi-role-screen`, `innerpage`, `service-role-selection`.
- **Dialogs & utils:** `common-dialog`, `message-dialog`, `notifications-dialog`, `loader`, `confirmation.service`, `location.service`, `config.service`, validator directives, search/order-by pipes.

104-only = clinical (case-sheet, CDSS, SNOMED, prescription, screenings) + SIO + surveyor. 1097-only = Everwell outbound, grievance, CO counselling, demographic reports. **The shared shell/auth/session/dialog/dashboard layer is the extraction target.**

### 5.3 Approach

Because Common-UI is Material/NgModule-bound, we cannot simply import it. Instead:

1. **Consume Gopi's Common-UI v2** — Gopi (MMU intern) is upgrading Common-UI to Angular 20 on an `angular-zard-migration` branch with a `v2/` folder structure (Mithun confirmed: copy existing src to v2 to start). Once ready, 104-NEXT will consume it directly.

2. **Port `SessionStorageService` encryption contract** — keep encrypted sessionStorage byte-compatible with MMU/1097 regardless of Common-UI version.

3. **Consume `tracking` now** — the only cleanly published piece of Common-UI; wire it in P0 foundation.

4. **Build shared shell/auth/session/dialog as standalone + ZardUI inside 104-NEXT** — in `app-modules/core/components/`, designed to be lifted out later if needed.

5. **Theme** (confirmed): Piramal blue/white — coordinate palette with Madhav + Gopi.

> **Coordination:** Monitor Gopi's Common-UI v2 branch. No need to fork or seed a separate standalone track — consume his work when ready.

---

## 6. Folder structure (AMRIT-aligned, Angular 20 standalone)

Adopts the AMRIT `src/app/app-modules/…` taxonomy; standalone internals; lazy routes per feature. **`core/tokens/` is removed — config lives in `environments/`** (confirmed decision; the old app had no `tokens/` folder, and the scaffold's `core/tokens/` is deleted).

```
src/
├─ app/
│  ├─ app-modules/
│  │  ├─ core/                      # app-wide singletons (provided once at root)
│  │  │  ├─ auth/                   # auth.service, auth.store (signals), auth.guard (CanActivateFn), role.guard
│  │  │  ├─ http/                   # auth.interceptor, error.interceptor, loader.interceptor (HttpInterceptorFn)
│  │  │  ├─ services/               # config, session (timeout/keepalive), socket (Socket.io), location, cti (czentrix REST)
│  │  │  ├─ models/                 # shared TS interfaces / API DTOs ({response,error} envelope, Previlege[])
│  │  │  ├─ components/             # app-level shared shell pieces (lift-out-ready → Common-UI)
│  │  │  ├─ directives/             # validators (name, mobile, email, password)
│  │  │  └─ ui/                     # ZardUI primitives (per components.json alias) + cn() helper
│  │  ├─ login/                     # login (PBKDF2/AES), captcha, set-password, set-security-questions, reset-password
│  │  ├─ role-selection/            # multi-role-screen, service-role-selection (privilege-driven)
│  │  ├─ dashboard/                 # dashboard + navigation + row-header + user-id
│  │  ├─ call/                      # INBOUND: shell (router, no jQuery), beneficiary-registration,
│  │  │                             #          case-sheet, cdss/algo, snomed-search, prescription,
│  │  │                             #          closure, insert-complaint, screenings
│  │  ├─ outbound/                  # worklist, outbound-call, allocate, search, reallocate
│  │  ├─ sio/                       # blood-on-call, organ-donation, food-safety, epidemic, grievance, information, scheme
│  │  ├─ supervisor/                # reports/ + config/
│  │  ├─ reports/                   # surveyor, covid, mental-health, medical-advise, blood-on-call
│  │  └─ registrar/                 # (future) Common-UI submodule consumption — registration/ABHA
│  ├─ shared/                       # reusable, no business logic (lift-out candidates → Common-UI)
│  │  ├─ components/                # data-table, page-header, confirm-dialog, file-export
│  │  ├─ pipes/                     # order-by, utc-date, search-filter
│  │  └─ utils/                     # export (csv/xlsx), crypto, date helpers
│  ├─ app.config.ts                 # providers (router, http, zoneless change detection)
│  ├─ app.routes.ts                 # top-level lazy routes
│  └─ app.ts                        # root component (shell outlet)
│
├─ environments/                    # environment.ts / .prod.ts / UAT — base URLs (ip104, commonAPI, telephoneServer), encKey, captcha, tracking
├─ Common-UI/                       # git submodule (tracking consumed now; shared shell contributed back later)
└─ styles.css                       # global + Tailwind v4 entry (CSS-first, see §9)

docs/
└─ migration-plan.md                # this file
```

**Conventions:** standalone components only; signals for state; `inject()` over constructor DI where natural; Reactive Forms; one feature = one folder + one `*.routes.ts`, lazy-loaded; `core/` provided once at root; `shared/` is import-only; GPL-3.0 license header on every file; `app` component prefix; Conventional Commits + commitlint; Prettier (2-space, single quotes, 80 col) — matching MMU.

---

## 7. ZardUI + component mapping

[ZardUI](https://zardui.com/) is the shadcn/ui equivalent for Angular: standalone, signal-based, OnPush, zoneless-ready components on TypeScript + TailwindCSS v4 + CVA. Free/OSS. The CLI **copies source into our repo** (we own it). Requires Angular 19+ (we're on 20.3 ✅), Node 20/22, **Tailwind v4**.

**Old `md-*` / `md2` / 3rd-party → ZardUI:**

| Old widget | ZardUI equivalent | Notes |
|---|---|---|
| `md-button`, `md-raised-button` | **Button** | variants via CVA |
| `md-input-container` + `mdInput` | **Input** + **Form** | reactive forms |
| `md-select` / `md-option` | **Select** | searchable → **Combobox** |
| `md-radio-group`, `md-checkbox`, `md-slide-toggle` | **Radio / Checkbox / Switch** | |
| `md-datepicker`, md2 datepicker | **Date Picker** / **Calendar** | |
| `md-dialog` (+ common/message dialogs) | **Dialog** / **Alert Dialog** | one reusable `ConfirmDialog` wrapper |
| `md-tab-group` | **Tabs** | |
| `md-toolbar`, `md-sidenav` | layout div + **Sheet**/**Menu** | build header/sidebar in `core/components` |
| `md-icon` | **Icon** (lucide via `@ng-icons`) | |
| `md-tooltip`, `md-menu`, `md-card`, `md-expansion-panel` | **Tooltip / Dropdown / Card / Accordion** | |
| `md-progress-spinner`, loader/ | **Loader** / **Progress** / **Skeleton** | |
| `md-snack-bar`, `angular2-toaster` | **Toast** | |
| `md-stepper` (`MatStepper`) | **Tabs**/**Segmented** + custom step state | ZardUI has no stepper |
| `ng2-smart-table` | **Table** + **Pagination** | build one reusable `DataTable` (sort/filter/paginate) |
| `angular2-csv`, `xlsx`, `file-saver` | keep as utilities | wrap in `shared/utils/export.ts` |
| jQuery carousel (`innerpage`) | Angular Router | remove jQuery entirely |

> Highest-leverage shared component to build early: a reusable **`DataTable`** — the supervisor/reports area needs it dozens of times.

---

## 8. Validation — the anti-stub gate (mentor point #1)

**Problem:** AI-generated code can silently leave `TODO`s, stubbed handlers, fake data, or unwired buttons that *look* done. No PR merges until it passes this gate. The gate is **two layers: an automated check + a human/AI reviewer checklist.**

### 8.1 Automated "no-stub" check (runs in CI + locally before every PR)
A script (`npm run verify:nostub`) fails the build if any of these appear in changed files:
- `TODO`, `FIXME`, `XXX`, `HACK`, `@ts-ignore`, `@ts-nocheck`
- `throw new Error('not implemented')` / `throw new Error('Method not implemented')`
- empty method bodies on declared handlers, `console.log(` left in, commented-out blocks of old code
- placeholder text: `lorem`, `dummy`, `placeholder`, hardcoded sample/fake API responses
- `: any` beyond an allowlist; unused imports; `(click)` handlers bound to non-existent / empty methods
- TypeScript build must pass with **no errors and no new warnings**; ESLint clean; Prettier clean.

### 8.2 Per-module Definition of Done (every module, before its PR)
A module's PR description **must** include this checklist, all boxes ticked:

- [ ] **Feature parity** — every screen/field/action from the old component is present (old vs new screenshot pair attached).
- [ ] **No stubs** — `npm run verify:nostub` passes; no TODO/placeholder/dummy data; every button/link is wired to a real action.
- [ ] **API verified** — every HTTP call diffed against the old service: same endpoint, method, request body shape, and `{response,error}` handling. UAT call tested and screenshot/log attached.
- [ ] **Forms** — all validations from the old template-driven form reproduced as Reactive Forms validators; error messages match.
- [ ] **State** — uses signal store / `inject()`; no leftover `dataService` god-object pattern; subscriptions cleaned up.
- [ ] **Roles/privileges** — screen visibility honours `Previlege[]` filtered to service `104`.
- [ ] **i18n** — all visible strings use i18n keys; same languages as the current app (confirmed decision); no hardcoded English.
- [ ] **Theme** — Piramal blue/white tokens; no stray Material/`md-*`; ZardUI components only.
- [ ] **Accessibility** — labels, focus order, keyboard nav, `aria-*` on dialogs/tables.
- [ ] **Responsive** — verified at mobile + desktop breakpoints.
- [ ] **Tests** — unit tests for services/stores and at least the happy-path + one error-path per component.
- [ ] **Build** — `ng build` (prod) clean; `ng serve` clean; lint clean.
- [ ] **AI review passed** — the subagent review (§10) ran and all High/Medium findings are resolved or explicitly waived with reason.

> This checklist is also kept as a copy-paste PR template in `.github/PULL_REQUEST_TEMPLATE.md`.

---

## 9. Tailwind v4 + CSS (already resolved in scaffold)

`zard-cli init` **hard-requires `src/styles.css`** (Tailwind v4 is CSS-first; SCSS is not supported), so the project standardized on **CSS**:
- `src/styles.css` holds `@import "tailwindcss";`, `@plugin "tailwindcss-animate";`, `@custom-variant dark`, and the theme tokens (to be set to **Piramal blue/white**).
- `angular.json`: `styles → src/styles.css`, `inlineStyleLanguage → css`, schematics `style → css`.
- Prefer Tailwind utility classes in templates; component `.css` files stay mostly empty.

Init also produced: deps (`tailwindcss@4`, `@tailwindcss/postcss`, `tailwindcss-animate`, `postcss`, `class-variance-authority`, `clsx`, `tailwind-merge`, `@ng-icons/core`, `@ng-icons/lucide`, `@angular/cdk`); files `.postcssrc.json`, themed `styles.css`, `components.json`, ZardUI helpers under `shared/core/` + `cn()` in `shared/utils/merge-classes.ts`.

> **Watch-item:** `@ng-icons/*@33` declare a peer of `@angular/common >=21` while we're on 20; install resolves via `--legacy-peer-deps` and build + `ng serve` pass. If icon issues appear, pin `@ng-icons/*` to the latest Angular-20-compatible major.

---

## 10. AI subagent PR-review process (mentor point #5)

**Principle (the mentor's): Claude reviews code better than it writes it.** Every PR is reviewed by Claude Code subagents *before* a human review, with findings gating the merge. The author (Aarti) writes the code; the subagents adversarially check it.

### 10.1 The review pipeline (run on each feature PR)
For each PR, launch a panel of focused review subagents, each with a single lens, against the diff + the corresponding old-app component:

1. **Parity reviewer** — diff the new component against the old one; list any field, action, validation, or branch present in the old code but missing in the new. *Output: missing-parity list.*
2. **Anti-stub reviewer** — hunt for TODOs, stubs, dummy data, unwired handlers, dead code, fake responses (backs up the automated check in §8.1, but reasons about intent — e.g. a handler that "succeeds" without calling the API). *Output: stub findings.*
3. **API-contract reviewer** — for every HTTP call, confirm endpoint/method/body/response handling matches the old service and the API contract in §4 (envelope, privilege, headers). *Output: contract deviations.*
4. **Correctness/security reviewer** — bugs, RxJS/signal misuse, subscription leaks, auth/encryption regressions (PBKDF2-SHA512+AES kept as-is), XSS via `innerHTML`, token handling. *Output: correctness/security findings.*
5. **Convention reviewer** — AMRIT folder placement (§6), standalone/signals usage, ZardUI-only (no stray Material), i18n keys, accessibility, license header, naming. *Output: convention findings.*

Each reviewer returns structured findings with **severity (High / Medium / Low)**, file:line, and a concrete fix. Findings are **adversarially verified** — a finding is only "real" if it survives a second skeptical pass (avoids false positives wasting review time).

### 10.2 Gating rule
- **High** findings → must fix before merge.
- **Medium** → fix, or explicitly waive in the PR with a one-line justification the mentor can see.
- **Low** → tracked, batched.
- The §8.2 checklist box "AI review passed" is only ticked when the above holds.

### 10.3 How it's run
- For routine PRs: `/code-review` (or the `code-review` skill) on the branch diff, plus a parity prompt pointed at the old component.
- For large/critical PRs (auth, case-sheet, outbound): a multi-lens workflow that fans out the 5 reviewers above in parallel, dedups, verifies, and posts a single consolidated review. This is the "review > write" investment the mentor asked for, concentrated where risk is highest.
- The consolidated findings are pasted into the PR thread so the mentor sees what was checked and what was fixed.

---

## 11. Migration order (priority tiers)

Foundation before features; the call workflow is the product core and comes before the long tail of reports.

- **P0 — Foundation:** re-align scaffold to `app-modules/` convention + remove `core/tokens/`, add Common-UI submodule + `tracking`, set Piramal theme, environments/config (UAT), `HttpClient` + functional interceptors, `AuthStore`, session, loader, toast, routing skeleton, guards, no-stub check + PR template + subagent review wiring, reusable `DataTable` + `ConfirmDialog`.
- **P1 — Critical path:** auth → role selection → dashboard → **inbound call workflow** (registration → case sheet → closure) → outbound workflow.
- **P2 — Breadth:** SIO sub-services, supervisor reports & config.
- **P3 — Long tail / stretch:** misc reports (surveyor, covid, mental-health, medical-advise), notifications, training resources, full i18n parity, niche case-sheet variants.

> **Honest scope note:** 118 old components. A clean, tested rebuild of **P0 + P1 + a meaningful slice of P2** by mid-August is the realistic high-quality commitment for one intern; P3 is stretch / handoff-documented.

---

## 12. Week-by-week plan — migration complete by **mid-August**, **September = cleanup / deploy / QA**

| Week | Dates | Focus | Deliverable / PR |
|---|---|---|---|
| **0–1** | Jun 10–22 | Study repos, rewrite this plan, **install ZardUI + Tailwind**, Piramal theme, **re-align scaffold to `app-modules/` + remove `core/tokens/`**, Common-UI submodule, no-stub check + PR template + subagent-review wiring, base layout shell | `chore: foundation + AMRIT structure + ZardUI + review tooling` |
| **2** | Jun 23–29 | Core infra: environments/config (UAT), `HttpClient` + auth/error/loader interceptors, `AuthStore`, session, loader, toast, reusable `DataTable` + `ConfirmDialog`, `tracking` from Common-UI | `feat: core http + state + shared UI` |
| **3** | Jun 30–Jul 6 | **Auth:** login (PBKDF2/AES as-is), captcha, set-password, set-security-questions, reset-password | `feat: auth flows` |
| **4** | Jul 7–13 | **Role selection:** multi-role-screen, service-role-selection, role/route guards, privilege-driven routing; **shell + dashboard** | `feat: role selection + shell + dashboard` |
| **5** | Jul 14–20 | **Inbound 1:** call shell (router, no jQuery) + beneficiary-registration (RO) | `feat: beneficiary registration` |
| **6** | Jul 21–27 | **Inbound 2:** case-sheet + SNOMED chief-complaint search + CDSS/algo | `feat: case sheet + cdss` |
| **7** | Jul 28–Aug 3 | **Inbound 3:** prescription, closure, insert-complaint, screenings + **mid-project QA & demo** | `feat: prescription + closure` |
| **8** | Aug 4–10 | **Outbound:** worklist, outbound-call, allocate / search / reallocate | `feat: outbound workflow` |
| **9** | Aug 11–15 | **SIO services** (blood-on-call, organ-donation, food-safety, epidemic, grievance, information, scheme) + **migration feature-complete checkpoint (mid-August)** | `feat: sio services` |
| **10** | Aug 18–24 | **Cleanup:** supervisor reports/config + misc reports as time allows; resolve all Medium AI-review findings; remove dead code; finalize Common-UI extraction proposal | `feat/chore: supervisor + cleanup` |
| **11** | Aug 25–31 | **QA:** accessibility, responsive, i18n parity, cross-role smoke tests on UAT; full §8.2 checklist sweep across all modules | `chore: QA pass` |
| **12** | Sep 1–7 | **Deployment** prep (CI build, env templating), deploy to UAT, mentor + stakeholder review | `chore: deploy to UAT` |
| **13** | Sep 8–14 | **Testing & sign-off:** final cross-role testing, docs, handoff notes; merge `angular-zard-migration` → `main` | `chore: release + docs` |

*Migration (feature build) lands by ~Aug 15. September is reserved for cleanup, deployment, QA, and testing per mentor direction. Supervisor/long-tail reports are the buffer/stretch.*

---

## 13. Cross-cutting migration rules

1. **Preserve API contracts.** Endpoints, request/response shapes, `{response,error}` envelope, privilege object, JWT-cookie + `Authorization`, and the `5002` force-logout path stay identical. Each call diffed against the old service (§8.2).
2. **No jQuery, no Bootstrap carousel.** `innerpage` screen-switching becomes Angular Router.
3. **HTTP:** `provideHttpClient(withInterceptors([authInterceptor, errorInterceptor, loaderInterceptor]))`; port the 401/403/5002 logic from `http.securityinterceptor.ts`.
4. **State:** replace `dataService` subjects with focused signal stores.
5. **Forms:** template-driven `ngModel` → Reactive Forms; port custom validator directives.
6. **Security:** keep client-side password encryption (CryptoJS PBKDF2-SHA512 + AES) and sessionStorage token behaviour **as-is** (confirmed). Keep `SessionStorageService` encryption byte-compatible with MMU/1097.
7. **CTI:** preserve the existing **REST** `czentrix.service` integration unchanged (iframe wording flagged for mentor, §2.4).
8. **i18n:** same languages as the current app (confirmed); no hardcoded strings.
9. **Config:** all environment/config in `environments/` — **no `tokens/` folder** (confirmed).
10. **Each PR:** small, one feature/slice, passes the no-stub check + §8.2 checklist + §10 AI review before human review.
11. **QA gate:** nothing merges to `main` until the full cross-role QA pass (Sep).

---

## 14. Open items for mentor (Dr. Mithun James)

1. **CTI:** confirm whether an iframe softphone exists, or "iframe" was imprecise and the REST `czentrix.service` integration is the whole story (§2.4).
2. **Piramal theme palette:** confirm exact blue/white tokens with Madhav + Gopi.
3. **UAT access:** confirm UAT base URLs (`ip104`, `commonAPI`, `telephoneServer`) and test credentials for integration testing.
4. **Scope commitment:** confirm P0 + P1 + partial P2 by mid-August as the deliverable, P3 as stretch/handoff.

---

## 15. Next step

Plan review with mentor → on approval: re-align the scaffold to `app-modules/` and remove `core/tokens/`, add Common-UI as a submodule, set the Piramal theme, wire the no-stub check + PR template + subagent review, add the first ZardUI component set (Button, Card, Input, Form, Dialog, Table, Pagination), and open the **foundation PR** on `angular-zard-migration`.
