# Week 9 — Challenges

Two harder, less-scaffolded problems, ~60 min each. The exercises told you exactly which query to write; these tell you the *business* problem and leave the modeling choices to you. There isn't always one right numeric answer — there is always a right way to reason about it, and you'll be graded (by yourself, against the checklist) on whether your numbers are internally consistent and your assumptions are stated, not on hitting a single target.

1. **[Challenge 1 — Repackage for a Target Segment](challenge-01-repackage-for-a-segment.md)** — some `Growth` customers want a `Scale`-only feature. Design a fix that doesn't force a full tier jump or give the feature away.
2. **[Challenge 2 — Forecast a Price Increase](challenge-02-forecast-a-price-increase.md)** — model the real 1,875-customer impact of raising the `Growth` tier from $69 to $79, with grandfathering.

## Before you start

- You've finished all three exercises — these challenges reuse the tier prices from Exercise 2 and the demand curve from Exercise 3 without re-deriving them for you.
- Keep every number traceable to a query or a stated assumption. "I estimated 25%" is fine; "25%, because [reasoning]" is what separates a defensible forecast from a guess with a percent sign on it.

## How these get harder

Every exercise this week gave you the exact SQL to write and told you which number to check it against. These challenges give you a decision a real pricing team would actually have to make, and the data to make it with — you choose the add-on price, the elasticity assumption, the discount terms. Write down every assumption in the file you submit; an unstated assumption is the single most common reason a forecast falls apart under questioning.
