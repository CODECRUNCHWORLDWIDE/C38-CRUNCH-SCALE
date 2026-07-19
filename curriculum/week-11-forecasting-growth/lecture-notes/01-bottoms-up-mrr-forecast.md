# Lecture 1 — Bottoms-Up MRR Forecasting

> **Duration:** ~2 hours. **Outcome:** You can take a trailing MRR bridge, compute the rates hiding inside it, and project it forward month by month in both SQL and pandas — with every assumption a named number you could defend in a meeting.

A forecast is a claim about the future stated precisely enough that reality can prove it wrong. "MRR will keep growing" is not a forecast — it's a mood. "Ending MRR will be $144,946 by December 2026, assuming new MRR grows $150/month from a $5,350 January base, expansion holds at 3.2% of starting MRR, contraction at 0.85%, and churn at 3.0%" is a forecast. Every number in that sentence is something you could be asked to defend, and *bottoms-up forecasting* is the discipline of building your projection so that every one of those numbers is visible, not buried inside a single opaque growth-rate assumption like "MRR grows 8% a month."

This lecture builds a full bottoms-up forecast for Crunch Flow, one line at a time, in SQL and then in pandas.

## 1. The bridge is already a forecasting engine

Recall the MRR bridge identity from Week 6 — it holds for every month, actual or forecast:

```
ending_mrr = starting_mrr + new_mrr + expansion_mrr − contraction_mrr − churned_mrr
```

and the chaining rule that makes a *sequence* of months work:

```
next_month.starting_mrr = this_month.ending_mrr
```

That's the entire mechanism. A twelve-month forecast is just this identity applied twelve times in a row, with `starting_mrr` for month *n* pulled from month *n−1*'s `ending_mrr`, and `new_mrr`, `expansion_mrr`, `contraction_mrr`, `churned_mrr` supplied by *you*, the forecaster, based on assumptions you can state out loud.

The entire forecasting problem reduces to: **where do those four numbers come from for a month that hasn't happened yet?** The rest of this lecture answers that.

## 2. Reading the rates out of your actuals

You don't guess `expansion_mrr`, `contraction_mrr`, and `churned_mrr` in dollars directly — dollar amounts don't generalize as the business grows. You express each as a **rate against the starting MRR that month**, because rates are stable(ish) even while the dollar base compounds.

```sql
SELECT
    month,
    starting_mrr,
    ROUND(100.0 * expansion_mrr   / starting_mrr, 2) AS expansion_rate_pct,
    ROUND(100.0 * contraction_mrr / starting_mrr, 2) AS contraction_rate_pct,
    ROUND(100.0 * churned_mrr     / starting_mrr, 2) AS churn_rate_pct,
    ROUND(100.0 * (starting_mrr + expansion_mrr - contraction_mrr - churned_mrr)
                 / starting_mrr, 2)                   AS net_revenue_retention_pct
FROM mrr_bridge_actuals
ORDER BY month;
```

Run this against the seed and look at December: `expansion_rate_pct ≈ 3.22`, `contraction_rate_pct ≈ 0.87`, `churn_rate_pct ≈ 2.95`, `net_revenue_retention_pct ≈ 99.40`. Read that last number carefully — **net revenue retention (NRR) just under 100% means that if Crunch Flow closed zero new deals next month, its existing base would shrink slightly.** That's not a red flag on its own (plenty of healthy, growing SaaS businesses run sub-100% NRR early on) but it *is* the single most important fact this forecast has to be honest about: **Crunch Flow's growth is sales-led, not expansion-led.** Every dollar of net growth is coming from `new_mrr`, not from the existing base compounding on its own. Hold that thought — it changes which lever matters most in Lecture 3's scenarios.

### Why the trailing average, and which trailing window

Look at the full twelve months of rates and you'll notice they wobble — August's `churn_rate_pct` (≈3.04%) is the year's worst, right alongside its `new_mrr` dip. A forecast built off a single volatile month is fragile; a forecast built off the full year is stale (it includes January, when the business looked different). The standard compromise is a **trailing-N-month average**, most often trailing 3 or trailing 6:

```sql
SELECT
    month,
    ROUND(100.0 * AVG(expansion_mrr   / starting_mrr) OVER w, 2) AS avg_expansion_rate_3mo,
    ROUND(100.0 * AVG(contraction_mrr / starting_mrr) OVER w, 2) AS avg_contraction_rate_3mo,
    ROUND(100.0 * AVG(churned_mrr     / starting_mrr) OVER w, 2) AS avg_churn_rate_3mo
FROM mrr_bridge_actuals
WINDOW w AS (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
ORDER BY month;
```

The trailing-3-month window ending December 2025 gives roughly: expansion ≈3.19%, contraction ≈0.85%, churn ≈2.98%. That's the base-case rate set this lecture uses going forward — recent enough to reflect the current business, smooth enough not to be whipsawed by any one month.

## 3. Projecting `new_mrr` forward

Rates handle expansion, contraction, and churn because they scale naturally with a growing base. `new_mrr` is different — it isn't a rate against anything; it's driven by sales/marketing capacity, and you have to project it on its own trend.

The simplest honest approach: fit a straight line through the trailing `new_mrr` history and read off the slope.

```sql
-- Simple linear trend via Postgres's built-in regression aggregates.
-- x = month index (1..12), y = new_mrr.
SELECT
    regr_slope(new_mrr, month_idx)     AS slope_per_month,
    regr_intercept(new_mrr, month_idx) AS intercept
FROM (
    SELECT new_mrr,
           ROW_NUMBER() OVER (ORDER BY month) AS month_idx
    FROM mrr_bridge_actuals
) t;
```

*(SQLite has no built-in `regr_slope`; do this step in pandas with `numpy.polyfit` instead — the pandas block below shows exactly that.)*

Running this against the twelve actuals gives a slope of roughly **+$140/month** and an intercept that lands January's fitted value near $3,020 (close to, but not exactly, the actual $3,200 — a straight line can't hit every point, including the August dip, and that's expected). For the base case this lecture rounds that to a clean **+$150/month**, starting from a January 2026 baseline of **$5,350** — a small, deliberate bump over December's $5,200 actual, reflecting the trend rather than freezing at the last data point.

**Naming the assumption out loud, the way you'll do in every memo this week:** *"New MRR is assumed to grow linearly by $150/month off a $5,350 January base, based on the trailing-twelve-month OLS trend. This assumes Crunch Flow's sales motion keeps adding new logos at roughly its current pace — it does **not** assume any new channel, price change, or headcount increase."* That sentence is the difference between a forecast and a guess.

## 4. Rolling the whole bridge forward in SQL

With rates and a `new_mrr` trend in hand, you can generate the forecast with a **recursive CTE** — each row's `starting_mrr` depends on the previous row's `ending_mrr`, which is exactly what recursion is for.

```sql
WITH RECURSIVE forecast AS (
    -- Anchor: January 2026, seeded from December 2025's actual ending_mrr.
    SELECT
        1                                            AS month_num,
        (SELECT ending_mrr FROM mrr_bridge_actuals
         WHERE month = '2025-12-01')                 AS starting_mrr,
        5350.0                                        AS new_mrr,
        (SELECT ending_mrr FROM mrr_bridge_actuals
         WHERE month = '2025-12-01') * 0.0319          AS expansion_mrr,
        (SELECT ending_mrr FROM mrr_bridge_actuals
         WHERE month = '2025-12-01') * 0.0085           AS contraction_mrr,
        (SELECT ending_mrr FROM mrr_bridge_actuals
         WHERE month = '2025-12-01') * 0.0298            AS churned_mrr

    UNION ALL

    -- Recursive step: chain starting_mrr, grow new_mrr by $150/mo, re-apply rates.
    SELECT
        f.month_num + 1,
        f.starting_mrr + f.new_mrr + f.expansion_mrr - f.contraction_mrr - f.churned_mrr,
        f.new_mrr + 150.0,
        (f.starting_mrr + f.new_mrr + f.expansion_mrr - f.contraction_mrr - f.churned_mrr) * 0.0319,
        (f.starting_mrr + f.new_mrr + f.expansion_mrr - f.contraction_mrr - f.churned_mrr) * 0.0085,
        (f.starting_mrr + f.new_mrr + f.expansion_mrr - f.contraction_mrr - f.churned_mrr) * 0.0298
    FROM forecast f
    WHERE f.month_num < 12
)
SELECT
    month_num,
    ROUND(starting_mrr)   AS starting_mrr,
    ROUND(new_mrr)         AS new_mrr,
    ROUND(expansion_mrr)    AS expansion_mrr,
    ROUND(contraction_mrr)   AS contraction_mrr,
    ROUND(churned_mrr)        AS churned_mrr,
    ROUND(starting_mrr + new_mrr + expansion_mrr - contraction_mrr - churned_mrr) AS ending_mrr
FROM forecast
ORDER BY month_num;
```

*(SQLite supports `WITH RECURSIVE` with the same syntax — this query runs unchanged on both engines.)*

Running the full twelve-month recursion gives quarter-ending values of roughly **Q1 ≈ $94,129 → Q2 ≈ $110,042 → Q3 ≈ $126,987 → Q4 ≈ $144,946**. Notice growth *accelerates* through the year even though the rates are flat — that's the base compounding: expansion and churn are both percentages of a starting MRR that's itself getting bigger every month, so their dollar amounts grow even while their rates don't.

## 5. The same forecast in pandas

A recursive CTE is elegant but SQL is an awkward place to *iterate* — pandas, with a plain Python loop, is often the more natural tool for the "build twelve rows, each depending on the last" pattern. This is the version you'll actually extend in the exercises and mini-project.

```python
import pandas as pd
import numpy as np

actuals = pd.read_sql("SELECT * FROM mrr_bridge_actuals ORDER BY month", conn)

# 1. Fit the new_mrr trend.
x = np.arange(1, 13)
slope, intercept = np.polyfit(x, actuals["new_mrr"], 1)
print(f"new_mrr trend: {slope:.2f}/month")   # ≈ 140/month

# 2. Trailing-3-month average rates (the base case).
recent = actuals.tail(3)
exp_rate    = (recent["expansion_mrr"]   / recent["starting_mrr"]).mean()
contr_rate  = (recent["contraction_mrr"] / recent["starting_mrr"]).mean()
churn_rate  = (recent["churned_mrr"]     / recent["starting_mrr"]).mean()

# 3. Roll the bridge forward, one row at a time.
rows = []
start = actuals.iloc[-1]["ending_mrr"]     # Dec 2025 ending_mrr = Jan 2026 starting_mrr
new   = 5350.0                             # rounded base-case January new_mrr

for m in range(1, 13):
    exp    = start * exp_rate
    contr  = start * contr_rate
    churn  = start * churn_rate
    end    = start + new + exp - contr - churn
    rows.append({
        "month_num": m, "starting_mrr": start, "new_mrr": new,
        "expansion_mrr": exp, "contraction_mrr": contr,
        "churned_mrr": churn, "ending_mrr": end,
    })
    start = end          # next month's starting_mrr
    new  += 150.0         # grow new_mrr by the fitted trend

forecast = pd.DataFrame(rows).round(0)
forecast["quarter"] = ((forecast["month_num"] - 1) // 3) + 1
print(forecast.groupby("quarter")["ending_mrr"].last())
```

The pandas version and the SQL recursive CTE will land on the same numbers (small rounding differences aside) because they encode the identical assumptions — that's the point of building it twice. If they *disagree*, one of them has a bug, and the mismatch is how you'd find it.

## 6. What bottoms-up forecasting is — and isn't — good for

**Strengths:**

- Every number traces to a named assumption. A CFO can pick any line and ask "why $150/month?" and you have an answer rooted in actual data, not "that felt right."
- It respects the actual mechanics of the business — MRR really is built from new, expansion, contraction, and churn, so a forecast shaped the same way stays honest about *where* growth or shrinkage comes from.
- It's easy to turn into scenarios (Lecture 3) — bump one rate, rerun.

**Weaknesses — hold onto these for later lectures:**

- It treats the whole customer base as one undifferentiated pool. It can't tell you that October's cohort behaves differently from January's — for that you need the cohort-level view in Lecture 2.
- A straight-line trend on `new_mrr` says nothing about *why* new_mrr grows — it has no model of market saturation, sales headcount, or seasonality. Challenge 1 forces you to confront that gap directly.
- A single point forecast (one number per month) implies false precision. Lecture 3 fixes that by attaching an honest range to it.

## Check yourself

1. Why do expansion, contraction, and churn get expressed as **rates against starting MRR** rather than fixed dollar amounts when projecting forward?
2. Crunch Flow's December 2025 net revenue retention is ≈99.4%. What does a *sub*-100% NRR tell you about where the company's growth is coming from?
3. In the recursive CTE, why must `next_month.starting_mrr` equal `this_month.ending_mrr` exactly — what breaks if it doesn't?
4. Name one thing a bottoms-up bridge forecast structurally cannot tell you that a cohort-level forecast can.

Next: [Lecture 2 — Retention-Adjusted Projection](./02-retention-adjusted-projection.md), where you stop treating the customer base as one pool and start forecasting cohort by cohort.
