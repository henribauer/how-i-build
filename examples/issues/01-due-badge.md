# 01: due-count pill on subject cards

Status: ready-for-agent

Spec: [../spec.md](../spec.md)

## Acceptance criteria

- [x] Pill shows the due count for each subject, `99+` above 99
- [x] No pill when the count is 0
- [x] Unreadable review log means no pill, error logged, hub still renders
- [x] Count computed after first paint
- [x] Unit test covers 0 / 12 / 150 / unreadable

## Comments

### 20260911 — Claude Code (builder)

Built. Checks: `node --test` 48/48 pass, lint clean. Review to run: `/codex:review`.
UI changed, so the design review also goes to Codex with fresh screenshots.

### 20260911 — review triage

| Finding | Severity | Decision |
|---|---|---|
| Count reads the whole log on every hub open | medium | **Fixed.** Cached per day, invalidated when a review is written. |
| Pill contrast 3.9:1 on the green wash | high (a11y) | **Fixed.** Text token darkened, now 5.2:1. Blocked the commit until fixed. |
| Suggests extracting a `Badge` component | low | **Rejected.** One use site, an abstraction can wait for a second one. |

Fixes under 30 lines, no logic change beyond the cache, so no second review.

### 20260911 — handoff

- Done: ticket complete, committed as `feat(hub): due-count pill on subject cards`.
- Next: none in this spec. Overdue state stays parked.
- Open findings: none.
- Uncommitted files: none.
