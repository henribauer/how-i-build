# Spec: due badges in the study hub

Status: approved (20260910)

## Diverge

Six W-questions, one line each:

- **Why:** I open subjects at random and waste the first minutes of a session.
- **Who:** me, at the start of a study session.
- **What:** a signal of where work is waiting.
- **Where:** the study hub, the first screen after launch.
- **When:** every launch, mostly on battery.
- **How:** read-only, from the existing review log.

Ideas (mine first, then the agent's), with the ones going forward marked ✅:

- Show what's waiting per subject
  - ✅ A due count on each subject card (mine)
  - Sort the cards by due count (agent's variant)
- Tell me where to start
  - One "start here" card at the top (mine)
  - Reverse it: hide subjects with nothing due (agent, reversal)
- From my side instead of the app's: "how long will today take?" as an estimate in minutes (agent)
  → parked, see deferred decisions

## Goal

On the study hub, each subject card shows how many flashcards are due today, so I can see where to
start without opening every subject.

## Acceptance examples

| Given | Then |
|---|---|
| A subject with 12 cards due | The card shows a pill: `12 due` |
| A subject with 0 cards due | No pill. An empty pill is decoration and isn't allowed. |
| More than 99 due | `99+ due` |
| The review log can't be read | The card renders without a pill, and the error goes to the log. The hub never fails to load. |

## Failure boundaries

- Read-only: this feature never writes to the review log.
- Counting must not block the first paint of the hub.

## Design

Existing project, so Mobbin references plus a HIG review.

| Reference | What we borrow |
|---|---|
| SuperHi course cards | Small pill tags at the foot of the card that *are* the metadata |
| Duolingo home | State shown as a small badge on the row it belongs to, nothing louder |

Tokens: `--tone-*-wash` for the pill ground, `--r-pill` radius, existing type scale. No motion.

## Six Hats

| Hat | Line |
|---|---|
| Facts | The review log already holds due dates; no new data needed. |
| Gut feeling | I'd use this every day. Sorting the cards would feel jumpy. |
| Risks | Counting slows the first paint; a broken log breaks the hub. Both are covered in the failure boundaries. |
| Benefits | Small, read-only, one ticket. |
| Best alternative not taken | A minutes-per-day estimate: more useful, but needs timing data we don't log yet. |

## Deferred decisions

- Overdue vs. due today as separate states: parked until the count is in use.
- A "how long will today take" estimate: needs per-card timing in the review log first.

## Tickets

- [01-due-badge](issues/01-due-badge.md)
