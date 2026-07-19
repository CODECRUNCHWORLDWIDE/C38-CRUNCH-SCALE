# Exercise 3 — Compute Per-Channel CAC

**Goal:** Join `ad_spend` to attributed conversions and produce a CAC report under two models side by side — the report Lecture 3 argued you always need, never just one number in isolation.

**Estimated time:** 60 minutes.

## Setup

Connected, 40 rows in `ad_spend` summing to $4,500 across 4 paid channels. New `solutions.sql` under `-- Task N` comments. You'll reuse your first-touch and last-touch queries from Exercise 2.

## Tasks

1. **Total spend by channel.** `SUM(spend_amount)` grouped by `channel_id`, joined to `channels` for the name. *(Expected: Google Search - Nonbrand $1,700, Google Search - Brand $400, Meta Ads $1,700, Affiliate $700.)*

2. **Last-touch CAC.** Join Task 1's spend to a last-touch conversion count **per paid channel only** (`WHERE channel_type = 'paid'`). Use `NULLIF` so a zero-conversion channel returns `NULL`, not an error. *(Expected: Google Search - Brand $200.00, Google Search - Nonbrand $1,700.00, Meta Ads $1,700.00, Affiliate `NULL`.)*

3. **First-touch CAC.** Same shape as Task 2, using first-touch conversion counts instead. *(Expected: Google Search - Brand $400.00, Google Search - Nonbrand $566.67, Meta Ads $850.00, Affiliate $700.00.)*

4. **One report, four columns.** Combine Tasks 1–3 into a single query: `channel_name`, `total_spend`, `first_touch_cac`, `last_touch_cac`. All four paid channels, one row each.

5. **The Affiliate question.** Affiliate has a real dollar spend ($700) and a real touch count (3 users touched), but a `NULL` last-touch CAC. Write two sentences: (a) why the CAC is `NULL` rather than `0`, and (b) whether "Affiliate has infinite CAC" or "Affiliate has undefined CAC" is the more accurate way to describe this to a stakeholder, and why the distinction matters when someone asks "should we cut Affiliate?"

6. **Revisit your Exercise 2 prediction.** Look back at what you wrote in Exercise 2, Task 6. Now that you can see `Google Search - Nonbrand`'s first-touch CAC (~$567) next to `Google Search - Brand`'s last-touch CAC ($200), does your original recommendation still hold? Write 3–4 sentences with your final answer, citing both CAC numbers and (if you want the stronger argument) the upstream-path check from Lecture 3 §3.

## Done when…

- [ ] Task 4's four-column report matches every Expected value in Tasks 1–3.
- [ ] Task 2 and Task 3 both correctly show `Affiliate` with a spend figure but a defined vs. undefined CAC as appropriate.
- [ ] Task 6 explicitly compares two specific numbers (not just "it depends") and reaches a stated recommendation.

## Stretch

- Add a fifth column: **touches per conversion** (total touches on that channel ÷ conversions under first-touch). Which channel needs the most touches to produce one buyer?
- Recompute Task 2's CAC using only spend from **days 1–5** (Lecture 3 §2's saturation window) instead of all 10 days for Meta Ads. How much does its effective CAC improve if you'd stopped spending on day 5?
- Write a query that flags any paid channel where `first_touch_cac > 2 * last_touch_cac` or vice versa — a >2x swing between models as an automatic "investigate the full path before trusting this number" flag.

## Submission

Commit `solutions.sql` to your portfolio under `c38-week-02/exercise-03/`.
