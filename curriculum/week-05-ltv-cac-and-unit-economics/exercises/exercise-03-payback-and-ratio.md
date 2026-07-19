# Exercise 3 — Compute Payback and LTV:CAC

**Goal:** Compute naive payback, retention-adjusted payback, and the LTV:CAC ratio in pandas, using the LTV numbers from Exercise 1 and the CAC numbers from Exercise 2 — and see, with your own code, exactly where the two payback methods disagree.

**Estimated time:** 45 minutes.

## Setup

You'll need these numbers from your own Exercises 1 and 2 (or the lecture values, if yours differ slightly due to rounding):

```python
margin = {'paid_search': 119.20, 'organic_content': 79.20}
ltv_projected = {'paid_search': 1342.73, 'organic_content': 2061.64}

cac = {
    'paid_search_blended':      2000.00,
    'organic_blended':          2571.43,
    'organic_q4':                1636.36,
}

# cumulative contribution margin per customer by age, from Exercise 1's cohort-sum work
cum_margin = {
    'paid_search':     {0: 119.20, 1: 227.56, 2: 311.00, 3: 381.64, 4: 446.21, 5: 508.65, 6: 574.87},
    'organic_content': {0: 79.20,  1: 148.50, 2: 219.78, 3: 289.66, 4: 351.89, 5: 409.49, 6: 471.09},
}
```

Create `analysis.py` (or a notebook). Label each answer `# Task N`.

## Tasks

1. **Naive payback.** `naive_payback = cac ÷ margin_per_month`, for all three CAC scenarios. *(Expected: `paid_search` → 16.78 months; `organic_content` blended → 32.47 months; `organic_content` Q4 → 20.66 months.)*

2. **Percent of CAC recovered by month 6.** For each channel, `cum_margin[6] ÷ cac`, as a percentage, for all three scenarios. *(Expected: `paid_search` → 28.7%; `organic_content` blended → 18.3%; `organic_content` Q4 → 28.8%.)*

3. **Does it ever cross?** Using each channel's `ltv_projected` (the reciprocal-formula LTV, which represents the *entire* projected lifetime, not just 6 months) compared against each CAC scenario, determine — with a one-line boolean expression, not a guess — whether cumulative contribution margin ever fully recovers that CAC. *(Expected: `paid_search` vs. blended CAC → `False`; `organic_content` vs. blended CAC → `False`; `organic_content` vs. Q4 CAC → `True`.)*

4. **LTV:CAC ratio.** `ltv_projected ÷ cac`, for all three scenarios, rounded to 2 decimals. *(Expected: 0.67 / 0.80 / 1.26.)*

5. **Health classification.** Write a function `classify(ratio: float) -> str` that returns `"healthy"` for ratio ≥ 3, `"marginal"` for 1 ≤ ratio < 3, and `"unprofitable"` for ratio < 1. Apply it to all three Task 4 results. *(Expected: `paid_search` → "unprofitable"; `organic_content` blended → "unprofitable"; `organic_content` Q4 → "marginal".)*

6. **The reconciliation.** In a short comment or docstring (3–4 sentences), explain why Task 1's naive payback for `paid_search` (16.78 months, which sounds survivable) and Task 3's answer for the same channel (payback never fully happens) are not a contradiction — they're answering different questions. Name the assumption naive payback makes that retention-adjusted payback doesn't.

## Expected result (spot checks)

- Task 1 → 16.78 / 32.47 / 20.66 months.
- Task 3 → False / False / True.
- Task 4 → 0.67 / 0.80 / 1.26.
- Task 5 → unprofitable / unprofitable / marginal.

## Done when…

- [ ] `analysis.py` runs top to bottom and prints all 5 numeric tasks matching the expected values.
- [ ] Task 6's explanation names the specific assumption (constant 100% retention, no churn ever) that makes naive payback optimistic.
- [ ] You can say, without checking your notes, which of the three scenarios is the only one classified above "unprofitable" — and why that one specifically.

## Stretch

- `organic_content`'s Q4 CAC gets it to "marginal" (1.26), not "healthy" (≥3). What would `organic_content`'s monthly churn rate need to drop to — holding CAC and MRR fixed — to reach a 3:1 ratio? *(Hint: solve `margin ÷ new_churn = 3 × cac` for `new_churn`.)*
- Build a small pandas `DataFrame` with one row per (channel, CAC-scenario) combination and all five metrics as columns. This is effectively the table the mini-project asks you to produce for both channels — get comfortable building it now.

## Submission

Commit `analysis.py` to your portfolio under `c38-week-05/exercise-03/`.
