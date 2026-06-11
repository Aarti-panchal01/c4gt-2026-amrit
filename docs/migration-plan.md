# Migration Plan — AMRIT Helpline104-UI
> Angular 4.1.3 → Angular 20 + ZardUI  
> June 10 – September 10, 2026 (13 weeks)  
> Status: DRAFT — pending confirmation at Jun 15 planning meeting  
> Last updated: Jun 11, 2026

## Strategy & Assumptions

- **Approach:** Fresh-workspace re-platform, NOT incremental ng update
- **Why not ng update:** @angular/http removed (v5+), RxJS 5→6 break (v6+), 
  Material Md* removal (v9+), md2 dead, .angular-cli.json removed (v6+), 
  8 abandoned libs with no compatible intermediate version
- **Key efficiency:** Do UI layer once — Phase 1 gets app running on Angular 20 
  with only mechanical Md*→Mat* rename. Phase 2 replaces Material/md2/jQuery 
  with ZardUI. Avoids touching 138 templates twice.
- **Critical path:** HTTP layer (Wk2) → services (Wk3–5) → components (Wk6–9)
- **ZardUI caveat:** Verify component coverage in Wk1 (tables, datepicker, 
  autocomplete). Keep CDK primitives as fallback for any gaps.

> ⚠️ Pending Jun 15 confirmation:
> - Monorepo for 104 + 1097 (yes/no)?
> - Approach: two-phase upgrade OR fresh repo + copy features with ZardUI from day one?
> - Coordination with Madhav (1097 intern) on shared code?

---

## Phase 1 — Angular 4 → Angular 20 (Weeks 1–9, Jun 10 – Aug 11)

### Week 1 (Jun 10–16) — Foundation & Scaffolding
**Goal:** Working Angular 20 workspace + ZardUI coverage validated

- New Angular 20 workspace: bootstrapApplication, angular.json, esbuild, 
  Node 20.11+/22, TS 5.6+, Dart sass
- Replace tslint + codelyzer → angular-eslint + Prettier
- Port src/environments/* → fileReplacements in angular.json
- Clean polyfills.ts (drop core-js, keep zone.js 0.15)
- Port version.js postinstall hook (git-version.json injection)
- Install @angular/material@20 + @angular/cdk@20 (temporary, mechanical use)
- Stand up dual-build CI (old WAR build + new Angular 20 build)
- **Spike:** ZardUI component coverage — table, datepicker, dialog, tabs, 
  pagination, autocomplete. Document gaps in research/zardui-coverage.md

### Week 2 (Jun 17–23) — HTTP Layer + Core Infra ⚠️ HIGHEST RISK
**Goal:** All HTTP infrastructure on HttpClient + functional interceptors

**Files to replace:**
- http.interceptor.ts → functional HttpInterceptor
- http.securityinterceptor.ts → functional HttpInterceptor (silent mode)
- http.factory.ts → deleted
- http.security.factory.ts → deleted

**What to build:**
- provideHttpClient(withInterceptors([appInterceptor]))
- HttpContext tokens: IS_SILENT_REQUEST (replaces two-class distinction)
- Loader logic: add request ref-counting (fix concurrent-spinner bug)
- 401/403 handling: alert + sessionStorage.clear() + redirect
- statusCode 5002 handling: confirm-dialog + BehaviorSubject force-logout 
  (must preserve exactly — concurrent-login flow depends on this)
- Remove console.error("authTkn", ...) — currently logs auth token every request

**Other services this week:**
- ConfigService, sessionStorageService, AuthService, dataService
- ListnerService, OutboundListnerService, ConfirmationDialogsService, LoaderService
- Establish RxJS 7 pipe(map(), catchError()) helper pattern

### Week 3 (Jun 24–30) — Services Wave 1 (Auth/Common/Core)
**Files:** loginService, authentication, authGuardService, common/*, 
captcha-service, http-services, dialog

**Per service:**
- @angular/http → HttpClient
- Remove .json() calls (HttpClient auto-parses)
- URLSearchParams → HttpParams
- Run rxjs-5-to-6-migrate codemod, hand-fix .pipe() sites
- Convert guards → functional CanActivateFn / CanDeactivateFn

### Week 4 (Jul 1–7) — Services Wave 2 (Clinical/Call)
**Files:** callservices, caseSheetService, cdssService, snomedService,
prescriptionServices, covidService, screening, searchBeneficiaryService,
register-services, czentrix, socketService

**Special fixes:**
- register-services: hardcoded 10.152.3.152:1040 → ConfigService
- socketService: make @Injectable, fix hardcoded 10.208.122.38:4000 → ConfigService

### Week 5 (Jul 8–14) — Services Wave 3 + Directives/Pipes
**Files:** sioService/*(7 services), supervisorServices/*(6), 
outboundServices/*(4), report-services, callTypeReports, surveyorServices,
coService/*, notificationService, adminServices/*(4), upload-services

**Special fixes:**
- adminServices: hardcoded localhost:8080 → ConfigService

**Also this week:**
- Migrate directives/ (password, mobile, email, name, address) → standalone: true
- Migrate pipes (order-by.pipe, utc-date.pipe, custom-pipe) → standalone: true
- Replace ng2-validation + ng2-custom-validation → built-in Validators

**🏁 Milestone: All 65 services + directives/pipes compile on Angular 20**

### Week 6 (Jul 15–21) — Auth/Shell Components + Routing
**Files:** login, resetPassword, set-password, set-security-questions, 
captcha, multi-role-screen, service-role-selection, dashboard-navigation,
dashboard-row-header, dashboard-user-id, loader, app.component

**Also this week:**
- Replace RouterModule.forRoot (531-line app.module.ts) → provideRouter + 
  lazy loadComponent routes
- Wire three functional guards (AuthGuard, AuthGuard2, SaveFormsGuard)
- Mechanical Md*→Mat* / md-*→mat-* rename only (no visual work)

**🏁 Milestone: Login → role select → dashboard works on Angular 20**

### Week 7 (Jul 22–28) — Core Call-Flow Components
**Files:** dashboard, innerpage, 104 role switchboard, 104-ro, 
beneficiary-registration-104, closure

**Note:** These are jQuery-heavy (104-ro/hao/co ≈ 60+ jQuery calls each).
Do minimal jQuery stabilization to run. Defer carousel/tabs/DOM rework to 
Phase 2. Scope CUSTOM_ELEMENTS_SCHEMA per-component and fix unknown-element 
errors it surfaces.

### Week 8 (Jul 29–Aug 4) — Case-Sheet + CDSS + Role Components
**Files:** case-sheet (+ all modals: cdssModal, case-sheet-covid-modal, 
case-sheet-comp-modal, history, recentPrescription), algo-component, cdss,
104-co, 104-mo, 104-hao, 104-pd, 104-counsellor, 104-consent/consent-form,
prescription, schedule-appointment

**Note:** CDSS → SNOMED → COVID chain is the densest logic in the app.

### Week 9 (Aug 5–11) — Remaining Features + Phase 1 Hardening
**Files:**
- SIO: sio-services, sio-blood-on-call, organ-donation, epidemic-outbreak,
  food-safety, scheme/upload-schemes, bal-vivah, grievience, information
- Outbound: outbound-worklist, outbound-allocate/search-records, 
  reallocate-calls, dial-beneficiary, agent-outboundcall
- Supervisor: all supervisor-*-report, notifications, quality-audit, 
  sms-template, configurations, emergency-contacts, force-logout
- Misc: surveyor, covid-19, directory-services, knowledge-management, 
  news/alerts, dashboard-reports

**Also this week:**
- Swap abandoned libs: angular2-toaster → ngx-toastr, 
  angular2-csv → export-to-csv, upgrade ngx-pagination
- Keep ng2-smart-table as interim (replaced in Phase 2)
- Drop hammerjs (0 usages in codebase)
- Remove CUSTOM_ELEMENTS_SCHEMA; fix surfaced unknown-element errors
- Rewrite ~20 core specs against HttpClientTestingModule + standalone TestBed
- Ticket remaining 90+ specs for later
- Playwright smoke tests: login → call → closure

**🏁 Phase 1 Milestone: Full app runs on Angular 20. No @angular/http. No RxJS 5.**

---

## Phase 2 — ZardUI Integration (Weeks 10–13, Aug 12 – Sep 10)

### Week 10 (Aug 12–18) — ZardUI + Tailwind Foundation
- Install + configure Tailwind CSS (content globs over src/, preflight 
  tuned to coexist with Bootstrap 3)
- ZardUI CLI + base setup
- Define design tokens to match current bootstrap-phibonacci/theme.scss palette
- Build shared UI-kit wrapper layer: button, input, select, form-field, dialog,
  toast, table, pagination, tabs, datepicker, card
- Re-point ConfirmationDialogsService → ZardUI dialog
- Kill md2 first (391 refs / 74 files — no Material equivalent anyway):
  datepicker/select/autocomplete/chips → ZardUI

### Week 11 (Aug 19–25) — ZardUI Rollout Wave 1 (Shell + Call Flow)
**Files:** login, multi-role-screen, dashboard, innerpage, 104-ro, 
beneficiary-registration-104, closure, case-sheet core → ZardUI

- Replace jQuery Bootstrap carousel/tabs in 104-co/104-hao/104-sio 
  with ZardUI tabs/stepper (eliminates largest jQuery clusters)
- ng2-smart-table → ZardUI table where used

### Week 12 (Aug 26–Sep 1) — ZardUI Rollout Wave 2 (Remaining Features)
**Files:** SIO services, outbound, supervisor reports (heavy tables + 
datepickers + CSV export), surveyor, prescription, notifications, 
sms-template, quality-audit

- Once last component converted: remove jQuery, Bootstrap 3, tether, 
  @angular/material, @angular/cdk (if unused), md2 from package.json
- Tailwind purge; delete legacy per-component CSS covered by utilities

### Week 13 (Sep 2–10) — Hardening, QA, Cutover
- Full visual-regression + a11y + responsive pass across all ~138 screens
- Expand Playwright e2e to all role flows
- Bundle-size + lazy-route check
- Security cleanups: unencrypted authToken/apiman_key, empty encKey, 
  always-true isAuthenticated()
- Production build, WAR packaging parity, staging deploy, sign-off
- Reserve ~2 days buffer for slippage

**🏁 Phase 2 Milestone: QA complete, build parity, final sign-off**

---

## Risk Register

| Risk | Severity | Week | Mitigation |
|------|----------|------|------------|
| Interceptor 5002/force-logout semantics | Critical | Wk2 | Pair-program; test concurrent-login early |
| jQuery × ZardUI DOM collisions | High | Wk11 | Deferred by design; Playwright catches regressions |
| ZardUI table gap | High | Wk1 | Spike Wk1; CDK fallback budgeted |
| CUSTOM_ELEMENTS_SCHEMA removal | Medium | Wk9 | Budget 1–2 days for fixes |
| Near-zero test safety net | High | All | Protect 20 core specs minimum |
