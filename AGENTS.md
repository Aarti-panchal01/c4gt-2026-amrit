# AGENTS.md — Repository Guidelines

This document defines conventions, architecture patterns, and guidance for Claude Code agents working on this AMRIT Helpline104-UI migration project.

---

## Project Overview

**AMRIT Helpline 104** is the web front-end for India's public-health call-center helpline (operated by Piramal Swasthya Management and Research Institute). This repository documents the **C4GT 2026 internship** migration from **Angular 4.1.3 → Angular 20** with **ZardUI** component library integration.

- **Scope:** 118 components, 65 services, 138 templates, single 530-line app.module.ts
- **Internship period:** Jun 10 – Sep 10, 2026 (13 weeks)
- **Code repo:** https://github.com/PSMRI/Helpline104-UI-NEXT
- **Branch strategy:** `main` → `angular-zard-migration` (migration branch) → individual feature branches → PRs back into `angular-zard-migration` → merge to `main` only after full QA
- **Documentation repo:** https://github.com/Aarti-panchal01/c4gt-2026-amrit (this repo)

---

## Documentation Structure

```
docs/
├── decisions.md                     # Architectural decisions
├── blockers.md                      # Known blockers + open questions
├── migration-plan.md                # 13-week Phase 1 + Phase 2 breakdown
├── architecture/                    # Detailed architecture analysis
│   ├── auth-flow.md                 # Login sequence, tokens, guards
│   ├── call-flow.md                 # End-to-end call lifecycle
│   ├── services-catalog.md          # All 67 services + endpoints
│   └── http-interceptor.md          # Current pattern + Angular 20 replacement
└── research/                        # Spike findings & comparison
    └── 104-vs-1097-comparison.md    # Shared code analysis + Common-UI tiers

daily-logs/
├── readme-daily-logs.md             # Daily log guidelines
└── D1-June10-26.md, D2-June11-26.md, ... D92-Sep10-26.md

weekly-logs/
├── readme-weekly-logs.md            # Weekly log guidelines
├── week-01.md                       # Jun 10–16
├── week-02.md                       # Jun 17–23 (to be filled)
└── ... week-13.md                   # Sep 2–10 (to be filled)
```

---

## Decisions

### D1: Fresh Workspace Re-Platform (Not `ng update`) ✅

- **Why:** `ng update 4→20` fails at every step: @angular/http removal (v5+), RxJS 5→6 break (v6+), Material Md* removal (v9+), md2 death, .angular-cli.json removal (v6+)
- **Solution:** Build new Angular 20 workspace; port code; run old+new in parallel
- **Implication:** All code changes go to feature/angular-20-migration branch in the code repo; this docs repo tracks progress

### D2: "Do UI Once" (Material as Interim Only) ✅

- **Why:** Avoid touching 138 templates twice (Material refactor + ZardUI refactor)
- **Solution:** Phase 1 = mechanical Md*→Mat* rename only; skip Material M3/MDC visual QA
- **Implication:** Phase 1 UI looks rough; visual polishing happens in Phase 2 when ZardUI replaces Material

---

## Codebase Overview

**Scope:**
- 118 components (auth, roles, call-handling, reports, SIO services)
- 65 services (auth, HTTP, data, business logic)
- 138 templates (HTML/CSS/Bootstrap 3)
- 112 test specs (Jasmine/Karma, mostly untouched)
- 440+ jQuery calls (mostly in role components)
- Current stack: Angular 4.1.3 / @angular/http / RxJS 5 / Material beta / md2 / Bootstrap 3

**Target stack (Angular 20):**
- Standalone components + lazy routes
- HttpClient + functional HttpInterceptor
- RxJS 7 with .pipe()
- Material M3 as interim; ZardUI as Phase 2 replacement
- Tailwind + Angular CDK (Phase 2)
- esbuild instead of Webpack

For detailed architecture notes, see `docs/decisions.md` and `docs/blockers.md`.
