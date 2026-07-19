# Exercise 2 — Analyze an A/B Result

**Goal:** run the complete read of a finished test against the `checkout_sessions` pilot — primary metric significance, a confidence interval, and both guardrail metrics — exactly the sequence a real launch review needs, and exactly what Exercise 3's SRM check should always precede in practice (you'll do that check next).

**Estimated time:** 75 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM checkout_sessions;   -- must print 100
```

Create `queries.sql` for the SQL tasks and `analysis.py` for the Python tasks. Label each with a `-- Task N` / `# Task N` comment.

## Tasks

### SQL (Tasks 1–3)

1. **Primary metric counts.** Write the `GROUP BY variant` query from Lecture 2, section 4 yourself — don't copy it verbatim — returning `variant, sessions, conversions, rate`. *(Expected: control 50/21/0.420, treatment 50/32/0.640.)*

2. **Guardrail 1: AOV.** Return `variant, aov` (average `order_value_usd`, converted sessions only, rounded to 2 decimals). *(Expected: control 78.00, treatment 70.31.)*

3. **Guardrail 2: refund rate.** Return `variant, conversions, refunds, refund_rate` (refunds as a fraction of conversions, rounded to 3 decimals — remember `refunded` is `NULL` for non-converted rows, so filter to `WHERE converted` first). *(Expected: control 21/2/0.095, treatment 32/4/0.125.)*

### Python (Tasks 4–7)

Bring your Task 1–3 results into Task 4 onward as plain variables (typed in by hand from your SQL output is fine at this size).

4. **Primary metric significance.** Compute the pooled two-proportion z-test from Lecture 2, section 4, on the Task 1 counts. Report `z` and the two-tailed `p_value`. *(Expected: z ≈ 2.204, p ≈ 0.0275.)*

5. **Primary metric confidence interval.** Compute the **unpooled** 95% CI for the difference in conversion rate, from Lecture 2, section 5. Report the interval in percentage points. *(Expected: +22.0 points, 95% CI ≈ [+2.9, +41.1] points.)*

6. **Guardrail significance — does the refund-rate increase hold up?** The raw refund rate rose from 9.5% to 12.5% (Task 3). Run the *same* z-test machinery from Task 4 on the refund counts (2 of 21 vs. 4 of 32 — note the denominators here are **conversions**, not all sessions, since refunds only apply to completed purchases). Report `z`, `p_value`, and the 95% CI for the difference. *(Expected: z ≈ 0.334, p ≈ 0.738, 95% CI roughly [−14.0, +20.0] points — crosses zero by a wide margin.)*

7. **The revenue reconciliation.** AOV fell (Task 2), but does *revenue per cart session* (using `COALESCE(order_value_usd, 0)` over **all** sessions, not just converted ones) still favor treatment? Compute it for both arms. *(Expected: control $32.76/session, treatment $45.00/session.)*

## Expected result (spot checks)

- Task 1 → control 42.0%, treatment 64.0%.
- Task 4 → z ≈ 2.204, p ≈ 0.0275 (significant at α=0.05).
- Task 5 → 95% CI ≈ [+2.9pp, +41.1pp].
- Task 6 → p ≈ 0.738 (**not** significant — the refund-rate guardrail signal is noise at this sample size, not a confirmed regression).
- Task 7 → treatment's revenue/session is ≈37% higher than control's, despite the lower AOV.

## Done when…

- [ ] `queries.sql` has all 3 SQL tasks, returning the expected values.
- [ ] `analysis.py` has all 4 Python tasks with printed output matching the expected values within rounding.
- [ ] You can explain, out loud, why a "significant" primary metric (Task 4) and a "not significant" guardrail regression (Task 6) can both be true at once, and why that's actually the *good* outcome here, not a contradiction.
- [ ] You can state, in one sentence each, what the Task 5 CI and the Task 6 CI each tell you that the corresponding p-value alone doesn't.

## Stretch

- Recompute Task 6 assuming the pilot had run **4 weeks** instead of 1 (multiply both arms' conversion and refund counts by 4, keeping the same rates). Does the refund-rate difference become significant at the larger sample size, even though the *rates* didn't change at all? What does that tell you about the relationship between sample size and your ability to detect a guardrail regression, specifically?
- Write one paragraph, as if for a launch-review doc, giving your overall ship/no-ship recommendation for Express Checkout based on everything in this exercise — primary metric, both guardrails, and the CI widths. Don't just report numbers; make the call and defend it.

## Submission

Commit `queries.sql` and `analysis.py` to your portfolio under `c38-week-08/exercise-02/`.
