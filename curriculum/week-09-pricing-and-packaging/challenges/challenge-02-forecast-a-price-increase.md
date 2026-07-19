# Challenge 2 — Forecast a Price Increase

**Estimated time:** 60 minutes.

## The situation

The `Growth` price increase from Lecture 3 ($69→$79) is approved and scheduled. Finance now asks the obvious follow-up: **"if that worked, why not raise `Starter` too?"** Your job is to run the same forecast methodology on a different tier, choose your own existing-customer elasticity assumption (Lecture 3 gave you `Growth`'s; `Starter` is a different customer base with different switching costs, and you have to reason about why its number should differ), and tell Finance whether it's a good idea.

**The proposal:** raise `Starter` from **$29 → $35** (a +20.7% increase), for new signups immediately and for existing `Starter` customers after the same 60-day grandfather window used for `Growth`.

## Tasks

### 1. Pull the baseline (SQL)

```sql
SELECT * FROM subscription_base WHERE tier = 'Starter';
```

Confirm: 1,150 existing customers, $29 current price, 85 new signups/month baseline.

### 2. Choose the new-signup elasticity

Unlike Lecture 3's `Growth` forecast, you don't need to extrapolate here — `price_experiment` tested **exactly** this price interval. Find the arc elasticity for the $29→$35 gap from Exercise 3 (or recompute it) and use it directly. *(Expected: −1.07.)* Compute the predicted new-signup rate at $35.

### 3. Choose and justify the existing-customer churn elasticity

This is the modeling decision Finance is really asking you to make. Lecture 3 assumed **−0.35** for `Growth`, reasoning that existing paying customers have switching costs a first-time prospect doesn't. `Starter` customers are `indie`-segment, solo builders — do they have *more* or *less* switching cost than a `team` account with multiple people's workflows built around the product? Pick a number, and justify it in 2–3 sentences using that reasoning (there's a reasonable range — anything between about −0.45 and −0.65 is defensible if you argue it; this challenge uses **−0.55** in its expected answers, but grade your own work on the *argument*, not on hitting that exact digit).

### 4. Run the 90-day forecast

Reuse Lecture 3 §6's model exactly — same structure, new numbers:

- Predicted incremental churn count and retained existing customers.
- New signups over 3 months at the new price.
- Total MRR and customer count at day 90, **with** vs. **without** the change.
- Net MRR impact ($ and %) and net customer impact (count and %).

*(Expected, using −0.55 for existing-customer elasticity: ~131 customers churn, 1,019 retained; ~66 new signups/month at $35; without-change day-90 MRR ≈ $40,745 (1,405 customers); with-change day-90 MRR ≈ $42,595 (1,217 customers); net impact ≈ **+$1,850 (+4.5%)**, customers **−188 (−13.4%)**.)*

### 5. Compare the two tiers' trades

Put the `Growth` result from Lecture 3 (+5.15% MRR, −8.16% customers) next to your `Starter` result side by side. Which tier gives a better MRR-uplift-per-customer-lost trade? Write two or three sentences on **why** — connect it back to Lecture 1's WTP ranges and Lecture 3 §3's discussion of switching costs, don't just restate the two percentages.

## Deliverable

`price-increase-forecast.md` with your Task 3 elasticity justification, the full Task 4 forecast (numbers + the code or query that produced them), and the Task 5 comparison with a final recommendation to Finance: ship the `Starter` increase, hold it, or redesign it (e.g., a smaller increase, or a longer grandfather window) — and why.

## Done when…

- [ ] Your churn-elasticity choice is justified by a specific comparison to `Growth`'s assumption, not asserted on its own.
- [ ] Your day-90 forecast numbers are internally consistent — retained customers + new signups should equal your reported total customer count, both with and without the change.
- [ ] Your Task 5 comparison names a concrete reason (WTP range, switching cost, or both) for why the two tiers' trades differ, not just that they do.

## Stretch

- Model a **discount mitigation** layer like Lecture 3 §4's annual-prepay offer for `Growth`: suppose `Starter` customers can lock in $27/month (below even their *old* $29 price) by prepaying annually ($324/yr), and 20% of the customers who would otherwise churn take that offer instead. Recompute Task 4's numbers with this layer added. Does it change your Task 5 recommendation?
- `subscription_base.monthly_signups` for `Starter` (85/month) is more than double `Growth`'s (40/month). Does a tier with a higher new-signup rate make a price increase **more** or **less** attractive, holding everything else constant? Explain using the two MRR-line structure from Lecture 3 §6 (existing-base line vs. new-signup line).

## Submission

Commit `price-increase-forecast.md` to your portfolio under `c38-week-09/challenge-02/`.
