# Exercise 3 — Measure Onboarding Step Drop-off

**Goal:** Build the full six-step onboarding funnel in SQL, compute step-over-step conversion with a window function, and formally locate the worst drop.

**Estimated time:** 60 minutes.

## Setup

Connected, seed loaded. New tasks in `solutions.sql` under `-- Task N` comments, plus a short `funnel-report.md`.

## Tasks

1. **Funnel counts.** Write a query (or six, if you prefer, before combining) that returns `users_reached` for each of the six ordered steps: `signed_up`, `verified_email`, `created_first_board`, `created_first_card`, `invited_teammate`, `completed_first_task`. *(Expected: 400, 322, 256, 209, 116, 101.)*

2. **Step-over-step conversion with `LAG`.** Build the single query from Lecture 3 §3 (a `UNION ALL` of the six counts feeding a `LAG() OVER (ORDER BY step_order)`), returning `step`, `n`, `prev_n`, `pct_converted`, `pct_dropped` for all six steps. *(Expected `pct_dropped`: —, 19.5%, 20.5%, 18.4%, 44.5%, 12.9%.)*

3. **Find the worst step programmatically.** Using the result of Task 2, write a query that returns the single row with the highest `pct_dropped` (excluding the first row, which has no `prev_n`). Don't eyeball it — make SQL find it. *(Expected: `created_first_card → invited_teammate`, 44.5%.)*

4. **Sanity-check against cumulative reach.** Confirm that `created_first_card`'s cumulative reach (209/400 = 52.2%) times its step conversion to `invited_teammate` (55.5%) gives you `invited_teammate`'s cumulative reach. Show the arithmetic in a comment. *(52.2% × 55.5% ≈ 29.0% — matches Task 1's Lecture 1 reach number.)* This is a useful check any time you compute a funnel two different ways.

5. **Segment the worst step by channel.** For users who reached `created_first_card`, compute the conversion rate to `invited_teammate` broken out by `channel`. Does any channel convert meaningfully worse than the others at this specific step? *(You should find the channels are all fairly close — roughly 50–58% conversion at the card stage across channels — which is itself a useful finding: this stall isn't a channel-quality problem, it's a product/onboarding-flow problem.)*

## Done when…

- [ ] Task 1's six counts match exactly.
- [ ] Task 2's query uses `LAG()` — not a self-join or six separate hardcoded subtractions — and returns all five `pct_dropped` values.
- [ ] Task 3 returns exactly one row: `created_first_card → invited_teammate`.
- [ ] Task 4's arithmetic check is shown and comes out within rounding of 29.0%.
- [ ] `funnel-report.md` states the worst step, its drop percentage, and one sentence connecting it to Task 5's channel finding (or lack of one).

## Stretch

- Rebuild the funnel restricted to `plan = 'Pro'` only, then `plan = 'Free'` only. Does the worst step move, or stay in the same place?
- Add a seventh "step" to the funnel: `connected_integration`, positioned after `completed_first_task`. Recompute step-over-step conversion including it. Does it change which transition is "worst," and should it — given what you know about `connected_integration`'s reach from Exercise 1?

## Submission

Commit `solutions.sql` and `funnel-report.md` to your portfolio under `c38-week-03/exercise-03/`.
