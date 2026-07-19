# Exercise 1 — Estimate WTP from Survey Data

**Goal:** Build all four Van Westendorp curves from `wtp_survey` in SQL, find PMC/PME/OPP/IDP, then check the blended answer against the segment-level truth — exactly as Lecture 1 did, but this time you write every query.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM wtp_survey;   -- must print 30
```

Create `solutions.sql` for the SQL tasks and `analysis.py` (or a notebook) for the pandas tasks. Label each with a `-- Task N` / `# Task N` comment.

## Tasks

### SQL (Tasks 1–5)

1. **Order sanity check.** Write a query that returns any respondent where `too_cheap < cheap < expensive < too_expensive` is **violated**. *(Expected: 0 rows — the seed data is already clean, but you should always run this check first on real survey data.)*

2. **The four cumulative curves.** Build the price-grid query from Lecture 1 §2.1 yourself — don't copy it verbatim, re-derive each `CASE`/`FILTER` condition from the table in §2.1. Produce `price, tc_pct, bargain_pct, getting_exp_pct, te_pct` for `price` in `{25, 45, 55, 65, 90}`. *(Expected, spot check: at price 45 → TC 33.3%, Bargain 53.3%, Getting Expensive 30.0%, TE 13.3%.)*

3. **Segment medians.** Group `wtp_survey` by `segment` and compute the median of all four columns. *(Expected: `indie` too_expensive median = 47.5; `team` too_expensive median = 104.5; `agency` too_expensive median = 256.0.)*

4. **Where does $39 sit?** For each segment, count how many respondents have `too_cheap >= 39` (still suspicious of quality at $39) and how many have `too_expensive <= 39` (would already refuse to buy at $39). *(Expected: `indie` → 0 too-cheap, 3 too-expensive (30% of indie already finds $39 unaffordable); `team` → 1 too-cheap, 0 too-expensive; `agency` → all 10 too-cheap, 0 too-expensive — every single agency respondent would distrust ScopeIQ's quality at $39.)*

5. **Behavioral cross-check.** Join `wtp_survey` to `accounts_usage` on `respondent_id = account_id`, and compute average `mtu` and average `too_expensive` by segment. *(Expected: `indie` avg_mtu ≈ 1,270; `team` avg_mtu ≈ 15,280; `agency` avg_mtu ≈ 85,000 — usage and stated WTP should move together, segment by segment.)*

### pandas (Tasks 6–8)

Bring your Task 2 curve values into a `DataFrame` (typed in from your SQL output, or loaded via `pandas.read_sql`) for the rest of this exercise.

6. **Find OPP.** Using the blended (all-30) curves, find the price where the Too Cheap curve and the Too Expensive curve cross — scan a price grid from $1 to $150 in $1 steps, and report the price where `TC(p) - TE(p)` changes sign (linear-interpolate between the two grid points that bracket the sign change for a precise answer). *(Expected: OPP ≈ $60.50, where both curves sit at 28.3%.)*

7. **Find PMC and PME.** Using the same technique, find PMC (Too Cheap ∩ Getting Expensive) and PME (Bargain ∩ Too Expensive). *(Expected: PMC ≈ $50.00 at 33.3%; PME ≈ $68.00 at 33.3%.)*

8. **The blended-vs-segmented writeup.** In 150 words or fewer: ScopeIQ's actual price is $39, which sits *below* your Task 7 PMC of $50. Taken at face value, that says "raise the price." Using your Task 3 segment medians, explain **why that's true for some segments and false for others** — be specific about which segment(s) $39 is already too high for, and which segment it's suspiciously low for.

## Expected result (spot checks)

- Task 1 → 0 rows (no order violations).
- Task 2 → price 45: TC 33.3%, Bargain 53.3%, Getting Expensive 30.0%, TE 13.3%.
- Task 3 → segment medians: `indie` (12.5, 21.0, 35.0, 47.5); `team` (29.0, 49.5, 78.5, 104.5); `agency` (77.5, 124.0, 194.0, 256.0).
- Task 6 → OPP ≈ $60.50 (28.3% resistance).
- Task 7 → PMC ≈ $50.00 (33.3%); PME ≈ $68.00 (33.3%).

## Done when…

- [ ] `solutions.sql` has all 5 SQL tasks, each returning the expected rows/values.
- [ ] `analysis.py` has all 3 pandas tasks, with printed OPP/PMC/PME matching the expected values within $0.50.
- [ ] You can explain, out loud, why `agency` respondents might distrust ScopeIQ's quality *because* the price is too low, not too high.
- [ ] Your Task 8 writeup names the specific segment(s) for which $39 is already past their median "expensive" threshold.

## Stretch

- Find IDP (Bargain ∩ Getting Expensive) the same way you found OPP/PMC/PME. *(Expected: IDP ≈ $59.00 at 40.0%.)*
- Recompute OPP, PMC, and PME **within the `team` segment only** (10 respondents). How far do they move from the blended numbers, and how close do they land to the $69 `Growth` price Lecture 2 chose?

## Submission

Commit `solutions.sql` and `analysis.py` to your portfolio under `c38-week-09/exercise-01/`.
