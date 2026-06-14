# Week 1 — Jun 10–16, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
Codebase analysis complete. Angular 20 scaffold created. ZardUI installed
and configured. Migration plan written. Team confirmed fresh repo approach.
Foundation committed locally, pending write access to push.

---

## Completed ✅

**Day 1 (Jun 10):**
- [x] Set up Claude Code with Piramal org account
- [x] Configured status line (session/weekly/left/reset timers)
- [x] Created AGENTS.md with project conventions
- [x] Full codebase analysis: auth flow, call flow, 67 services,
      HTTP interceptor pattern, migration blockers
- [x] Built initial migration plan
- [x] Set up documentation repo structure

**Day 2 (Jun 11):**
- [x] Read all 5 Martin Fowler AI articles (Dr. Mithun assignment)
- [x] Cloned + explored Helpline1097-UI
- [x] Full 104 vs 1097 comparison: shared code, differences, Common-UI tiers
- [x] Saved all architecture analysis to docs/architecture/
- [x] Confirmed Angular 4 (not 16) for 104; Angular 20 as target
- [x] Created docs/claude-context.md (gitignored Claude Code session starter)

**Day 3 (Jun 12):**
- [x] Received PSMRI org access + cloned Helpline104-UI-NEXT
- [x] Confirmed approach with Dr. Mithun: fresh repo + ZardUI from day one
- [x] Installed Node 24 + Angular CLI 20
- [x] Set up MCPs: Context7 ✅, Playwright ✅, claude-in-chrome ✅ (built-in)
- [x] Updated Claude Code to v2.1.175
- [x] Explored UAT app (all 4 roles: HAO, CO, MO, Supervisor)
- [x] Read merge vs rebase
- [x] Claude Code studied old codebase + ZardUI docs + wrote new migration plan
- [x] Angular 20 workspace scaffolded (zoneless, standalone, signals)
- [x] ZardUI + Tailwind v4 installed and configured
- [x] 8 starter components added (Button, Input, Form, Dialog, Table,
      Pagination, Toast, Loader)
- [x] Full folder structure created (core/, shared/, layout/, features/)
- [x] Build passes ✅, ng serve works ✅
- [x] Foundation committed locally (commit 4875251, branch angular-zard-migration)

## Day 4 — June 13, 2026

**Focus:** AMRIT platform setup, deep codebase analysis, foundation PR

- Cloned all 6 AMRIT repos into AMRIT folder (Helpline104-UI, 1097-UI, Common-UI, Common-API, MMU-UI, Helpline104-API)
- Claude Code ran deep analysis across all repos — key finding: Common-UI is NgModule/Material bound, cannot be directly consumed by Angular 20
- Rewrote migration plan to v2 addressing all 5 mentor feedback points
- Set up SSH key, resolved GitHub 403 write access issue
- Created and pushed foundation PR #1 to PSMRI/Helpline104-UI-NEXT
- Sent Teams update to Dr. Mithun + Sneha with PR and plan links

## Day 5 — June 14, 2026

**Focus:** Common-UI strategy update, Monday meeting prep

- Read Teams conversation — Gopi handling Common-UI Angular 20 upgrade (v2 folder approach confirmed by Mithun)
- Updated migration plan to v3 — Common-UI strategy now: consume Gopi's v2 branch when ready
- Prepared for Monday meeting with Dr. Mithun James

---

## In Progress / Blocked 🚧
- [ ] Write access to PSMRI/Helpline104-UI-NEXT (pending — B7)
- [ ] Push foundation PR (blocked on write access)
- [ ] Plan review + approval from Dr. Mithun + Sneha Unki

---

## Learnings & Gotchas 💡
- Fresh repo approach confirmed — NOT ng update
- ZardUI hard-requires CSS not SCSS — switched global stylesheet (D13)
- @ng-icons/core@33 has Angular ≥21 peer dep — resolves via legacy-peer-deps,
  not breaking, but watch it
- ZardUI CLI copies component source into repo (shadcn-style) — you own the code
- merge vs rebase: use rebase to sync feature branch with main updates,
  merge (via PR) to land work into angular-zard-migration
- Zoneless Angular 20 — no zone.js, better performance
- UAT app requires live CTI call to access case-sheet/closure screens

---

## Risks Identified ⚠️
- Write access delay could push Week 2 start (B7)
- @ng-icons version skew (watch item, not blocking)
- Scope: 118 components in 13 weeks — P0+P1+partial P2 is realistic commitment

---

## Metrics
- Old codebase: 118 components, 60 services, 9 roles
- ZardUI components installed: 8
- New scaffold files committed: 91
- MCPs configured: 3 (Context7, Playwright, Chrome built-in)

---

## Next Week's Plan
- Get write access + push foundation PR
- Get plan approval from Mithun + Sneha
- Start Week 2: core HTTP interceptors, AuthStore, reusable DataTable