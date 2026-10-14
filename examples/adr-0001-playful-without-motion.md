# 0001: A playful register, carried by colour and shape, never by motion

Status: accepted (20260910)

## Context

The brief was: "everything in the app should be playful, but it should not hinder the
functionality."

"Playful" was the third style word this interface had been given without a definition. The first
two ("liquid chrome", "colour") each cost a full redesign pass because they were built before they
were pinned down. So this record was written before any rule was touched.

The two worst surfaces had the same problem: a list of seventeen subjects that differed only in
their text, and a language picker you couldn't see. The missing playfulness and the missing
usability were the same thing. Peer items on an index carried no identity.

## What the references do

Headspace, SuperHi, Deel, Duolingo and Babbel, read on Mobbin: in every case the playfulness comes
from ground colour, radius and small tags. Hue does work: it identifies, groups and marks state.
None of them is playful because something moves.

## Decision

Playful means five static devices, each repeating information that is already on screen:

1. Tinted grounds on index items, one tone per group.
2. A larger radius on content cards (not on controls).
3. Pill tags that carry metadata. An empty pill is not allowed.
4. One loud element per surface. A second one is a bug.
5. One simple shape per tinted card, no illustrations.

No motion is added.

## Consequences

- Every future "make it more playful" request is answered from this list first.
- Any new device must also carry information, or it doesn't go in.
