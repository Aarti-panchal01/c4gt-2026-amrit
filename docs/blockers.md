# Blockers — C4GT 2026 AMRIT Helpline104-UI

This document tracks known blockers, open questions, and workarounds discovered during the migration.

---

## Active Blockers

### B1: ZardUI Table Component Coverage — Unclear

**Severity:** High | **Discovered:** Day 1 | **Status:** 🚧 Spike needed (Wk1)

**Issue:**
ZardUI's table component may lack built-in sorting, filtering, pagination. Our app has heavy table usage (supervisor reports, outbound worklist, call logs).

**Current Understanding:**
- Basic ZardUI table renders data; sorting/filtering/pagination API unclear
- Fallback: CDK DataSource + custom sorting, or keep `ng2-smart-table` as interim

**Next Steps:**
1. Week 1 spike: test ZardUI table with sort/filter/page
2. If insufficient: evaluate ng2-smart-table compatibility with Angular 20 OR build CDK wrapper
3. Document findings in `research/zardui-component-coverage.md`

**Mitigation:**
- Budget 3–4 days in Wk11 for table-component solution (Phase 2)
- Have CDK + custom sort/filter as fallback

---

### B2: jQuery × ZardUI DOM Collisions — High Risk

**Severity:** High | **Discovered:** Day 1 (codebase analysis) | **Status:** 🔴 Defer to Phase 2

**Issue:**
~440 jQuery calls across 20+ components (104-sio ×70, 104-hao ×64, 104-co ×61) directly manipulate DOM. ZardUI components may have different DOM structure than current Material/Bootstrap; jQuery selectors could break.

**Current Understanding:**
- jQuery clusters in 104-* role components; heavy carousel/tabs/modal manipulation
- Phase 1 defers jQuery refactor (Decision D6); Phase 1 leaves jQuery untouched
- Phase 2 Wk11 is the scheduled fix (swap jQuery tabs/carousel for ZardUI tabs/stepper)

**Next Steps:**
1. Phase 1: keep jQuery as-is; flag any layout regressions in weekly logs
2. Phase 2 Wk7 (Wk11 total): refactor 104-co/hao/sio jQuery to ZardUI components
3. Test in staging before cutover

**Mitigation:**
- Already deferred by design; Wk11 is scheduled for this
- Will use Playwright e2e to catch regressions

---

### B3: CUSTOM_ELEMENTS_SCHEMA Removal Will Surface Unknown-Element Errors

**Severity:** Medium | **Discovered:** Day 1 (codebase analysis) | **Status:** 🔴 Defer to Phase 1 end (Wk9)

**Issue:**
Current codebase uses `CUSTOM_ELEMENTS_SCHEMA` globally in `app.module.ts` to suppress "unknown element" errors. When removed, latent template bugs (typos, malformed selectors, old component names) will surface.

**Current Understanding:**
- Schema currently masks these errors
- Phase 1 Wk9 (end of component migration) is when we remove it
- Expect 5–20 errors; will need to fix each (rename/remove stray selectors)

**Next Steps:**
1. Phase 1 Wk9: scope `CUSTOM_ELEMENTS_SCHEMA` per-component (if needed for CDK)
2. Try removing it; collect errors
3. Fix unknown-element errors one by one

**Mitigation:**
- Budget 1–2 days in Wk9 for fixes
- Will likely be simple renames/removals; not complex logic

---

### B4: Hardcoded URLs in 6 Services (AdminLanguage, AdminUser, Socket, Register, etc.)

**Severity:** Medium | **Discovered:** Day 1 (codebase analysis) | **Status:** 🟡 Ticket for Phase 1 Wks 3–5

**Issue:**
6 services hardcode base URLs (localhost:8080, 10.152.3.152:1040, 10.208.122.38:4000) instead of using ConfigService. Breaks environment-driven config.

**Affected Services:**
- AdminLanguageService, AdminRoleService, AdminScreenService, AdminServicemasterService → localhost:8080
- AdminUserService, RegisterService → 10.152.3.152:1040
- SocketService → 10.208.122.38:4000

**Current Understanding:**
- ConfigService already supports these via `environment.ts` → `fileReplacements`
- Need to add entries if not present, then refactor each service

**Next Steps:**
1. Phase 1 Wks 3–5: during services migration, fix each hardcode
2. Move URL to ConfigService
3. Test in dev + staging

**Mitigation:**
- Straightforward find-replace
- Part of normal services-migration work; no extra task

---

### B5: AuthService.isAuthenticated() Always Returns `true`

**Severity:** High (Security) | **Discovered:** Day 1 (codebase analysis) | **Status:** 🟡 Security review + fix planned

**Issue:**
`AuthService.isAuthenticated()` currently returns `true` unconditionally (real check is commented out). Guards rely on checking sessionStorage for authToken instead. Not a blocker but a known weakness.

**Current Understanding:**
- Real bearer is authToken in sessionStorage (unencrypted)
- apiman_key also in sessionStorage (unencrypted)
- sessionStorageService tries to encrypt, but encKey = '' (trivially reversible)
- isAuthenticated() is dead code

**Next Steps:**
1. Phase 1 Wk3 (auth flow refactor): document this as-is (don't fix, preserve behavior)
2. Phase 2 or post-handover: security review + proper authToken handling (e.g., HttpOnly cookie, if architecture allows)

**Mitigation:**
- Not a blocker for migration; preserve current behavior to avoid breaking things
- Note for post-internship security hardening

---

### B6: Force-Logout Flow (statusCode 5002) Must Preserve Exactly

**Severity:** Critical | **Discovered:** Day 1 (codebase analysis) | **Status:** 🟡 Implement + validate in Wk2

**Issue:**
Current interceptors handle statusCode 5002 (force-logout) with a confirm-dialog + BehaviorSubject that the login component subscribes to. Functional interceptor rewrite must preserve this exactly, or concurrent-login flows will break.

**Current Understanding:**
- Triggered when user logs in from another location
- Backend returns 5002; interceptor shows confirm dialog → user clicks OK → doLogout=true → re-auth
- BehaviorSubject broadcast to login component drives the flow
- Exact semantics: if user denies, request should not be retried automatically

**Next Steps:**
1. Week 2: When rewriting HTTP layer, implement 5002 handling in functional interceptor with same UX
2. Test with manual concurrent-login scenario (open app in two browser tabs, log in from second tab)
3. Verify login component receives correct signals

**Mitigation:**
- Pair-program this with someone experienced in auth flows
- Test early + often (don't batch test to end of week)

---

## B7 — GitHub write access to PSMRI/Helpline104-UI-NEXT
**Status: RESOLVED (Jun 13, 2026)**
Joined PSMRI org via invite email. SSH key set up. Push successful.
Foundation PR #1 created.

---

### B8: Old Migration Plan in Docs Repo — RESOLVED

**Severity:** Low | **Discovered:** Day 3 (Jun 12) | **Status:** ✅ Resolved

**Issue:**
docs/migration-plan.md described the old two-phase upgrade approach
(Angular upgrade first, then ZardUI). This was incorrect after Jun 12
Teams confirmation of fresh repo approach.

**Resolution:**
Replaced with new plan written by Claude Code after full codebase +
ZardUI analysis. New plan reflects fresh repo, feature-by-feature
migration, ZardUI from day one.

---

## B9: shared/ui ZardUI components — RESOLVED (Jun 17, 2026)

**Status:** ✅ RESOLVED

**Resolution:** Added Common-UI as git submodule. src/app/shared/ui/ deleted. ZardUI now consumed via @common-ui/ui/ alias from Common-UI v2.

---

## B11: P0 Core Foundation Follow-ups (Jun 17, 2026)

**Severity:** Medium | **Discovered:** Jun 17, 2026 | **Status:** 🟡 Partially resolved

**Follow-ups:**
1. ✅ /login route — now registered in app.routes.ts (commit 3536d97)
2. ✅ Login calls AuthStore.setSession() — implemented correctly in login component
3. 🟡 5002 "logout-from-other-device" confirm path — still deferred, only hard logout implemented
4. 🟡 Wire real ZardUI dialog + spinner UI — window.* stubs still in place

**Next Steps:**
- Fix 3 when role-selection + session flows are tested end-to-end
- Fix 4 when ConfirmDialog is wired to real UI

---

## B12: UAT/DEV Base URLs — RESOLVED (Jun 15, 2026)

**Status:** ✅ RESOLVED

**Resolution:** Confirmed by Sneha Unki (Teams, Jun 15):
- UAT: https://uatamrit.piramalswasthya.org/
- DEV: https://amritwprdev.piramalswasthya.org/
Updated in environment.dev.ts. CTI (telephoneServer) URL and 104api path suffix still unconfirmed — needed before first deploy.

---

## B13: "Powered by: WIPRO" Footer Text (Jun 20, 2026)

**Severity:** Low | **Discovered:** Jun 20, 2026 | **Status:** 🟡 Waiting for Mithun confirmation

**Issue:**
Dashboard footer shows "Powered by: WIPRO" — carried over from legacy UAT app. Unsure if this is a contractual requirement or outdated.

**Next Steps:**
- Ask Mithun: keep, update, or remove?
- Do not change until confirmed.
---


## Open Questions

### Q1: ZardUI CLI + Version Management

**Status:** 🟡 Investigate Wk1

**Question:**
What's the current ZardUI CLI + recommended version for Angular 20? Is it stable? Any breaking changes?

**Next Steps:**
- Week 1 spike: check ZardUI docs + GitHub releases
- Pin version in package.json once confirmed
- Document in `research/zardui-component-coverage.md`

---

### Q2: Supervisor Report Tables + CSV Export — ZardUI Capable?

**Status:** 🟡 Investigate Wk1

**Question:**
Supervisor reports are data-heavy (paginated tables, filters, CSV export). Can ZardUI table + ngx-csv do this, or fallback needed?

**Next Steps:**
- Week 1 spike: test ZardUI table with CSV export library
- If yes: move ahead; if no: evaluate ng2-smart-table or CDK + custom wrapper
- Plan accordingly for Wk12 (report conversion)

---

### Q3: Material M3 Theming — Apply in Phase 1 or Defer?

**Status:** 🟡 Defer to Phase 2 (per Decision D2)

**Question:**
Should we upgrade Material to M3 in Phase 1 before Phase 2 replaces it with ZardUI? Or skip Material theming and go straight to ZardUI?

**Decision Made:**
Skip Material theming in Phase 1 (Decision D2); Phase 1 is mechanical Md*→Mat* only. Phase 2 does visual work.

**Rationale:**
- Material M3 + MDC refactor is expensive (all templates)
- Work is discarded when ZardUI replaces Material anyway
- Phase 1 + Phase 2 both touch every template if we do Material M3 in Phase 1

---

## Resolved / Non-Blockers

### ✅ N1: Angular 4 Feature Gaps (Decorators, Signals, etc.)

**Status:** Resolved (not applicable)

**Note:**
Angular 4 predates signals/standalone components. Phase 1 + 2 will introduce these naturally. No migration blocker.

---

### ✅ N2: Node Version — Can I Use Node 20?

**Status:** Resolved

**Answer:**
Yes. Angular 20 requires Node 18.17+; Node 22 is stable + recommended. Update to Node 22 in Wk1.

---

### ✅ N3: ZardUI Table Coverage — Resolved

**Status:** Resolved (Day 3 spike)

**Answer:**
ZardUI Table component exists and is installed. Sorting/filtering/pagination
confirmed via docs. Plan is to build one reusable DataTable wrapper component
(Week 2) used across all supervisor reports and outbound worklist screens.

---

### ✅ N4: ZardUI + Angular 20 Compatibility — Resolved

**Status:** Resolved (Day 3 setup)

**Answer:**
ZardUI fully supports Angular 20. `npx zard-cli init` ran successfully.
Build passes. ng serve works. One watch item: @ng-icons/core@33 declares
peer of Angular ≥21 — resolves via --legacy-peer-deps, no build errors.

## Workarounds & Notes

**Loader Spinner Ref-counting Bug:**
Current LoaderService shows spinner on first request, hides on first response — doesn't count concurrent requests. Workaround: add a ref-counter in LoaderService during Wk2 rewrite. Prevents spinner flickering during sequential requests.

**SessionStorage Encryption (Empty Key):**
sessionStorageService.encKey = '' locally → encryption is trivial. Workaround: it still works; session keys are persisted. Don't "fix" during Phase 1; keep as-is to avoid auth regressions.

**Hardcoded localhost URLs:**
Many services use hardcoded URLs for local dev. Workaround for now: add entries to `environment.local.ts` and use ConfigService. Final fix: consolidate all hardcodes to ConfigService (Wks 3–5).

---

## Template

Use this for new blockers:

```
### B[N]: [Title]

**Severity:** [Critical / High / Medium / Low] | **Discovered:** [Date] | **Status:** [🚧 Active / 🟡 Ticket / 🟢 Mitigated / ✅ Resolved]

**Issue:**
[What is it?]

**Current Understanding:**
[What do we know?]

**Next Steps:**
1. [Action]
2. [Action]

**Mitigation:**
[How do we handle this?]
```
