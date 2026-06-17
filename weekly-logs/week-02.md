# Week 2 — Jun 16–22, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
P0 complete. Login component built and polished. Common-UI wired as submodule.
AMRIT theme applied. PR #1 has 23 commits — waiting for Mithun's review + merge.
Friday sync scheduled with team.

---

## Completed ✅

**Day 7 (Jun 16):**
- [x] Added ZardUI components to Common-UI v2/ui/
- [x] Fixed all import paths + GPL-3.0 headers
- [x] PR #49 opened + approved by Mithun + merged ✅
- [x] Coordinated with Gopi on shared AMRIT theme

**Day 8 (Jun 17):**
- [x] ConfigService, AuthStore, SessionStorageService, SessionService
- [x] HTTP interceptors: auth, error, loader (functional HttpInterceptorFn)
- [x] Subagent review — caught 4 issues, fixed 2 immediately
- [x] CodeRabbit fixes: APIMAN key bleed, timer init, storage guards
- [x] Login component with password encryption (PBKDF2-SHA512 + AES)
- [x] Routing skeleton (/login, /role-selection placeholder)
- [x] DataTable wrapper (sort, filter, pagination, CSV export)
- [x] ConfirmDialog service wrapper (Observable API)
- [x] AMRIT blue/white theme applied from Gopi (#0277bd)
- [x] Common-UI added as git submodule (@common-ui/ui/ alias)
- [x] src/app/shared/ui/ removed — ZardUI from Common-UI submodule
- [x] Login UI polished — Piramal Swasthya branding, eye icon, input icons, forgot password link
- [x] PR #1 updated — 23 commits, waiting for Mithun review

---

## In Progress / Blocked 🚧
- [ ] PR #1 merge — waiting for Mithun review
- [ ] Gopi's UI skill — will use for further UI polish
- [ ] 5002 "logged-in-elsewhere" confirm path (B11)
- [ ] SessionStorage encryption (deferred)
- [ ] Auth guard on /role-selection (deferred to P1)

---

## Learnings & Gotchas 💡
- Old 104 uses raw Authorization header (no Bearer prefix)
- apiKey appended as ?apikey= (APIMAN gateway)
- 5002 has two branches — "kick other session" vs hard logout
- Login must call AuthStore.setSession() not write sessionStorage directly
- Subagents caught real bugs before commit every time
- Always check UAT live app before building any screen!
- Mithun: "barebones Claude code won't cut it" — UI needs polish
- Common-UI submodule: use HTTPS URL not SSH (CI requirement)

---

## Risks ⚠️
- PR #1 still not merged — blocking start of next features
- 118 components, mid-August deadline — need to ship fast
- Gopi's UI skill pending — login UI can be improved further

---

## Metrics
- New files committed this week: 50+
- PRs opened: PR #49 (Common-UI — merged), PR #1 (104-NEXT — pending)
- Build size: ~155 kB (login chunk)
- Login screen: polished with branding + icons

---

## Next
- Mithun reviews + merges PR #1
- Friday team sync
- Start role-selection component (P1) after merge
- Apply Gopi's UI skill when available