# Decisions — C4GT 2026 AMRIT Helpline104-UI

**Status:** Decisions D1–D2 confirmed Day 1; D3–D9 are design drafts pending team alignment meeting Jun 15.

This document records architectural and technical decisions made during the migration. Each entry includes the rationale, alternatives considered, and outcome.

---

## D1: Fresh Workspace Re-platform (CONFIRMED)

**Date:** Jun 9, 2026 (pre-internship)
**Context:** Angular 4.1.3 → 20 migration path
**Decision:** Build a fresh Angular 20 workspace; port code into it; run old + new in parallel, cut over per feature area.

**Rationale:**
- `ng update 4→5→…→20` is not viable: multiple hard blockers hit early (RxJS break, Material removal, @angular/http death, third-party lib abandonment).
- Fresh workspace allows: clean esbuild setup, parallel old+new validation, per-feature cutover.

**Outcome:** ✅ **CONFIRMED.** Scaffolding begins Week 1.

---

## D2: ZardUI as Phase 2 UI Library (CONFIRMED)

**Date:** Jun 9, 2026 (pre-internship, from meeting)
**Context:** UI modernization path
**Decision:** Post-Angular 20 migration, integrate **ZardUI** (Tailwind + Angular CDK) as replacement for Material/md2/jQuery.

**Rationale:**
- ZardUI chosen by team as modern, unstyled-by-default component approach.
- Tailwind fits modern design-token-first systems.

**Outcome:** ✅ **CONFIRMED.** Will spike ZardUI component coverage in Wk1; full integration planned post-Angular migration.

---

## ⏳ Design Drafts (Pending Jun 15 Planning Meeting)

The following decisions (D3–D10) are design drafts based on initial assumptions. **They are pending confirmation/refinement in the team planning meeting on Monday, Jun 15, 2026.**

Once confirmed, they will move to the **CONFIRMED** section. Until then, treat as proposals.

### D3: HttpContext Tokens for Auth/Silent Logic

**Date:** Jun 10, 2026 (Day 1)
**Context:** 13-week schedule with Phase 1 (Angular 4→20) + Phase 2 (ZardUI integration)
**Decision:** In Phase 1, replace Material/`md2`/jQuery UI with only a mechanical Md*→Mat* rename. Skip Material M3 theming + MDC visual-QA until Phase 2 when ZardUI replaces it wholesale.

**Rationale:**
- 138 templates × 2 (Material refactor + ZardUI refactor) = 276 template edits without the shortcut.
- Material M3 + MDC DOM changes are expensive (layout, spacing, component internals all change).
- ZardUI will replace Material + `md2` + jQuery anyway, rendering Phase 1 Material work throw-away.
- **By deferring UI:** materialize Phase 1 as "app compiles on Angular 20 + HttpClient + RxJS 7 + lazy routes" (no visual QA yet).
- Phase 2 does the visual work once, against ZardUI.

**Alternatives Considered:**
1. Full Material M3 upgrade in Phase 1 — adds 2–3 weeks; work is discarded when ZardUI replaces it.
2. Skip Material entirely in Phase 1, jump to ZardUI — violates milestone clarity; Phase 1 deliverable becomes "works but looks broken."

**Outcome:**
- Accepted. Allows 13-week timeline.
- Material beta → Mat* rename only (Wk 6, mechanical find-replace).
- Visual/MDC/Tailwind validation deferred to Phase 2 (Wks 10–13).
- Will track any layout regressions and defer fixing to Phase 2.

---

## D3: HttpContext Tokens for Auth/Silent Logic (vs. Two-Class Distinction)

**Date:** Jun 10, 2026 (design phase)
**Context:** Replace custom `Http` subclasses with functional `HttpInterceptor`s
**Decision:** Use `HttpContext` tokens to distinguish between standard (loader + full error handling) and silent (no loader, lenient error) request modes. Single interceptor, branching on token value.

**Rationale:**
- Current app has two `Http` subclasses: `InterceptedHttp` (public-facing, loader, aggressive error UX) and `SecurityInterceptedHttp` (silent, lenient).
- Services inject both explicitly — interception is opt-in per service.
- In functional `HttpInterceptor`s, we can use `req.context.get(IS_SILENT_REQUEST)` to branch instead of duplicating the interceptor.
- HttpContext is the modern Angular pattern (HttpClient 4.3+).

**Alternatives Considered:**
1. Two separate functional interceptors — adds complexity, shares no logic.
2. URL pattern matching (e.g., `/silent/*`) — fragile; not semantically clear.
3. Request header (e.g., `X-Silent: true`) — works but HttpContext is more idiomatic.

**Outcome:**
- Decision confirmed. Will implement in Week 2 as part of HTTP layer refactor.
- Reference implementation to live in `research/interceptor-reference.md` after Wk2 completion.

---

## D4: Standalone Components + Lazy `loadComponent` Routes

**Date:** Jun 10, 2026 (design phase)
**Context:** Replace single ~530-line `app.module.ts` with modern standalone + lazy-loading
**Decision:** Convert all 118 components to `standalone: true`, bootstrap via `bootstrapApplication`, define routes using lazy `loadComponent()` (not lazy modules).

**Rationale:**
- Modern Angular best practice; reduces complexity.
- Single `app.module.ts` is a maintainability smell: all components declared + all routes defined in one file.
- `standalone: true` + lazy `loadComponent` is the pattern for Angular 15+; Phase 1 target is Angular 20.
- Shrinks initial bundle (fewer imports upfront).

**Alternatives Considered:**
1. Keep NgModule + lazy-loaded feature modules — still works in Angular 20, but not modern; higher boilerplate.
2. Mixed approach (some standalone, some module) — adds consistency confusion.

**Outcome:**
- Accepted. Phase 1 scaffold (Wk1) sets up `bootstrapApplication` in main.ts.
- Component-by-component standalone conversion during Phase 1 (Wks 6–9).

---

## D5: ZardUI vs. Angular Material + CDK (Post-Phase 1 UI)

**Date:** Jun 10, 2026 (stakeholder decision, pre-internship)
**Context:** Phase 2 component library choice
**Decision:** Integrate **ZardUI** (Tailwind + Angular CDK component kit) instead of staying on Angular Material v19/20.

**Rationale:**
- ZardUI is a modern shadcn-style, unstyled-by-default, Tailwind-driven approach.
- Fits modern design systems (design tokens, utility-first, M3 out of box).
- Not locked into Material's MDC/theming overhead.
- Younger project, so spike required (Wk1) to validate component coverage vs. our needs.

**Alternatives Considered:**
1. Upgrade to latest Material 20 — leaves Material complexity; not modern shadcn-style.
2. Other libraries (Headless UI, Chakra, etc.) — less integrated with Angular CDK; ZardUI chosen by team.

**Outcome:**
- Accepted as Phase 2 target.
- Spike in Week 1 to validate table, datepicker, dialog, tabs, pagination coverage.
- Fallback strategy: CDK primitives + custom styling if ZardUI gap found.

---

## D6: Keep Bootstrap 3 + jQuery Until Phase 2 (No Early Rip-out)

**Date:** Jun 10, 2026 (design phase)
**Context:** Cleanup strategy for legacy CSS/JS
**Decision:** Leave Bootstrap 3 and jQuery dependencies untouched until Phase 2. Do not refactor jQuery in Phase 1.

**Rationale:**
- ~440 jQuery calls across 20+ components, heavily clustered in 104-sio/hao/co.
- Refactoring jQuery + migrating to RxJS 7 in same week = high churn + risk.
- Better to stabilize code on Angular 20 + HttpClient + RxJS 7 first, then rip out jQuery when ZardUI replaces DOM.
- Bootstrap 3 CSS coexists with Tailwind (preflight tuned) — no early-phase conflict.

**Alternatives Considered:**
1. Refactor jQuery early (Phase 1) — adds 1–2 weeks; blocks other progress.
2. Drop Bootstrap 3 + jQuery immediately — layout regressions in Phase 1; hard to debug while also migrating frameworks.

**Outcome:**
- Accepted. jQuery/Bootstrap left as-is in Phase 1 (Wks 1–9).
- jQuery removal + ZardUI tabs/stepper conversion bundled in Wk11 (Phase 2).
- Will track any layout bugs in logs and defer fixing to Phase 2.

---

## D7: Test Strategy: Rewrite Core Subset, Ticket the Rest

**Date:** Jun 10, 2026 (design phase)
**Context:** 112 existing specs (Karma/Jasmine) + Protractor e2e
**Decision:** Rewrite a critical subset of specs (auth, HTTP layer, core services) against HttpClientTestingModule. Ticket remaining specs for later. Replace Protractor with Playwright smoke tests (login → call → closure).

**Rationale:**
- All 112 specs depend on Angular 4's TestBed, HttpModule, etc. Bulk rewrite is expensive.
- Core 15–20 specs (auth flow, interceptor correctness, session/guard logic) are high-value + must-have for confidence.
- Remaining 90+ specs are important but lower-priority; ticket them as "test-rewrite: <component>" for follow-up.
- Protractor is dead (Selenium-based, no active maintenance); Playwright is modern + more reliable.

**Alternatives Considered:**
1. Rewrite all 112 specs in Phase 1 — adds 2–3 weeks; schedule blows up.
2. Skip testing in Phase 1, do it all in Phase 2 — risky; no safety net during core refactor.
3. Run old specs on Angular 20 (no rewrite) — likely won't work; old test patterns incompatible.

**Outcome:**
- Accepted. Core specs (~20) rewritten in Wk5–9 as services land.
- Remaining 90+ specs created as tickets; lower priority.
- Playwright smoke suite (3–5 key workflows) added in Wk13.

---

## D8: Functional Route Guards (CanActivateFn, CanDeactivateFn)

**Date:** Jun 10, 2026 (design phase)
**Context:** Replace class-based guards with modern functional pattern
**Decision:** Convert AuthGuard, AuthGuard2, SaveFormsGuard to functional `CanActivateFn`, `CanDeactivateFn` predicates.

**Rationale:**
- Functional guards are the modern Angular pattern (15+).
- Less boilerplate; easier to test (pure functions).
- Integrate cleanly with `provideRouter(routes)` in standalone bootstrap.

**Alternatives Considered:**
1. Keep class-based guards — works in Angular 20, but not modern; requires DI + class setup.

**Outcome:**
- Accepted. Will be implemented during Wk6 (routing refactor).

---

## D9: Consolidate Hardcoded URLs to ConfigService

**Date:** Jun 10, 2026 (codebase analysis)
**Context:** 6 services hardcode base URLs (localhost, 10.152.3.152, 10.208.122.38) instead of using ConfigService
**Decision:** Fix all hardcoded URLs to reference ConfigService (already supports environment-driven config via src/environments/). Blocklist: AdminLanguageService, AdminRoleService, AdminScreenService, AdminServicemasterService, AdminUserService, SocketService, RegisterService.

**Rationale:**
- Hardcoded URLs defeat environment config (localhost for dev, IP for prod, etc.).
- Makes local development fragile; non-deterministic based on dev machine.
- Easy fix: add entries to ConfigService if needed, use getApiBaseUrl() everywhere.

**Alternatives Considered:**
1. Leave hardcodes as-is — bad practice; enables future deployment bugs.

**Outcome:**
- Added to blockers list; fix in Phase 1 (Wks 3–5 during services migration).

---

## D10: Tailwind Preflight Tuning for Bootstrap 3 Coexistence

**Date:** Jun 10, 2026 (design phase)
**Context:** Phase 2 will add Tailwind CSS alongside Bootstrap 3 (until Bootstrap is fully replaced)
**Decision:** Configure Tailwind's preflight to not break Bootstrap 3 (e.g., keep button reset rules minimal, preserve box-sizing model).

**Rationale:**
- Bootstrap 3 CSS will be present in Phase 1 + early Phase 2.
- Tailwind's preflight resets styles globally; without tuning, it conflicts with Bootstrap.
- Light preflight config allows both to coexist until Phase 2 migration is complete.

**Alternatives Considered:**
1. Disable Tailwind preflight entirely — adds CSS resets we'd have to do manually elsewhere.
2. Remove Bootstrap 3 upfront — violates "do UI once" decision.

**Outcome:**
- Accepted. Tailwind setup in Wk10 will include tuned preflight config.

---

## Next Entry

Add entries here as you make decisions during the internship. Format:

```
## D[N]: [Title]

**Date:** [Date]
**Context:** [What was the question/problem?]
**Decision:** [What did we decide?]

**Rationale:** [Why this choice?]

**Alternatives Considered:**
1. [Alt 1 + why ruled out]
2. [Alt 2 + why ruled out]

**Outcome:** [What happens next / when it's implemented?]
```
