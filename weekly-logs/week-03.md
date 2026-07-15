# Week 3 — Jun 24–30, 2026

**Status:** ✅ COMPLETE

---

## Summary
Auth flows shipped (PR #4 merged) and the inbound-call track opened: beneficiary
registration with full legacy field parity, the CTI-driven Innerpage call shell
(PR #5 merged), and the HAO workspace with a CDK stepper replacing the legacy
carousel. PR #3 (dashboard) merged at the start of the week.

---

## Completed ✅

**Day 15 (Jun 24):**
- [x] PR #3 merged ✅ — full dashboard with all Mithun + Sneha feedback addressed
- [x] Created feat/auth-flows branch
- [x] Full UAT audit of auth flows via Playwright (docs/AUTH_FLOWS_AUDIT.md)
- [x] Key finding: UAT hardened against username enumeration (neutral 5002) — old app swallows it
- [x] Started building auth flow components

**Day 16 (Jun 25):**
- [x] Built all 3 auth flow components (reset-password, set-password, set-security-questions)
- [x] UAT comparison audit; button alignment fix; show/hide answer toggle
- [x] Sneha confirmed first-login flow semantics (admin-set default password)

**Day 18 (Jun 26):**
- No coding work. Rest/planning day.

**Day 19 (Jun 27):**
- [x] Committed auth flows; addressed CodeRabbit review (envelope errors, stale store, transactionId, whitespace validators)
- [x] transactionId cleared on Q&A edits — prevents stale save replay
- [x] PR #4 merged ✅

**Day 20 (Jun 28):**
- [x] Beneficiary registration component (BeneficiaryService, models, /innerpage/registration)
- [x] CTI-driven Innerpage shell — CallStore, guard, route
- [x] callId/beneficiaryId reset on new call; timer aria-label i18n
- [x] PR #5 merged ✅

**Day 21 (Jun 29):**
- [x] Rebuilt beneficiary registration with full legacy field parity

**Day 22 (Jun 30):**
- [x] HAO workspace — CDK stepper (replaces legacy carousel), case sheet, service delivery, closure
- [x] Second CodeRabbit round: CLI guard, emergency reset, DOB parsing, cascade race, readData typing
- [x] spouseName mapping + DOB timezone offset fixes

---

## Resolved Blockers ✅
- PR #3 Sneha UI feedback — all addressed, merged Jun 24
- Auth flows CodeRabbit findings — all fixed before PR #4 merge

## Still Open 🚧
- First-login E2E verification — still waiting on a fresh "New"-status test account from Mithun
- 5002 "logged-in-elsewhere" real dialog — landed with auth flows; UAT-verified later

---

## Learnings & Gotchas 💡
- Always audit live UAT before building — username-enumeration hardening (neutral 5002) was invisible in the old code
- Stale transactionId can replay an old save — clear on every Q&A edit
- Call-scoped state must reset on each new call, or IDs leak across calls
- DOB has two separate traps: calendar parsing and timezone offset
- Legacy jQuery carousel maps cleanly to a CDK stepper — no jQuery needed

---

## Risks Identified ⚠️
- CTI features can't be E2E-verified without softphone/UAT CTI access
- HAO API contracts drifted from the legacy models — every workspace needs live-UAT contract checks

---

## Metrics
- PRs merged: #3 (dashboard), #4 (auth flows), #5 (inbound call)
- New feature areas: auth flows (3 components), beneficiary registration, call shell, HAO workspace

---

## Week 4 Goals
- Merge beneficiary registration + HAO workspace PRs
- SNOMED CT search + CDSS for the case sheet
- Remaining role workspaces + case-sheet extras + screenings
- SIO service tabs

*Backfilled on Jul 16, 2026 from daily logs and the Helpline104-UI-NEXT commit history.*
