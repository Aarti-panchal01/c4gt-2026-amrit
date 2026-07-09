# Week 5 — Jul 7–11, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
Full end-to-end UAT verification of everything built so far, then the P2/P3 build-out: agent-facing extras (polling, consent, reports, notifications) and the whole Supervisor area (workspace shell, reports hub, configuration screens), plus the CTI login handshake and case-sheet wiring. PRs #15–#23 opened (on top of #14), all CodeRabbit Major findings fixed, every PR endpoint verified against live UAT, and 5 demo recordings captured against UAT. Final pre-submission review written.

---

## Completed ✅

**Jul 7–8:**
- [x] Full E2E UAT audit of all built screens — 29 screenshots + `docs/e2e-report.md`
- [x] Migration plan refreshed to **v4** (repo audit + UAT E2E): 89 migratable components, ~63% merged
- [x] PR #14 opened — HAO service-tab real-component wiring, SNOMED/CDSS in case sheet, role-dispatcher auto-routing, beneficiary-search timeout state
- [x] Confirmed the CTI login handshake / call lifecycle were NOT yet in the codebase (grep-verified) → planned

**Jul 9 (D17):**
- [x] Built + opened **PRs #15–#23**:
  - #15 dashboard agent-status polling + status chip
  - #16 beneficiary-consent modal (CO / Counsellor)
  - #17 agent call-type (CDI) report suite (+ Surveyor)
  - #18 dashboard alerts & notifications dialog
  - #19 supervisor workspace shell → #20 reports hub (8 reports, xlsx) → #21 configuration screens (stacked)
  - #22 CTI login handshake (getLoginKey/getAgentIPAddress/doAgentLogin) wired into login/logout
  - #23 casesheet history + disease summary + schedule appointment wired in; `districtID` via CallStore
- [x] Fixed **all CodeRabbit Major findings** (KM creds → ConfigService/env, alternate-email `canSend()`, upload-schemes extension + FileReader race, i18n typos, silent grievance-email failure, supervisor role-guard)
- [x] **Live UAT API verification** — 35/41 endpoints verified (rest were harness artifacts except MMU/TM 403)
- [x] **5 screen recordings** vs live UAT with network-proof logs
- [x] Final review report → `docs/final-review-2026-07-09.md`

---

## In Progress / Blocked 🚧
- [ ] PRs #14–#23 awaiting Mithun review (supervisor stack merges in order #19 → #20 → #21)
- [ ] **CTI call lifecycle** (startCall/closeCall/beneficiaryByCallID) — blocked on UAT CTI access (Kundan Kumar)
- [ ] **SIO tabs** (#13) E2E — blocked on an SIO test credential
- [ ] MMU/TM case-sheet history — 403 for 104 users (cross-program access)
- [ ] Common-UI #78 (undefined-zContent crash fix) — awaiting maintainer merge, then bump submodule

---

## Learnings & Gotchas 💡
- UAT auth is carried by the **`Jwttoken` cookie** (`withCredentials`) *plus* the Authorization header — a header-only request 401s.
- UAT enforces **one active session per user**; re-login triggers the 5002 "already logged in" dialog which must be accepted to kick the prior session.
- The **backend does not gate supervisor endpoints by role** — client guards are defence-in-depth only; server-side authz is the real fix.
- `104hao` on UAT has `agentID = null` → CTI/agent-status features correctly no-op for it.
- On a stacked set of PRs, prefer **follow-on commits** over amending the base — amending the base forces a full re-rebase of every branch above it.
- Playwright `recordVideo` + response logging gives honest demo proof (real UAT, no mocking).

---

## Risks ⚠️
- **Server-side authorization gap** on supervisor endpoints (incl. destructive force-logout) — needs backend fix before GA.
- CTI features can't be fully E2E-verified without softphone/CTI access — a known dependency on Kundan Kumar.
- Large, stacked supervisor PRs — merge order matters; the reports placeholder must be dropped when #20/#21 land.
- Mid-August deadline: P3 supervisor area is the last big chunk — mostly built now, pending review/merge.

---

## Metrics
- PRs opened this week: #14 (Jul 8), #15–#23 (Jul 9) — 10 PRs
- Components moved to In Review this sprint: ~30 (agent extras + full supervisor area + CTI handshake + casesheet wiring)
- CodeRabbit Major findings fixed: all reported (KM creds, canSend, upload-schemes×2, i18n, email failure, role-guard)
- UAT endpoints verified: 35/41
- Demo recordings: 5 (.webm vs live UAT)
- Migration plan (v4): 89 migratable · ~63% merged before this sprint's In-Review batch

---

## Next Week's Plan
- Land Mithun's review feedback on #14–#23; merge the supervisor stack in order.
- Push for the server-side supervisor authorization fix.
- Unblock + verify CTI call lifecycle and SIO tabs once access/credentials arrive.
- Begin September-track cleanup/QA prep.
