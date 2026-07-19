# Exercise 2 — Compute Fully-Loaded CAC per Channel

**Goal:** Compute media-only CAC, fully-loaded CAC, and the trailing-vs-blended split from `channel_costs` and `customers`, exactly as Lecture 2 did, writing every query yourself.

**Estimated time:** 45 minutes.

## Setup

```sql
SELECT COUNT(*) FROM channel_costs;   -- must print 24
```

Create `solutions.sql`. Label each answer with a `-- Task N` comment.

## Tasks

1. **New customers per month per channel.** Group `customers` by `channel, signup_month` and count rows. *(Expected, spot check: `organic_content` in `2025-01-01` → 1; `2025-11-01` → 4.)*

2. **Media-only CAC, full year.** For each channel, `SUM(ad_spend) ÷ total new customers for the year`. *(Expected: `paid_search` → $900.00; `organic_content` → $171.43.)*

3. **Fully-loaded CAC, full year (blended).** For each channel, `SUM(ad_spend + team_cost) ÷ total new customers for the year`. *(Expected: `paid_search` → $2,000.00; `organic_content` → $2,571.43.)*

4. **The hidden-cost gap.** For each channel, compute `fully_loaded_cac − media_only_cac` and express it as a percentage of the fully-loaded number (i.e., "what share of the real CAC would you have missed using media spend alone?"). *(Expected: `paid_search` → the gap is $1,100.00, 55% of the fully-loaded number; `organic_content` → the gap is $2,400.00, 93.3% of the fully-loaded number.)*

5. **Trailing-quarter CAC.** Restrict Task 3's logic to `cost_month >= '2025-10-01'` (Q4 only) for both the spend and the new-customer count. *(Expected: `paid_search` → $2,000.00 unchanged; `organic_content` → $1,636.36.)*

6. **H1 vs. Q4, side by side.** Produce one result set with columns `channel, h1_cac, q4_cac, full_year_cac`, using `cost_month < '2025-07-01'` for H1. *(Expected: `paid_search` → 2000.00 / 2000.00 / 2000.00 across the board; `organic_content` → 4000.00 / 1636.36 / 2571.43.)*

7. **Blended CAC across both channels.** One number: total loaded spend across **both** channels for the year, divided by total new customers across **both** channels. *(Expected: $2,250.00.)*

## Expected result (spot checks)

- Task 2 → $900.00 / $171.43.
- Task 3 → $2,000.00 / $2,571.43.
- Task 5 → $2,000.00 / $1,636.36.
- Task 7 → $2,250.00.

## Done when…

- [ ] `solutions.sql` has all 7 tasks with matching values.
- [ ] You can explain, without looking it up, why `organic_content`'s Task 4 gap (93.3%) is so much larger than `paid_search`'s (55%).
- [ ] You can state, in one sentence, why Task 6's three numbers for `organic_content` tell a completely different story than the single blended number in Task 3 does alone.
- [ ] You understand why "blended CAC" is a single company-wide input (Task 7) but "channel CAC" (Tasks 3, 5, 6) is what actually drives a scale/pause decision.

## Stretch

- Lumen Metrics is considering hiring one more content writer for `organic_content` at $4,500/month, expecting it to lift new signups from 4/month to 6/month (Q4 run rate). Recompute Q4 CAC under that assumption. Does it improve or worsen the trailing CAC? Show your work.
- If `paid_search`'s ad platform raised prices 15% (i.e., `ad_spend` up 15%, `team_cost` unchanged, volume unchanged), what would the new fully-loaded CAC be?

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-05/exercise-02/`.
