# C4GT 2026 — AMRIT Helpline104-UI: Angular 4 → Angular 20 Migration

> **Code for GovTech (C4GT) 2026** internship project
> Migrating the **AMRIT Helpline104-UI** front-end from **Angular 4 to Angular 20**, with **ZardUI** as the new component library.
> Organization: **AMRIT** by **Piramal Swasthya Management and Research Institute (PSMRI)**

[![C4GT 2026](https://img.shields.io/badge/C4GT-2026-orange)](https://www.codeforgovtech.in/)
[![Angular](https://img.shields.io/badge/Angular-4%20→%2020-dd0031)](https://angular.dev/)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

---

## �️ Table of Contents

1. [Overview](#-overview)
2. [My Role](#-my-role)
3. [Tech Stack](#-tech-stack)
4. [Migration Scope](#-migration-scope)
5. [Project Timeline](#-project-timeline)
6. [Weekly Logs](#-weekly-logs)
7. [Architecture Notes](#-architecture-notes)
8. [Getting Started](#-getting-started)
9. [Project Links](#-project-links)
10. [Mentorship](#-mentorship)
11. [License](#-license)

---

## �📋 Overview

**Helpline104-UI** is the web front-end used by call-center agents and clinical staff of India's **"104" public-health helpline** — providing medical advice, counseling, grievance resolution, directory services, epidemic-outbreak information, medicine prescription, and maternal/child death review. It is one module of the larger **AMRIT** (Accessible Medical Records via Integrated Technology) EHR platform.

This repository documents my C4GT 2026 internship work: re-platforming the application from a legacy **Angular 4.1.3** codebase to **Angular 20**, and replacing its abandoned UI stack (Angular Material beta, `md2`, jQuery/Bootstrap 3) with **ZardUI** (a modern Tailwind + Angular CDK component library).

- **Intern:** Aarti Panchal
- **Internship period:** **June 10 – September 10, 2026** (13 weeks)
- **Documentation repo:** https://github.com/Aarti-panchal01/c4gt-2026-amrit

---

## 👩‍💻 My Role

As the C4GT contributor on this project, I am responsible for:

- **Codebase analysis** — documenting the existing Angular 4 architecture (auth flow, call flow, ~67 services, the custom HTTP interceptor pattern) and identifying migration blockers.
- **Phase 1 — Framework migration:** re-platforming to Angular 20 (standalone components, functional `HttpInterceptor`s, RxJS 7, lazy routing, `angular.json`/esbuild build).
- **Phase 2 — UI migration:** integrating **ZardUI** + Tailwind CSS, replacing Angular Material/`md2`/jQuery components.
- **Quality & handover:** test migration (Karma/Jasmine → modern setup, Protractor → Playwright), visual/accessibility QA, and maintaining this documentation.

---

## ️ Tech Stack

| Area | From (Angular 4) | To (Angular 20) |
|------|------------------|-----------------|
| Framework | Angular 4.1.3 (NgModule) | Angular 20 (standalone components) |
| HTTP | `@angular/http` + custom `Http` subclasses | `HttpClient` + functional interceptors |
| Reactive | RxJS 5 (patch operators) | RxJS 7 (pipeable operators) |
| UI library | Angular Material beta, `md2`, jQuery, Bootstrap 3 | **ZardUI** + Tailwind CSS |
| Build | Angular CLI 1.2.6, `.angular-cli.json`, Webpack | Angular CLI 20, `angular.json`, esbuild |
| Tooling | TSLint + codelyzer, Node 16, TS 3.9 | angular-eslint, Node 20.11+/22, TS 5.6+ |
| Testing | Karma/Jasmine, Protractor | Modern TestBed, Playwright |

---

## 🎯 Migration Scope

The codebase (≈118 components, 65 services, 138 templates) is migrated as a **fresh-workspace re-platform** — not an incremental `ng update` — because `@angular/http`, the Material `Md*` barrel, RxJS 5 patch operators, `md2`, and `.angular-cli.json` are all removed/abandoned in the target.

**Key work areas:**
- Custom `Http`-subclass interceptors → functional `HttpInterceptor`s (`provideHttpClient(withInterceptors(...))`)
- RxJS 5 → 7 (`.pipe()` migration across ~108 files)
- Single root `NgModule` → standalone components with lazy `loadComponent` routes
- Angular Material / `md2` / jQuery UI → ZardUI
- Replacement of abandoned libraries (`ng2-smart-table`, `angular2-toaster`, `angular2-csv`, `ng2-validation`)

---

## 📅 Project Timeline

| Phase | Weeks | Dates | Focus |
|-------|-------|-------|-------|
| **Phase 1** | 1–9 | Jun 10 – Aug 11 | Angular 4 → Angular 20 upgrade |
| **Phase 2** | 10–13 | Aug 12 – Sep 10 | ZardUI integration & UI migration |

**Milestones:**
- **Wk 2** — HTTP layer + core infra on HttpClient / RxJS 7
- **Wk 5** — All services + directives/pipes compile on Angular 20
- **Wk 9** — 🏁 Phase 1 complete: full app runs on Angular 20
- **Wk 12** — All components on ZardUI; jQuery/Bootstrap/Material removed
- **Wk 13** — 🏁 QA complete, build parity, final sign-off

> See [`weekly-logs/`](weekly-logs/) for the full week-by-week breakdown.

---

## 📝 Weekly Logs

| Week | Dates | Summary | Log |
|------|-------|---------|-----|
| 1 | Jun 10–16 | Workspace scaffolding, ZardUI spike | [week-01](weekly-logs/week-01.md) |
| 2–13 | Jun 17–Sep 10 | TBD (planning session Jun 15) | pending |

---

## 🏛️ Architecture Notes

Detailed analysis of the existing system lives in [`docs/`](docs/):

- **Authentication & login flow** — crypto-js encryption, multi-role login, route guards, session handling
- **End-to-end call flow** — C-Zentrix CTI integration, screen-pop, case-sheet, CDSS, closure
- **Services catalog** — all 67 services and their backend microservice targets
- **HTTP interceptor pattern** — the custom `Http`-subclass approach and its modern equivalent

---

## 🚀 Getting Started

```bash
# Clone the application repo (not this docs repo)
git clone https://github.com/PSMRI/Helpline104-UI.git
cd Helpline104-UI

npm install --force
npm start          # dev server at http://localhost:4200/#/login
```

---

## 🔗 Project Links

| Resource | Link |
|----------|------|
| C4GT 2026 | https://www.codeforgovtech.in/ |
| AMRIT platform (main repo) | https://github.com/PSMRI/AMRIT |
| Helpline104-UI (app repo) | https://github.com/PSMRI/Helpline104-UI |
| Helpline104-UI DeepWiki | https://deepwiki.com/PSMRI/Helpline104-UI |
| Piramal Swasthya (PSMRI) | https://piramalswasthya.org/ |
| ZardUI | https://www.zardui.com/ |
| Angular | https://angular.dev/ |
| Community (Discord) | https://discord.gg/FVQWsf5ENS |

---

## 🤝 Mentorship

This project is carried out under the Code for GovTech 2026 program, mentored by the AMRIT / Piramal Swasthya engineering team.

- **Organization:** Piramal Swasthya Management and Research Institute
- **Mentor:** Dr. Mithun James (Principal Consultant, PSMRI)
- **Contact:** amrit@piramalswasthya.org

---

## 📄 License

This project follows the AMRIT platform license — GNU General Public License v3.0 (GPLv3).
See the AMRIT repository (https://github.com/PSMRI/AMRIT) for details.

---

**Last Updated:** Jun 10, 2026 (Day 1)
**Status:** ⏳ Day 1 setup complete; team planning meeting Jun 15
