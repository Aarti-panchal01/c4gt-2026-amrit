# Helpline104-UI → Angular 20 + ZardUI Migration Plan

> **Project:** Migrate AMRIT Helpline 104-UI from Angular 4.1.3 to Angular 20 + ZardUI
> **Approach:** Fresh repo — rebuild feature-by-feature, **not** an in-place upgrade
> **Author:** Aarti Panchal (C4GT 2026 intern, Piramal Swasthya / AMRIT)
> **Mentor:** Dr. Mithun James
> **Old repo:** https://github.com/PSMRI/Helpline104-UI (Angular 4.1.3)
> **New repo:** https://github.com/PSMRI/Helpline104-UI-NEXT (Angular 20, scaffolded)
> **Timeline:** 10 Jun 2026 → 10 Sep 2026 (~13 weeks)
> **Working branch:** `angular-zard-migration` → small PRs → merge to `main` only after full QA

---

## 1. Why a fresh repo (not an upgrade)

The old app is **Angular 4.1.3** — six major architecture generations behind Angular 20. A direct upgrade is not realistic because nearly every foundational API the old code relies on has been removed or replaced:

| Old (Angular 4) | Angular 20 |
|---|---|
| `@angular/http` (`Http`, `XHRBackend`, `RequestOptions`) | `HttpClient` + functional interceptors |
| NgModules (`app.module.ts` is ~870 lines, one giant module) | Standalone components, no NgModule |
| `@angular/material` **2.0.0-beta.11** + `md2 0.0.18` (the `md-*` API) | Removed entirely → **ZardUI** |
| RxJS 5.4 (`.map()`, `rxjs/Rx`, `Observable.merge`) | RxJS 7.8 (`.pipe(map())`) + **Signals** |
| Template-driven `ngModel` everywhere | Reactive forms / Signals |
| Zone.js change detection | **Zoneless** (`provideZonelessChangeDetection`) |
| jQuery + Bootstrap carousels for navigation | Angular Router + ZardUI |

Rebuilding lets us drop jQuery, the carousel-based screen switching, the deprecated HTTP stack, and the `md-*`/`md2` widget zoo in one clean pass while preserving the **API contracts and business logic** (which stay valid — only the UI/runtime layer changes).

---

## 2. What the old app does (functional map)

AMRIT Helpline 104 is a **health helpline call-center application**. Agents in different roles handle inbound and outbound calls, register beneficiaries, fill medical case sheets, and close calls. Supervisors run reports and configure the system.

### 2.1 Roles

| Code | Role | Responsibility |
|---|---|---|
| **RO** | Registration Officer | Beneficiary registration, search, eligibility, initial inbound handling |
| **HAO** | Health Advice Officer | Health advice, counseling, lifestyle guidance |
| **MO** | Medical Officer | Medical assessment, prescription, CDSS, clinical review |
| **CO** | Complaint Officer | Complaints, referrals, feedback, categorization |
| **Counsellor** | Health Counsellor | Detailed counseling, behaviour-change communication |
| **PD** | Program Data | Data validation, reporting, QA |
| **SIO** | Special Information Officer | Blood-on-call, epidemic, food safety, grievance, organ donation, schemes, information |
| **Supervisor** | Supervisor | Agent monitoring, quality audit, reports, config, alerts, training |
| **Surveyor** | Field Surveyor | Survey data, call-type reports |

A user can hold **multiple roles**; after login they pick a service + role on the **multi-role screen**, which determines which screens are shown (privileges come from `response.previlegeObj[]`, filtered to service `"104"`).

### 2.2 Core workflows

**Inbound call pipeline:**
`Incoming call (Czentrix CTI)` → `Beneficiary registration (RO)` → `Case sheet (HAO/MO/Counsellor)` → `Closure (call type / sub-type / followup)`

**Outbound pipeline:**
`Worklist (pre-allocated by role)` → `Select record` → `Outbound call + case sheet` → `Closure` (with allocate / search / reallocate tooling)

### 2.3 Feature inventory (old repo: 118 components, 60 services)

- **Auth:** login (PBKDF2-SHA512 + AES client-side encryption, optional captcha), set-password, set-security-questions, reset-password
- **Shell / dashboard:** multi-role-screen, service-role-selection, dashboard, dashboard-navigation, dashboard-row-header, dashboard-user-id, innerpage (carousel host)
- **Inbound:** beneficiary-registration-104, case-sheet (+ covid/cancer/general/mcts/mmu variants), cheif-complaint-snomed-search, cdss / algo-component, prescription, closure, insert-complaint, bp-screening, diabetic-screening, schedule-appointment, consent
- **Outbound:** outbond-worklist, agent-outbondcall, outbound-allocate-records, outbound-search-records, reallocate-calls, dial-beneficiary
- **SIO sub-services:** blood-on-call, epidemic-outbreak, food-safety, grievience, information, organ-donation, scheme, sio-services-history, sio-outbound-provider
- **Supervisor (~18):** call-summary / call-quality / calltype / complaint-detail / district-wise-call-volume / diseases-summary / quality / unblock-user reports; configurations, upload-schemes, alerts-notifications, notifications, emergency-contacts, training-resources, location-communication, blood-url, grievance; agent-status, block-unblock-number, force-logout, quality-audit
- **Other reports:** medical-advise-report, mental-health-report, blood-on-call-detailed-report, surveyor-calltype-reports, covid-19, bal-vivah
- **Shared:** loader, captcha, common-dialog, message-dialog, notifications-dialog, edit-notifications, rating, custom-pipe (search filter), order-by.pipe, utc-date.pipe, directives (validators)
- **Integrations:** Czentrix CTI (iframe + service), Socket.io (real-time), SNOMED-CT coding, CDSS engine, i18n (`assets/i18n`)

### 2.4 Services layer (key ones)

`loginService`, `dataService` (central RxJS-subject state store), `AuthService`, `ConfigService`, `HttpServices`, `SearchService`, `RegisterService`, `CaseSheetService`, `CallServices`, `PrescriptionService`, `CDSSService`, `SnomedService`, `LocationService`, `SocketService`, `SessionService`, `LoaderService`, the four `Outbound*` services, six `sio*` services, ~10 `supervisor*` services, plus the HTTP plumbing (`http.interceptor.ts`, `http.securityinterceptor.ts`, `http.factory.ts`, `http.security.factory.ts`).

> **Migration note:** `dataService` is a god-object using `Subject`/`BehaviorSubject` for cross-component state. In Angular 20 this becomes **small, focused signal stores** (e.g. `AuthStore`, `CallStore`, `RoleStore`), and the two HTTP interceptor classes collapse into **functional `HttpInterceptorFn`s** registered with `provideHttpClient(withInterceptors([...]))`.

---

## 3. ZardUI: what it is and what it gives us

[ZardUI](https://zardui.com/) is the shadcn/ui equivalent for Angular: beautifully designed, accessible, **standalone, signal-based, OnPush, zoneless-ready** components built on **TypeScript + TailwindCSS v4 + Class Variance Authority (CVA)**. It is **free and open source**.

- **Requirements:** Angular 19+ (our scaffold is 20.3 ✅), Node 20/22, **TailwindCSS v4**.
- **Distribution model (shadcn-style):** the CLI **copies component source into our repo** (we own the code), it is not a black-box npm dependency.
- **Important constraint:** ZardUI styles via **Tailwind v4 utility classes**, not SCSS component stylesheets. (See §6 for how we reconcile this with our SCSS scaffold.)

### Available components (47)

**Form & input:** Button, Input, Input Group, Form, Checkbox, Radio, Select, Combobox, Switch, Slider, Calendar, Date Picker
**Layout & nav:** Accordion, Breadcrumb, Menu, Tabs, Divider, Resizable
**Overlays:** Dialog, Alert Dialog, Sheet, Popover, Tooltip, Dropdown, Command
**Feedback:** Alert, Toast, Progress Bar, Loader, Skeleton, Badge, Empty
**Display:** Avatar, Card, Table, Icon
**Misc:** Toggle, Toggle Group, Segmented, Pagination

---

## 4. ZardUI component mapping (old `md-*` / `md2` / 3rd-party → ZardUI)

| Old widget | ZardUI equivalent | Notes |
|---|---|---|
| `md-button`, `md-raised-button` | **Button** | variants via CVA |
| `md-input-container` + `mdInput` | **Input** + **Form** / **Input Group** | reactive forms |
| `md-select` / `md-option` | **Select** | searchable → **Combobox** |
| `md-radio-group` / `md-radio-button` | **Radio** | |
| `md-checkbox` | **Checkbox** | |
| `md-slide-toggle` | **Switch** | |
| `md-datepicker`, md2 datepicker | **Date Picker** / **Calendar** | |
| `md-dialog` (+ common-dialog, message-dialog) | **Dialog** / **Alert Dialog** | one reusable `ConfirmDialog` wrapper |
| `md-tab-group` / `md-tab` | **Tabs** | |
| `md-toolbar` | layout div + Tailwind | no 1:1; build header in `layout/` |
| `md-sidenav` | **Sheet** (mobile) + **Menu** (desktop sidebar) | |
| `md-icon` (material-design-icons) | **Icon** (lucide) | |
| `md-chips` | **Badge** (+ small custom chip) | |
| `md-tooltip` / `mdTooltip` | **Tooltip** | |
| `md-menu` | **Dropdown** / **Menu** | |
| `md-card` | **Card** | |
| `md-list` | **Menu** / Tailwind list | |
| `md-expansion-panel` | **Accordion** | |
| `md-progress-spinner`, loader/ | **Loader** / **Progress Bar** / **Skeleton** | |
| `md-snack-bar`, `angular2-toaster` | **Toast** | |
| `md-stepper` (`MatStepperModule`) | **Tabs** or **Segmented** + custom step state | ZardUI has no stepper |
| `md-button-toggle` | **Toggle Group** / **Segmented** | |
| `md-grid-list` | Tailwind CSS grid | |
| `ng2-smart-table` | **Table** + **Pagination** | build one reusable `DataTable` wrapper (sort/filter/paginate) — used widely in reports |
| `ngx-pagination` | **Pagination** | |
| `angular2-csv`, `xlsx`, `file-saver` | keep as **utilities** (export logic) | not UI; wrap in `shared/utils/export.ts` |
| `captcha` (reCAPTCHA) | keep custom component | not a ZardUI concern |
| jQuery carousel (innerpage) | Angular Router + **Tabs**/**Segmented** | remove jQuery entirely |

> The single highest-leverage shared component to build early is a **reusable `DataTable`** (Table + Pagination + sort/filter), because the supervisor/reports area depends on it dozens of times.

---

## 5. Proposed folder structure (Angular 20 standalone)

Feature-first, lazy-loaded. Each feature owns its routes and is loaded with `loadChildren` / `loadComponent`.

```
src/
├─ app/
│  ├─ core/                       # app-wide singletons (provided once)
│  │  ├─ auth/
│  │  │  ├─ auth.service.ts
│  │  │  ├─ auth.store.ts         # signal store (token, user, current role)
│  │  │  ├─ auth.guard.ts         # functional CanActivateFn
│  │  │  └─ role.guard.ts
│  │  ├─ http/
│  │  │  ├─ auth.interceptor.ts   # HttpInterceptorFn (adds Authorization)
│  │  │  ├─ error.interceptor.ts  # 401/403, global error handling
│  │  │  └─ loader.interceptor.ts # show/hide global loader
│  │  ├─ services/
│  │  │  ├─ config.service.ts     # base URLs / environment
│  │  │  ├─ session.service.ts    # timeout / keepalive
│  │  │  ├─ socket.service.ts     # Socket.io real-time
│  │  │  ├─ location.service.ts   # state/district/block/village
│  │  │  └─ cti.service.ts        # Czentrix CTI
│  │  ├─ models/                  # shared TS interfaces / API DTOs
│  │  └─ tokens/                  # InjectionTokens
│  │
│  ├─ shared/                     # reusable, no business logic
│  │  ├─ ui/                      # ← ZardUI components land here (per components.json alias)
│  │  ├─ components/              # app-level reusable: data-table, page-header, confirm-dialog, file-export
│  │  ├─ pipes/                   # order-by, utc-date, search-filter
│  │  ├─ directives/              # validators (name, mobile, email, password strength)
│  │  └─ utils/                   # export (csv/xlsx), crypto, date helpers
│  │
│  ├─ layout/                     # app shell: header, sidebar nav, shell component
│  │
│  ├─ features/
│  │  ├─ auth/                    # login, set-password, reset-password, security-questions, captcha
│  │  ├─ role-selection/          # multi-role-screen, service-role-selection
│  │  ├─ dashboard/
│  │  ├─ call/                    # INBOUND: innerpage shell, beneficiary-registration,
│  │  │                           #          case-sheet, cdss/algo, snomed-search,
│  │  │                           #          prescription, closure, insert-complaint, screenings
│  │  ├─ outbound/                # worklist, outbound-call, allocate, search, reallocate
│  │  ├─ sio/
│  │  │  ├─ blood-on-call/
│  │  │  ├─ organ-donation/
│  │  │  ├─ food-safety/
│  │  │  ├─ epidemic-outbreak/
│  │  │  ├─ grievance/
│  │  │  ├─ information/
│  │  │  └─ scheme/
│  │  ├─ supervisor/
│  │  │  ├─ reports/
│  │  │  └─ config/
│  │  └─ reports/                 # surveyor, covid, mental-health, medical-advise, blood-on-call
│  │
│  ├─ app.config.ts               # providers (router, http, zoneless)
│  ├─ app.routes.ts               # top-level lazy routes
│  └─ app.ts                      # root component (shell outlet)
│
├─ environments/                  # environment.ts / environment.prod.ts
└─ styles.css                    # global + Tailwind v4 entry (see §6)

docs/
└─ migration-plan.md              # this file
```

**Conventions**
- Standalone components only; no NgModules.
- Signals for component state; `inject()` over constructor DI where natural.
- Reactive Forms for all forms.
- One feature = one folder + one `*.routes.ts`, lazy-loaded.
- `core/` provided once at root; `shared/` is import-only (no providers).

---

## 6. Tailwind v4 + SCSS reconciliation — RESOLVED (switched to CSS)

The scaffold was generated with **SCSS** (`inlineStyleLanguage: scss`, `src/styles.scss`, per-component `.scss`). The `zard-cli init` command **hard-requires `src/styles.css`** (it refuses to run without it) — ZardUI/Tailwind v4 is CSS-first and does not support SCSS.

**Decision made during setup:** standardize the project on **CSS**, not SCSS. Concretely:
- `src/styles.scss` → `src/styles.css` (now holds `@import "tailwindcss";`, `@plugin "tailwindcss-animate";`, the `@custom-variant dark`, and the Neutral oklch theme tokens that init generated).
- `src/app/app.scss` → `src/app/app.css`; `styleUrl` updated.
- `angular.json`: `styles` → `src/styles.css`, `inlineStyleLanguage` → `css`, schematics `style` → `css` (so `ng g component` emits `.css`).
- **Prefer Tailwind utility classes in templates** for layout/spacing/color (the ZardUI way); component `.css` files stay mostly empty.

> This supersedes the original "keep `.scss`" recommendation — the CLI made CSS mandatory, and CSS is the correct long-term choice for a Tailwind-v4 project.

### 6.1 What `zard-cli init` produced
- Deps: `tailwindcss@4`, `@tailwindcss/postcss`, `tailwindcss-animate`, `postcss`, `class-variance-authority`, `clsx`, `tailwind-merge`, `@ng-icons/core`, `@ng-icons/lucide`, `@angular/cdk`.
- Files: `.postcssrc.json`, themed `src/styles.css`, `components.json`, and ZardUI internal helpers under `src/app/shared/core/` (`string-template-outlet`, event-manager plugins, `providezard`) + `cn()` in `src/app/shared/utils/merge-classes.ts`.
- `components.json` aliases were tuned: `components → @/shared/ui` (ZardUI primitives, kept separate from app-level composites in `@/shared/components`), `utils → @/shared/utils`, `core → @/shared/core`, `services → @/shared/services`. These aliases only drive the ZardUI CLI codegen; app code resolves `@/*` → `src/app/*` via tsconfig.

### 6.2 Known issue — `@ng-icons` version skew
`@ng-icons/core@33` / `@ng-icons/lucide@33` declare a peer of `@angular/common >=21`, while we are on Angular 20. Install resolves via `--legacy-peer-deps` and **the build + `ng serve` both pass cleanly**, but this is a watch-item: if icon issues appear, pin `@ng-icons/*` to the latest Angular-20-compatible major (≈ v31/v32) in `package.json`.

---

## 7. Migration order (priority tiers)

Foundation before features; the call workflow is the product's core and comes before the long tail of reports.

- **P0 — Foundation (must exist before any feature):** ZardUI + Tailwind install, theme, layout shell, config/environment, `HttpClient` + functional interceptors, `AuthStore`, session, loader, toast, routing skeleton, guards, reusable `DataTable` + `ConfirmDialog`.
- **P1 — Critical path:** auth → role selection → dashboard → **inbound call workflow** (registration → case sheet → closure) → outbound workflow.
- **P2 — Breadth:** SIO sub-services, supervisor reports & config.
- **P3 — Long tail / stretch:** misc reports (surveyor, covid, mental-health, medical-advise), notifications, training resources, i18n parity, niche case-sheet variants.

> **Honest scope note:** the old app has 118 components. A clean, well-tested rebuild of **P0 + P1 + a meaningful slice of P2** is a realistic, high-quality outcome for one intern in 13 weeks. P3 is explicitly stretch / handoff-documented, not a commitment.

---

## 8. Week-by-week plan (10 Jun – 10 Sep 2026)

| Week | Dates | Focus | Deliverable / PR |
|---|---|---|---|
| **0–1** | Jun 10–16 | Study old repo, write this plan, **install ZardUI + Tailwind v4**, theme, base layout shell, folder structure | `chore: foundation + ZardUI setup` |
| **2** | Jun 17–23 | Core infra: environments/config, `HttpClient` + auth/error/loader interceptors, `AuthStore`, session, loader, toast, reusable `DataTable` + `ConfirmDialog` | `feat: core http + state + shared UI` |
| **3** | Jun 24–30 | **Auth**: login (PBKDF2/AES encryption), captcha, set-password, set-security-questions, reset-password | `feat: auth flows` |
| **4** | Jul 1–7 | **Role selection**: multi-role-screen, service-role-selection, role/route guards, privilege-driven routing | `feat: role + service selection` |
| **5** | Jul 8–14 | **Shell + dashboard**: navigation/sidebar, header, dashboard widgets, dialogs/notifications | `feat: app shell + dashboard` |
| **6** | Jul 15–21 | **Inbound 1**: innerpage shell (router-based, no jQuery) + beneficiary-registration (RO) | `feat: beneficiary registration` |
| **7** | Jul 22–28 | **Inbound 2**: case-sheet + SNOMED chief-complaint search + CDSS/algo | `feat: case sheet + cdss` |
| **8** | Jul 29–Aug 4 | **Inbound 3**: prescription, closure, insert-complaint, screenings + **mid-project QA & demo** | `feat: prescription + closure` |
| **9** | Aug 5–11 | **Outbound**: worklist, outbound-call, allocate / search / reallocate | `feat: outbound workflow` |
| **10** | Aug 12–18 | **SIO services**: blood-on-call, organ-donation, food-safety, epidemic, grievance, information, scheme | `feat: sio services` |
| **11** | Aug 19–25 | **Supervisor** reports (call-summary, calltype, quality, complaint, district, disease) + DataTable reuse | `feat: supervisor reports` |
| **12** | Aug 26–Sep 1 | **Supervisor config** + misc reports (surveyor, covid, mental-health, medical-advise) + notifications/training | `feat: supervisor config + reports` |
| **13** | Sep 2–10 | **Full QA**: accessibility, responsive, i18n, cross-role smoke tests, docs; final review → merge `angular-zard-migration` → `main` | `chore: QA + docs + release` |

*(Weeks slip; reports in 11–12 and the P3 long tail are the buffer/stretch if earlier weeks run long.)*

---

## 9. Cross-cutting migration rules

1. **Preserve API contracts.** Endpoints, request/response shapes, and auth/encryption behaviour stay identical — only the client UI/runtime changes. Verify each migrated call against the old service.
2. **No jQuery, no Bootstrap carousel.** Screen switching becomes Angular Router + Tabs/Segmented.
3. **HTTP:** `provideHttpClient(withInterceptors([authInterceptor, errorInterceptor, loaderInterceptor]))`. Port the 401/403 re-auth logic from `http.securityinterceptor.ts`.
4. **State:** replace `dataService` subjects with focused signal stores.
5. **Forms:** template-driven `ngModel` → Reactive Forms; port the custom validator directives.
6. **Security:** keep client-side password encryption (CryptoJS PBKDF2-SHA512 + AES) and token-in-`sessionStorage` behaviour; confirm with mentor before changing.
7. **Each PR:** small, one feature/slice, builds clean, includes the ZardUI mapping notes for any screen migrated.
8. **QA gate:** nothing merges to `main` until the full cross-role QA pass at the end.

---

## 10. Open questions for mentor (Dr. Mithun James)

1. ~~Global styles~~ — RESOLVED: switched to `styles.css` (zard-cli hard-required it, CSS is correct for Tailwind v4).
2. ZardUI theme — default **Neutral**, or a Piramal/AMRIT brand palette?
3. Is a backend/API instance available for local integration testing during the rebuild?
4. Keep the existing client-side password encryption scheme as-is?
5. Czentrix CTI: same iframe integration, or is a newer telephony approach planned?
6. i18n: which languages must reach parity for the migration to be considered complete?
7. Confirm scope expectation: P0+P1+partial P2 as the committed deliverable, P3 as stretch?

---


## 11. Current status

Foundation is complete and committed locally:
- Commit: `4875251` on branch `angular-zard-migration`
- Angular 20 scaffold + ZardUI + Tailwind v4 + 8 starter components + folder structure — all verified (build passes, ng serve works)

**Next:** Plan review + approval from Dr. Mithun James + Sneha Unki → write access to PSMRI/Helpline104-UI-NEXT → push foundation PR → start Week 2 (core HTTP interceptors + AuthStore + reusable DataTable).
