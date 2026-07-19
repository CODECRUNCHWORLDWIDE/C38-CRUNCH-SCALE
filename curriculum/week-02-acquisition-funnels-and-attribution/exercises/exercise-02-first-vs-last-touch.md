# Exercise 2 — Compare First- vs. Last-Touch Attribution

**Goal:** Build both single-touch attribution models from the `touches` table, reconcile each to total revenue, and explain — with your own query output — why they disagree about which channel matters most.

**Estimated time:** 90 minutes.

## Setup

Connected, 31 rows in `touches`, 10 rows in `conversions` summing to $1,590. New `solutions.sql` under `-- Task N` comments.

## Tasks

1. **First-touch, by channel.** Write the `ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts ASC)` query from Lecture 2 §2. Return `channel_name`, `conversions_credited`, `revenue_credited`, ordered by revenue descending. *(Expected 6 rows: Organic Search $498, Affiliate $349, Google Search - Nonbrand $347, Meta Ads $198, Referral $149, Google Search - Brand $49.)*

2. **Last-touch, by channel.** Flip the `ROW_NUMBER` sort to `DESC` and re-run. *(Expected 5 rows: Email $1,145, Google Search - Brand $198, Organic Search $149, Meta Ads $49, Google Search - Nonbrand $49.)*

3. **Reconcile both to total revenue.** Run `SELECT SUM(revenue) FROM conversions;` and confirm both Task 1's and Task 2's `revenue_credited` columns sum to that number. *(Expected: $1,590 in both cases.)* If either doesn't match, you have a bug — find it before continuing (check for a missing `touch_rank = 1` filter or a duplicate join row).

4. **The channel that flips.** Which channel has the largest *ratio* between its last-touch and first-touch revenue (in either direction)? Report the channel, both numbers, and the ratio. *(Expected: `Google Search - Brand`, $49 → $198, ~4.0x.)*

5. **The channel with zero under one model.** Name one channel that earns **$0** under first-touch and one that earns **$0** under last-touch. For each, write one sentence explaining *why* that channel structurally can't win credit under that model, given how it's used in this data (check its role in multi-touch paths, not just its name).

6. **Pick a side, then check yourself.** Before running anything new, predict: if Crunch's growth lead proposed cutting `Google Search - Nonbrand` spend in half to fund more `Google Search - Brand`, would you agree, based only on Tasks 1–2? Write your answer in a comment — you'll revisit it in Exercise 3 once CAC is on the table.

## Done when…

- [ ] Tasks 1 and 2 match the Expected values exactly (not approximately).
- [ ] Task 3 shows both totals equal to $1,590, or documents the bug you found and fixed.
- [ ] Task 4 correctly identifies `Google Search - Brand` and computes the ~4x ratio.
- [ ] Task 5 names two *different* channels (one per model) with a reasoned, path-specific explanation — not "it just doesn't show up."

## Stretch

- Compute first-touch and last-touch **conversion counts** (not revenue) for every channel, including ones with zero touches leading to a conversion (`Direct`). A `LEFT JOIN` from `channels` will be necessary — a plain `JOIN` silently drops the zero-count rows.
- For the two 3-touch users (9 and 18), write out their full attribution story in one sentence each: who gets credit under first-touch, who under last-touch, and whether that credit assignment feels fair to you. There's no right answer — defend your read.
- Modify Task 1 to break ties correctly: if two touches somehow had the identical `touch_ts` for one user, what would `ROW_NUMBER()` do, and is that safe? What would you add to the `ORDER BY` to make tie-breaking deterministic?

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-02/exercise-02/`.
