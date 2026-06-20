# Week 2 — Jun 17–23, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
P0 complete. Full dashboard built with UAT parity across all 4 roles. PR #1 + PR #2 merged. PR #3 open for Mithun's review.

---

## Completed ✅

**Day 8 (Jun 17):**
- [x] ConfigService, AuthStore, SessionStorageService, SessionService
- [x] HTTP interceptors: auth, error, loader (functional HttpInterceptorFn)
- [x] Subagent review — caught 4 issues, fixed all
- [x] Login component with password encryption (PBKDF2-SHA512 + AES)
- [x] Routing skeleton (/login, /role-selection, /dashboard, /reset-password)
- [x] DataTable wrapper (sort, filter, pagination, CSV export)
- [x] ConfirmDialog service wrapper (Observable API)
- [x] AMRIT blue/white theme applied (#0277bd)
- [x] Common-UI added as git submodule (@common-ui/ui/ alias)
- [x] src/app/shared/ui/ removed — ZardUI from Common-UI submodule
- [x] Login UI polished — Piramal Swasthya branding, eye icon, input icons, forgot password link
- [x] PR #1 merged ✅

**Day 9 (Jun 18):**
- [x] RoleSelectionComponent (ports ServiceRoleSelectionComponent from old app)
- [x] Feature code mapping (Registration→RO, Health_Advice→HAO, Counselling→CO etc.)
- [x] AuthGuard (CanActivateFn) protecting /role-selection and /dashboard
- [x] Dashboard placeholder
- [x] PR #2 merged ✅

**Day 10 (Jun 19):**
- [x] Missed Friday sync (health)
- [x] Reviewed Atomic Design book (Mithun's recommendation)

**Day 11 (Jun 20):**
- [x] Deep UAT comparison: all 4 roles via Playwright audit
- [x] Full dashboard: header, sidebar, agent ID, campaign toggle (radios), call statistics, alerts, reports, activity, rating, footer
- [x] i18n layer: en/hi/as, 24-language selector, signal-based, sessionStorage-persisted
- [x] Feedback page after logout (/feedback?sl=104)
- [x] Layout fixes: absolute sidebar, full-width header/footer, centered login card
- [x] Icon tooltips, red logout icon, dynamic copyright year
- [x] Fixed all CodeRabbit findings (supervisor toggle, LOGIN_ROUTE, null agentId)
- [x] PR #3 opened ✅ https://github.com/PSMRI/Helpline104-UI-NEXT/pull/3

---

## In Progress / Blocked 🚧
- [ ] PR #3 waiting for Mithun review
- [ ] 5002 "logged-in-elsewhere" real ZardUI dialog (B11 item 3) — deferred to auth-flows PR
- [ ] window.confirm stubs → real ZardUI dialog (B11 item 4) — deferred
- [ ] "Powered by: WIPRO" footer — waiting for Mithun confirmation on whether to keep
- [ ] Feedback API POST not wired — no 104 feedback endpoint confirmed yet

---

## Learnings & Gotchas 💡
- Always check UAT before building any screen — saved hours of guessing
- UAT language selector has 24 languages but only 3 actually work (en/hi/as)
- Sidebar must be position:absolute to avoid pushing content
- HAO logs in as "RO Dashboard" — privilege remap in legacy app, replicated as-is
- CodeRabbit rate limit hits fast on big PRs — batch fixes before pushing
- Subagent review caught real issues every single time — never skip it
- WIPRO footer text is from original 2018 build — may be contractual, needs Mithun confirmation
- Feedback link + logout both navigate to /feedback?sl=104 — same destination

---

## Risks ⚠️
- PR #3 is a big PR (2725 lines) — Mithun may have feedback
- 118 components, mid-August deadline — need to ship fast after PR #3 merges
- 5002 path still stubbed — needs to be wired before auth flows are complete

---

## Metrics
- PRs merged this week: PR #1, PR #2
- PRs open: PR #3 (dashboard)
- New files in PR #3: 27 files, +2725 lines
- Dashboard components built: 14
- i18n keys: 64 × 3 locales (en/hi/as)
- Roles covered: HAO/RO, MO, CO, Supervisor

---

## Next Week's Plan
- Mithun reviews + merges PR #3
- Auth flows: set-password, set-security-questions, reset-password
- Wire 5002 real ZardUI dialog
- Branch: feat/auth-flows