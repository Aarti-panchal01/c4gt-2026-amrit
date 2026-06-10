# Weekly Logs — C4GT 2026 Internship

One file per week, named `week-01.md` through `week-13.md` (Jun 10 – Sep 10, 2026).

## What Goes in Each File

**Summary:** 1–2 sentences on what you accomplished this week

**Completed:** List of tasks + outcomes

**In Progress / Blocked:** What's still ongoing or stuck

**Learnings & Gotchas:** Patterns discovered, issues hit, decisions made

**Risks & Mitigations:** What could go wrong, how you'll handle it

**Metrics:** Services migrated, tests passing, bundle size, etc.

**Next Week's Plan:** What you're starting Monday

## Weekly Entry Template

```markdown
# Week 1: Jun 10–16 — Foundation & ZardUI Spike

## Summary
Set up Angular 20 workspace scaffolding + dual-build CI. Conducted ZardUI component-coverage spike to validate it covers table/datepicker/dialog needs.

## Completed ✅
- [x] New Angular 20 standalone workspace (Node 22, TS 5.6+, esbuild)
- [x] Port src/environments/ → fileReplacements
- [x] ZardUI spike: tested button, input, select, dialog, table, datepicker

## In Progress / Blocked 🚧
- [ ] Commit workspace to feature/angular-20-prep branch

## Learnings & Gotchas 💡
- ZardUI tables are thin — will need CDK DataSource or custom wrapper
- Angular 20 esbuild: ~4s build time (vs Webpack 12s)
- Keep Bootstrap 3 CSS until Phase 2 to avoid layout regressions

## Risks & Mitigations ⚠️
- **Risk:** ZardUI table gap might affect Phase 2
  - **Mitigation:** Budget 3–4 days in Wk11 for table component if needed

## Metrics
- Files scaffolded: 32
- Spike findings documented: 1
- Build time: 4s

## Next Week's Plan
- HTTP layer refactor (Week 2, critical path)
- Rewrite interceptor pattern
- Test force-logout flow
```

## Workflow

**Daily (throughout week):**
- Update the "Completed" section with today's work
- Note blockers in "In Progress / Blocked"
- Add learnings to "Learnings & Gotchas"

**Friday (end of week):**
- Finalize all sections
- Add metrics
- Fill "Next Week's Plan"
- Commit: `git add weekly-logs/ && git commit -m "docs(logs): week N summary"`

## File Naming

`week-NN.md` where NN = week number

- `week-01.md` ← Jun 10–16
- `week-02.md` ← Jun 17–23
- ... through `week-13.md` ← Sep 2–10
