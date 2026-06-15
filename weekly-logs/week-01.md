# Week 1 — Jun 10–16, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
Codebase analysis complete. Angular 20 scaffold created. ZardUI installed
and configured. Migration plan written. Team confirmed fresh repo approach.
Foundation PR #1 open — LGTM from Mithun ✅

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

**Day 4 (Jun 13):**
- [x] Cloned all 6 AMRIT repos into AMRIT folder
- [x] Claude Code deep analysis across all repos
- [x] Key finding: Common-UI is NgModule/Material bound, cannot be directly consumed by Angular 20
- [x] Rewrote migration plan to v2 addressing all 5 mentor feedback points
- [x] Set up SSH key, resolved GitHub 403 write access issue
- [x] Created and pushed foundation PR #1 to PSMRI/Helpline104-UI-NEXT
- [x] Sent Teams update to Dr. Mithun + Sneha with PR and plan links

**Day 5 (Jun 14):**
- [x] Read Teams conversation — Gopi handling Common-UI Angular 20 upgrade
- [x] Updated migration plan to v3 — Common-UI strategy updated
- [x] Prepared for Monday meeting

**Day 6 (Jun 15):**
- [x] Re-aligned scaffold to app-modules/ convention (studied from MMU-UI)
- [x] Removed core/tokens/ — config now in environments/
- [x] Scaffolded environment files (local/dev/test/prod/ci) matching AMRIT pattern
- [x] Updated DEV/UAT URLs with confirmed values from Mithun
- [x] Ran first subagent review on PR #1 — caught + fixed real issues
- [x] Fixed all CodeRabbit comments
- [x] Added GPL-3.0 license headers to all authored files
- [x] Fixed "Integrated Technology" → "Integrated Technologies" typo
- [x] Coordinated with Gopi on Common-UI v2 — will consume shared/ui after his migration
- [x] PR #1 — LGTM from Mithun ✅

---

## In Progress / Blocked 🚧
- [ ] shared/ui → Common-UI v2 (waiting for Gopi's standalone + ZardUI migration — B9)
- [ ] prod/test hostnames still placeholders (waiting backend confirmation)

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
- AMRIT GPL-3.0 header typo — "Integrated Technologies" not "Technology"
- environment.ts is gitignored in AMRIT convention — devs cp from environment.local.ts
- Subagents: figure out by doing, not planning — first real review caught real issues

---

## Risks Identified ⚠️
- @ng-icons version skew (watch item, not blocking)
- Scope: 118 components in 13 weeks — P0+P1+partial P2 is realistic commitment
- shared/ui move to Common-UI depends on Gopi's timeline

---

## Metrics
- Old codebase: 118 components, 60 services, 9 roles
- ZardUI components installed: 8
- New scaffold files committed: 93
- MCPs configured: 3 (Context7, Playwright, Chrome built-in)
- PR #1 commits: 8

---

## Next Week's Plan
- Wait for Gopi's standalone migration PR on Common-UI v2
- Start Week 2: core HTTP interceptors, AuthStore, reusable DataTable
- Confirm prod/test hostnames with backend team