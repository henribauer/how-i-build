# Spec: due badges in the study hub

Status: approved (20260910)

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

## Deferred decisions

- Overdue vs. due today as separate states: parked until the count is in use.

## Tickets

- [01-due-badge](issues/01-due-badge.md)
