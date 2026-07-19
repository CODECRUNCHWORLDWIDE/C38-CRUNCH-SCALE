# Exercise 3 — Model a Price-Sensitivity Curve

**Goal:** Fit a linear demand curve to `price_experiment` in SQL, find the revenue-maximizing price with calculus, and compute both arc and point elasticity — exactly as Lecture 3 did, but this time you write every line yourself.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM price_experiment;   -- must print 5
SELECT SUM(quotes_shown) FROM price_experiment;  -- must print 2000
```

Create `solutions.sql` and `analysis.py`.

## Tasks

### SQL (Tasks 1–2)

1. **Conversion rates.** Compute `price, conversions, quotes_shown, conversion_rate_pct` for all five rows, ordered by price. *(Expected: $29 → 22.00%; $39 → 15.50%; $55 → 7.00%.)*

2. **The regression.** On PostgreSQL, use `REGR_INTERCEPT`, `REGR_SLOPE`, and `REGR_R2` (all with `conversions` as the dependent variable and `price` as the independent variable — mind the argument order, it's `(y, x)` not `(x, y)`) to fit the line in one query. On SQLite, skip to the pandas step and use `numpy.polyfit` instead — note in your file which engine you used. *(Expected: intercept ≈ 152.96, slope ≈ −2.30, R² ≈ 0.996.)*

### pandas (Tasks 3–6)

3. **Rebuild the fit.** Load the five rows into a `DataFrame` and refit with `numpy.polyfit(prices, conversions, 1)`. Confirm your slope/intercept match Task 2 within rounding.

4. **Revenue-maximizing price.** Using `p* = −a / (2b)` (derived in Lecture 1 §1.2 — no, Lecture **3** §1.2, re-derive it: write out `R(p) = a·p + b·p²`, take `dR/dp`, set it to zero, solve for `p`), compute `p*`, the predicted conversions at `p*`, and the predicted revenue at `p*`. *(Expected: p\* ≈ $33.26, conversions ≈ 76.5 / 400 (19.1%), revenue ≈ $2,543.69 per 400-quote cohort.)*

5. **Arc elasticity, all four gaps.** Compute the midpoint-method arc elasticity between every pair of adjacent tested prices ($29→$35, $35→$39, $39→$45, $45→$55). *(Expected: −1.07, −1.38, −1.78, −2.63 — elasticity should get more negative (more elastic) as price rises.)*

6. **Point elasticity at three prices.** Using the fitted line and `E(p) = b · (p / Q(p))`, compute point elasticity at $29, $39, and $55. *(Expected: ≈ −0.77 at $29, ≈ −1.42 at $39, ≈ −4.78 at $55 — demand is actually *inelastic* at the very bottom of the tested range, and increasingly elastic above it.)*

## Expected result (spot checks)

- Task 1 → $39 → 15.50% conversion rate.
- Task 2/3 → `Q(p) = 152.96 − 2.30·p`, R² ≈ 0.996.
- Task 4 → p\* ≈ $33.26, revenue ≈ $2,543.69.
- Task 5 → arc elasticities: −1.07, −1.38, −1.78, −2.63.
- Task 6 → point elasticity at $39 ≈ −1.42.

## Done when…

- [ ] `solutions.sql` has Tasks 1–2, with the regression output (or a note that you moved it to pandas for SQLite).
- [ ] `analysis.py` has Tasks 3–6, with printed values matching the expected numbers within rounding.
- [ ] You can explain, out loud, why the revenue-maximizing price ($33.26) is *below* ScopeIQ's actual signup price ($39) — what does that imply is currently happening on the signup page?
- [ ] You can state, without looking it up, whether demand is elastic or inelastic across the entire tested price range.

## Stretch

- At what tested price is demand **closest** to unit-elastic (`|E| = 1`)? Is it above or below the revenue-maximizing price you found in Task 4 — and does that ordering make sense given how elasticity and revenue-maximization relate (hint: revenue is maximized exactly where `|E| = 1`, using the *exact* elasticity at that point rather than the coarser two-point arc estimate)?
- `price_experiment` never tested a price above $55. Extrapolate the fitted line to $70 and to $80. At what price does the model predict conversions hit zero? Is it responsible to trust the model's prediction for a $79 `Growth`-tier price, given that this experiment was run on the flat single-plan signup page, not the current three-tier pricing page? Write two sentences on the risk of extrapolating a fitted line beyond its observed range.

## Submission

Commit `solutions.sql` and `analysis.py` to your portfolio under `c38-week-09/exercise-03/`.
