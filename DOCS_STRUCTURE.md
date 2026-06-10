# Documentation Structure — C4GT 2026 AMRIT Helpline104-UI

How to organize and maintain your internship documentation.

---

## Folder Layout

```
c4gt-2026-amrit/
├── README.md                          # Main overview + internship details
├── AGENTS.md                          # Repository guidelines
├── QUICK_START.md                     # Daily workflow
├── INDEX.md                           # This index
│
├── docs/
│   ├── decisions.md                   # Architectural decisions
│   └── blockers.md                    # Known blockers + open questions
│
├── daily-logs/
│   ├── readme-daily-logs.md           # Daily log format + guidelines
│   ├── D1-June10-26.md                # Day 1 standup
│   ├── D2-June11-26.md                # Day 2 standup
│   └── ... through D92-Sep10-26.md
│
├── weekly-logs/
│   ├── readme-weekly-logs.md          # Weekly log format + guidelines
│   ├── week-01.md                     # Jun 10–16 progress
│   ├── week-02.md                     # Jun 17–23 progress
│   └── ... week-13.md                 # Sep 2–10 progress
│
├── research/
│   └── [spike findings, as discovered]
│
└── _archive/
    └── [old files after internship ends]
```

---

## File Purposes

### `docs/`
**Stable reference material** — decisions and known issues.

- **decisions.md** — architectural choices + rationale (e.g., "Why fresh workspace vs ng update?")
- **blockers.md** — known issues, workarounds, open questions

### `daily-logs/`
**Daily standup** — quick snapshot of each day's work.

- One file per day (D1-June10-26.md through D92-Sep10-26.md)
- Format: "What I did today" (bullets), blockers, learning, tomorrow's plan
- See `readme-daily-logs.md` for format guide

### `weekly-logs/`
**Your journal** — weekly progress, daily standup, learnings.

- One file per week (week-01.md through week-13.md)
- Updated daily with what you did, blockers hit, learnings
- Metrics (services migrated, tests passing, etc.) updated end-of-week
- See `readme-weekly-logs.md` for format guide

### `research/`
**Spikes & reference implementations** — validation work before committing.

- Created as-needed (e.g., zardui-component-coverage.md)
- Referenced in weekly-logs when you use the findings

### `decisions/` & `_archive/`
- **decisions/** — detailed design docs if needed
- **_archive/** — old files after internship ends (for reference)

---

## How to Update Daily

1. **Open** `daily-logs/DN-MonDD-YY.md` (e.g., D2-June11-26.md)
2. **Fill** "What I Did" with bullets
3. **Note** any blockers immediately (don't wait till Friday)
4. **Check** your learning/discovery — add to docs/ if it's a decision or blocker

At **end of week:**
1. **Summarize** in `weekly-logs/week-NN.md`
2. **Update** any docs/ files based on the week's findings

