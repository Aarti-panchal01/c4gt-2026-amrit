# Week 2 — Jun 16–22, 2026

**Status:** 🚧 IN PROGRESS

---

## Summary
P0 core foundation complete. HTTP interceptors, AuthStore, ConfigService,
SessionService committed. ZardUI components merged into Common-UI v2.
Waiting for Gopi's AMRIT theme file. Next: login component (P1).

---

## Completed ✅

**Day 7 (Jun 16):**
- [x] Added ZardUI components to Common-UI v2/ui/
- [x] Fixed all import paths + GPL-3.0 headers
- [x] PR #49 opened + approved by Mithun + merged ✅
- [x] Coordinated with Gopi on shared AMRIT theme

**Day 8 (Jun 17):**
- [x] ConfigService wired to environments/
- [x] AuthStore (signals: token, user, role, privileges, isAuthenticated)
- [x] SessionStorageService (plain wrapper, TODO encrypt)
- [x] SessionService (27-min idle timer, keepalive, force-logout)
- [x] HTTP interceptors: auth, error, loader (functional HttpInterceptorFn)
- [x] All wired in app.config.ts
- [x] Subagent review — caught 4 issues, fixed 2 immediately
- [x] Build passes ✅ — commit 241457d

---

## In Progress / Blocked 🚧
- [ ] Gopi's AMRIT theme file — waiting to apply to styles.css
- [ ] /login route — doesn't exist yet (B11)
- [ ] 5002 "logged-in-elsewhere" confirm path (B11)
- [ ] SessionStorage encryption (deferred)

---

## Learnings & Gotchas 💡
- Old 104 uses raw Authorization header (no Bearer prefix)
- apiKey appended as ?apikey= (APIMAN gateway)
- 5002 has two branches — "kick other session" vs hard logout
- Login must call AuthStore.setSession() not write sessionStorage directly
- Subagent review caught real parity gap before commit!

---

## Risks ⚠️
- Theme dependency on Gopi — blocking final PR #1 merge
- 118 components, mid-August deadline — need to ship fast

---

## Metrics
- New files committed this week: 17 (core foundation)
- PRs opened: 1 (Common-UI #49 — merged)
- Build size: 237.38 kB

---

## Next
- Apply Gopi's AMRIT theme → merge PR #1
- Build login component (P1)
- Wire /login route + routing skeleton