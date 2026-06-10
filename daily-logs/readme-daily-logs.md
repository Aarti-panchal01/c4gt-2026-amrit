# Daily Logs — C4GT 2026 Internship

One file per day, named `D1-June10-26.md`, `D2-June11-26.md`, etc.

## What Goes in Each File

- What task you started
- What you completed today
- Any blockers hit
- Learnings/discoveries
- Plan for tomorrow

**Format:** Quick, scannable bullet points (not essays)

## Daily Entry Template

```markdown
# D1 — Jun 10, 2026

## Task
- [ ] Finish HTTP interceptor rewrite
- [ ] Test with login flow

## What I Did
- Rewrote http.interceptor.ts → functional HttpInterceptor
- Added HttpContext tokens for IS_SILENT_REQUEST
- Tested 401/403 redirect flow

## Blockers Hit
- Socket.io constructor commented out — need to check if safe to ignore

## Learnings
- HttpContext tokens cleaner than two-class pattern
- Force-logout (5002) flow must be tested early

## Tomorrow
- Port auth services to HttpClient
- Test concurrent-login dialog
```

## Workflow

**Each morning:**
- Open today's file (e.g., `D2-June11-26.md`)
- Fill "Task" — what are you starting?
- Keep notes throughout day under "What I Did"

**Each evening:**
- Complete "What I Did", "Blockers Hit", "Learnings", "Tomorrow"
- Optional: quick commit if major breakthrough

**Each Friday:**
- Collect the week's daily notes
- Write weekly summary in `weekly-logs/week-NN.md`
- Commit both: `git commit -m "docs(logs): week N summary"`

## File Naming Format

`DN-MonthDD-YY.md`

- `D1-June10-26.md` ← Day 1
- `D2-June11-26.md` ← Day 2
- `D3-June12-26.md` ← Day 3
- ... through Sep 10
