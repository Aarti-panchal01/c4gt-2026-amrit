# Week 1 — Jun 10–16, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
Deep codebase analysis complete. Angular 20 + ZardUI confirmed as targets. 
13-week migration plan drafted. 104 vs 1097 comparison done. 
Team alignment meeting Jun 15 before Phase 1 sprint begins.

---

## Completed ✅

**Day 1 (Jun 10):**
- [x] Set up Claude Code with Piramal org account
- [x] Configured status line (session/weekly/left/reset timers via node, no jq)
- [x] Created AGENTS.md with project conventions
- [x] Full codebase analysis: auth flow, call flow, 67 services, 
      HTTP interceptor pattern, migration blockers
- [x] Built 13-week migration plan (Phase 1: Angular 20, Phase 2: ZardUI)
- [x] Set up documentation repo structure (this repo)

**Day 2 (Jun 11):**
- [x] Read all 5 Martin Fowler AI articles (Dr. Mithun assignment)
- [x] Cloned + explored Helpline1097-UI
- [x] Full 104 vs 1097 comparison: shared code, differences, Common-UI tiers
- [x] Saved all architecture analysis to docs/architecture/
- [x] Confirmed Angular 4 (not 16) for 104; Angular 20 as target
- [x] Created docs/claude-context.md (gitignored Claude Code session starter)

---

## In Progress / Blocked 🚧
- [ ] Monorepo decision (PENDING — Jun 15 meeting with Dr. Mithun)
- [ ] Angular 20 workspace scaffold (starts after Jun 15 confirmation)
- [ ] ZardUI component coverage spike (starts Week 1 after Jun 15)

---

## Learnings & Gotchas 💡
- Fresh re-platform, not ng update — too many blockers at intermediate versions
- HTTP interceptor rewrite (Wk2) is highest risk — 5002/force-logout must 
  preserve exactly; concurrent-login flow depends on it
- 1097's token.interceptor.ts is closer to Angular 20 target than 104's 
  Http-subclass — use it as starting point if monorepo confirmed
- ~440 jQuery calls deferred to Phase 2 — touching them in Phase 1 
  adds risk with no benefit
- CUSTOM_ELEMENTS_SCHEMA is masking unknown-element errors — removal 
  in Wk9 will surface latent bugs

---

## Risks Identified ⚠️
- Interceptor 5002/force-logout semantics (Critical — Wk2)
- jQuery × ZardUI DOM collisions (High — Wk11, deferred by design)
- ZardUI table component coverage unclear (High — spike needed Wk1)
- Near-zero test safety net during migration (High — protect 20 core specs)

---

## Metrics
- Services cataloged: 67
- Components counted: 118
- Templates counted: 138
- Hardcoded URLs found: 7 services
- jQuery calls found: 440+
- Weeks planned: 13

---

## Next Week's Plan (after Jun 15 confirmation)
- Angular 20 workspace scaffold
- ZardUI component coverage spike
- Dual-build CI setup
- Begin HTTP layer planning

