# 📚 Documentation Index — C4GT 2026 Internship
---

## 🗂️ Repository Contents

```
c4gt-2026-amrit/
│
├── README.md                    ← Project overview + internship details
├── AGENTS.md                    ← Repository guidelines for Claude
├── QUICK_START.md               ← Daily workflow guide
├── INDEX.md                     ← Index
│
├── docs/
│   ├── decisions.md             ← Architectural decisions
│   ├── blockers.md              ← Known blockers + open questions
│   ├── migration-plan.md        ← 13-week migration plan
│   ├── architecture/            ← Auth flow, call flow, services catalog, interceptor analysis
│   └── research/                ← Spike findings (104 vs 1097 comparison, ...)
│
├── daily-logs/
│   ├── readme-daily-logs.md     ← Daily log format guide
│   └── D1-June10-26.md ... D35-July16-26.md (one per day)
│
└── weekly-logs/
    ├── readme-weekly-logs.md    ← Weekly log format guide
    └── week-01.md ... week-06.md (one per week)
```

---

## 📖 Start Here

**New to this project?**

1. Read [README.md](README.md) (2 min) — what you're doing, timeline, tech stack
2. Read [QUICK_START.md](QUICK_START.md) (2 min) — how to log daily work
3. Bookmark [docs/decisions.md](docs/decisions.md) — why decisions were made
4. Bookmark [docs/blockers.md](docs/blockers.md) — known issues + open questions

**Are you Claude Code?** See [AGENTS.md](AGENTS.md) for project conventions.

---

## 📝 Weekly Progress

- [Week 1 (Jun 10–16)](weekly-logs/week-01.md) ✅ — Foundation: Angular 20 scaffold, ZardUI, migration plan, PR #1 + Common-UI PR #49
- [Week 2 (Jun 17–23)](weekly-logs/week-02.md) ✅ — P0 complete: login, role selection, full dashboard; PR #1 + #2 merged, PR #3 reviewed
- [Week 3 (Jun 24–30)](weekly-logs/week-03.md) ✅ — Auth flows, beneficiary registration, call shell, HAO workspace; PRs #3–#5 merged
- [Week 4 (Jul 1–7)](weekly-logs/week-04.md) ✅ — SNOMED/CDSS, all role workspaces, screenings, SIO tabs, P1 non-CTI; PRs #6–#10 merged
- [Week 5 (Jul 7–13)](weekly-logs/week-05.md) ✅ — E2E UAT verification, agent extras, supervisor area, CTI handshake + lifecycle; 12 PRs (#14–#25) merged
- [Week 6 (Jul 14–20)](weekly-logs/week-06.md) 🚧 — Post-merge E2E fixes, PR #27, documentation backfill

Daily logs D1–D35 (Jun 10 – Jul 16) live in [daily-logs/](daily-logs/). Week 7–13 logs will be filled as work progresses.

---

## 💾 Using This Repository

### Daily
- Open `weekly-logs/week-NN.md`
- Log what was completed today (bullets under "What I Did")
- Note any blockers immediately

### When a task is completed
- Update the checkbox in the week log
- If it's a design decision, add to `docs/decisions.md`
- If it's a blocker, add to `docs/blockers.md`

### Research & spikes
- Save findings to `docs/research/` as they're discovered
- Reference them in the weekly log

---

## 📍 Key Files

| File | Purpose | When to update |
|------|---------|---|
| `README.md` | Overview, timeline, role | Once at start |
| `docs/decisions.md` | Why you made each choice | As decisions made |
| `docs/blockers.md` | Issues, workarounds, open Qs | As blockers hit |
| `weekly-logs/week-NN.md` | Daily standup + learning | Daily |
| `docs/research/*` | Spike findings, reference code | As spikes complete |

---

**Questions?** Check [QUICK_START.md](QUICK_START.md) or contact Dr. Mithun James (mentor).
