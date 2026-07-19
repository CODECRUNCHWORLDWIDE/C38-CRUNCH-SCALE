# Lecture 2 — Retention-Adjusted Projection

> **Duration:** ~2 hours. **Outcome:** You can build a pooled revenue-retention-by-age curve from right-censored cohort data without letting older cohorts silently dominate it, extend that curve responsibly past the range you've actually observed, and use it to project what today's cohorts will still be worth a year from now.

Lecture 1's bridge forecast treats Crunch Flow's entire customer base as one undifferentiated pool: one expansion rate, one churn rate, applied uniformly to everyone. That's a reasonable approximation, but it hides something the cohort data makes visible — **not every customer behaves the same way at the same age.** A customer three months in is still discovering the product and more likely to churn; a customer eighteen months in has either become sticky or already left. Averaging all of that into one monthly rate smooths over the shape entirely.

This lecture builds the other kind of forecast: instead of one pool, you track each signup cohort separately, learn how its MRR decays (or grows) with age, and project that curve forward — which is exactly the retention-curve technique from Week 5's LTV lecture, now applied to *revenue* survival instead of customer-lifetime value.

## 1. What `cohort_revenue_retention` actually measures

Recall the seed table: for each cohort (the month a group of customers signed up) and each `months_since_signup`, you have `starting_cohort_mrr` (the cohort's MRR at age 0 — when they signed up) and `active_cohort_mrr` (that same cohort's MRR today, at this age). The ratio is the cohort's **revenue retention** at that age:

```sql
SELECT
    cohort_month,
    months_since_signup,
    starting_cohort_mrr,
    active_cohort_mrr,
    ROUND(100.0 * active_cohort_mrr / starting_cohort_mrr, 1) AS pct_revenue_retained
FROM cohort_revenue_retention
ORDER BY cohort_month, months_since_signup;
```

This is **net revenue retention at the cohort level** — it nets out churn (customers leaving), contraction (downgrades), *and* expansion (upgrades) within that one cohort. It can rise as well as fall. Look at the January 2025 cohort: it starts at 100% (age 0, by definition), *drops* to a low of 88.8% around age 5 (churn and downgrades dominate early), then *climbs back* to 98.2% by age 11 as the survivors upgrade and offset the losses. That dip-then-recover pattern is the **"smile" curve** from Week 4's retention lecture, now measured in dollars instead of logos — and it's the single most important shape to recognize in this data, because a forecast that only looks at the first few months of a cohort's life will see nothing but decline and miss the recovery entirely.

## 2. The right-censoring problem — and the wrong way to fix it

Here's the trap. You have six cohorts, but not the same number of data points for each: January has 12 (ages 0–11), June has only 7 (ages 0–6), because June simply hasn't had time to reach age 11 yet by the December 2025 cutoff. If you build a "typical curve by age" with a naive average:

```sql
-- WRONG (or at least misleading): a naive average by age.
SELECT
    months_since_signup,
    ROUND(AVG(100.0 * active_cohort_mrr / starting_cohort_mrr), 1) AS avg_pct_retained,
    COUNT(*) AS n_cohorts_observed
FROM cohort_revenue_retention
GROUP BY months_since_signup
ORDER BY months_since_signup;
```

Run it and look at `n_cohorts_observed`: age 0–6 has all 6 cohorts averaged in; age 7–8 only has 5 and 4; age 9–11 has just 2, 2, and **1** (January alone). The "average" at age 11 isn't an average at all — it's a single cohort's number wearing an average's clothes. Worse, because older cohorts are the *only* ones that reach the high ages, any real difference between cohorts (like June's post-onboarding improvement) gets buried the moment you blend cohorts together without tracking which one contributed what.

**The fix has two parts.**

**Part one — weight by revenue, not by cohort count**, so a $4,000 cohort counts four times as much as a $1,000 one would (not relevant with our roughly-similar cohort sizes, but it matters the moment cohorts vary a lot in size — always weight):

```sql
SELECT
    months_since_signup,
    ROUND(100.0 * SUM(active_cohort_mrr) / SUM(starting_cohort_mrr), 1) AS weighted_pct_retained,
    COUNT(*) AS n_cohorts_observed
FROM cohort_revenue_retention
GROUP BY months_since_signup
ORDER BY months_since_signup;
```

**Part two — segment by curve, not just by age.** The June cohort isn't just "one more data point at age 6" — it's evidence of a *different* underlying curve, because Crunch Flow changed its onboarding flow that month. Blending it into the same weighted average as the pre-onboarding cohorts (Jan–May) would understate how much retention actually improved. The right move is to keep two curves:

```sql
SELECT
    'legacy_onboarding' AS curve,
    months_since_signup,
    ROUND(100.0 * SUM(active_cohort_mrr) / SUM(starting_cohort_mrr), 1) AS pct_retained
FROM cohort_revenue_retention
WHERE cohort_month < '2025-06-01'
GROUP BY months_since_signup

UNION ALL

SELECT
    'new_onboarding' AS curve,
    months_since_signup,
    ROUND(100.0 * SUM(active_cohort_mrr) / SUM(starting_cohort_mrr), 1) AS pct_retained
FROM cohort_revenue_retention
WHERE cohort_month >= '2025-06-01'
GROUP BY months_since_signup
ORDER BY curve, months_since_signup;
```

Compare the two curves at the ages they overlap (0–6). At age 3, legacy holds 90.5% (this is the pooled Jan–May average, close to January's own 90.5%); new-onboarding (June alone) holds 93.5%. At age 6, legacy is ≈89.7%; new-onboarding is 94.0%. **That's a real, three-to-four-point improvement in revenue retention that shows up within the first half-year** — exactly the kind of signal a single blended curve would have diluted. This matters for forecasting: any cohort that signs up from June 2025 onward — including every cohort you haven't observed yet in 2026 — should be projected on the *new* curve, not the legacy one.

## 3. Extending a curve past the range you've observed

The legacy curve has data out to age 11 (thanks to the January cohort). The new-onboarding curve only has data out to age 6 (June is the only cohort on it, and it's only 6 months old at the cutoff). To project *any* cohort a full year or more forward, you need retention percentages at ages you haven't observed yet — and this is where forecasters get sloppy.

**The tempting-but-wrong move:** fit a line or curve to the observed points and extrapolate the trend indefinitely. The new-onboarding curve is still *rising* at age 6 (94.0%, up from 93.0% at age 4) — extrapolate that trend for another twelve months and you'd predict retention climbing past 100%, which is not impossible for a single cohort with heavy expansion, but is not something you should *assume* without evidence.

**The defensible move: flat continuation.** Once a curve has entered the "recovery" phase of the smile and stabilized (legacy: ages 9–11 sit in a tight 93.8–98.2% band; new-onboarding: ages 4–6 sit in a tight 93.0–94.0% band), hold it flat at its **last observed value** for every age beyond that. You're explicitly choosing not to bet on a trend you haven't confirmed yet — which is the conservative, honest choice for a number that's going into a forecast someone else will rely on.

```sql
-- Flat-continuation lookup: last observed value per curve, used for any
-- age beyond what's been directly measured.
SELECT
    'legacy_onboarding' AS curve, MAX(months_since_signup) AS max_observed_age,
    (SELECT ROUND(100.0*SUM(active_cohort_mrr)/SUM(starting_cohort_mrr),1)
     FROM cohort_revenue_retention
     WHERE cohort_month < '2025-06-01' AND months_since_signup = 11) AS flat_value_pct
FROM cohort_revenue_retention WHERE cohort_month < '2025-06-01'

UNION ALL

SELECT
    'new_onboarding', MAX(months_since_signup),
    (SELECT ROUND(100.0*SUM(active_cohort_mrr)/SUM(starting_cohort_mrr),1)
     FROM cohort_revenue_retention
     WHERE cohort_month >= '2025-06-01' AND months_since_signup = 6)
FROM cohort_revenue_retention WHERE cohort_month >= '2025-06-01';
```

That gives you: legacy flat value ≈98.2% (held for any age beyond 11), new-onboarding flat value ≈94.0% (held for any age beyond 6).

## 4. Projecting existing cohorts forward to a target date

Now put it together. Suppose Finance wants to know: **how much of December 2026's MRR is already "baked in" from customers who signed up in 2025** (as opposed to customers Crunch Flow hasn't sold to yet)? Each 2025 cohort's age by December 2026 is fixed by the calendar — the January cohort turns 23 months old, the December cohort turns 12.

```python
import pandas as pd

# Each 2025 cohort's starting MRR (age 0) — pulled from mrr_bridge_actuals.new_mrr,
# since a cohort's starting MRR *is* the new MRR added that signup month.
cohorts_2025 = pd.read_sql("""
    SELECT month AS cohort_month, new_mrr AS starting_cohort_mrr
    FROM mrr_bridge_actuals ORDER BY month
""", conn)
cohorts_2025["cohort_month"] = pd.to_datetime(cohorts_2025["cohort_month"])

target_date = pd.Timestamp("2026-12-01")
cohorts_2025["age_at_target"] = (
    (target_date.year - cohorts_2025["cohort_month"].dt.year) * 12
    + (target_date.month - cohorts_2025["cohort_month"].dt.month)
)

LEGACY_MAX_AGE, LEGACY_FLAT = 11, 0.982
NEW_MAX_AGE,    NEW_FLAT    = 6,  0.940

def retained_pct(row):
    is_new_curve = row["cohort_month"] >= pd.Timestamp("2025-06-01")
    max_age, flat = (NEW_MAX_AGE, NEW_FLAT) if is_new_curve else (LEGACY_MAX_AGE, LEGACY_FLAT)
    # (In the exercise you'll look up the exact observed value when age <= max_age;
    # here every 2025 cohort's target age already exceeds its curve's observed range,
    # so every row uses flat continuation.)
    return flat if row["age_at_target"] > max_age else flat

cohorts_2025["retained_pct"] = cohorts_2025.apply(retained_pct, axis=1)
cohorts_2025["projected_mrr_at_target"] = (
    cohorts_2025["starting_cohort_mrr"] * cohorts_2025["retained_pct"]
)
print(cohorts_2025[["cohort_month","age_at_target","retained_pct","projected_mrr_at_target"]])
print("Total 2025-cohort MRR still active, Dec 2026:",
      round(cohorts_2025["projected_mrr_at_target"].sum()))
```

Running this: the six 2025 cohorts (Jan–Jun, legacy curve) contribute roughly **$20,552** to December 2026's MRR; the six cohorts Crunch Flow will sign in the second half of 2025 (Jul–Dec, all on the new-onboarding curve, using their `new_mrr` from the bridge table) contribute roughly **$22,936**. **Total: ≈$43,488** of December 2026's MRR is already "baked in" from customers who exist today, before Crunch Flow closes a single new deal in 2026.

## 5. Reading this against the bridge forecast

Lecture 1's bottoms-up bridge projected December 2026 ending MRR at roughly **$144,946**. This cohort-level view says only **≈$43,488** of that comes from customers who already exist. The gap — over $100,000 — has to come from customers Crunch Flow *hasn't sold yet*: every 2026 cohort's `new_mrr`, plus whatever expansion happens on top of the 2025 base beyond pure survival.

**These two methods will never tie out exactly, and that's fine — they're not supposed to.** The bridge forecast models the whole business as one pool with one set of rates; the cohort forecast tracks individual cohorts on individual curves. When they're in the same ballpark, that's a genuine cross-check that increases your confidence. When they diverge wildly, that's the signal to go find out why — maybe the bridge's blended churn rate is masking a cohort that's about to fall off a cliff, or maybe a cohort curve's flat-continuation assumption is too conservative. Either way, you found something worth investigating *before* the forecast went to the board, which is the entire point of building it two ways.

## Check yourself

1. Why does a naive `AVG()` by age silently misrepresent the retention curve once cohorts have unequal numbers of observed ages?
2. What's the difference between weighting a pooled curve by cohort **count** versus by cohort **starting MRR** — and why does the second one matter more as cohorts vary in size?
3. Why is "hold the curve flat at its last observed value" a more defensible default than "extrapolate the observed trend" when projecting past the observed range?
4. The bridge forecast and the cohort forecast disagree by over $100,000 at the twelve-month mark. Is that a bug? What would make you worried it *was* a bug?

Next: [Lecture 3 — Scenarios and Uncertainty](./03-scenarios-and-uncertainty.md), where you fit a time-series model, put an honest range around a point forecast, and build the base/bull/bear scenarios Finance actually asked for.
