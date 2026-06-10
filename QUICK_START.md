# Daily Workflow — Quick Start

How to log your work each day.

---

## Morning

1. Open `daily-logs/DN-MonDD-YY.md` (e.g., D2-June11-26.md)
2. Fill "Task" — what are you starting?
3. Get coding

## During the Day

Found a **blocker**? → Add to daily file immediately (don't wait)

Made a **discovery**? → Add to daily file under "Learnings"

---

## End of Day

1. Open `daily-logs/DN-MonDD-YY.md`
2. Fill "What I Did" — bullets of actual work
3. Fill "Blockers Hit" — any show-stoppers
4. Fill "Learnings" — new patterns, gotchas, decisions made
5. Fill "Tomorrow" — what's next

---

## End of Week (Friday)

1. Open `weekly-logs/week-NN.md`
2. Summarize the week (what you built, metrics, key learnings)
3. Copy any blockers from daily files into `docs/blockers.md`
4. Update `docs/decisions.md` with any confirmed choices

---

## Files to Know

| File | Purpose | Update |
|------|---------|--------|
| `daily-logs/DN-MonDD-YY.md` | Today's standup | Daily |
| `weekly-logs/week-NN.md` | Weekly summary + metrics | Friday |
| `docs/blockers.md` | Persistent blockers | As they surface |
| `docs/decisions.md` | Design choices | As they're made |

---

## Example Daily Entry

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

---

**Questions?** See [DOCS_STRUCTURE.md](DOCS_STRUCTURE.md) for the full system.
