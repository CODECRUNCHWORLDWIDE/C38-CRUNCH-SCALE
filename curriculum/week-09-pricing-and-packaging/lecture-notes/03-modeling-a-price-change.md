# Lecture 3 — Modeling a Price Change

> **Duration:** ~2 hours. **Outcome:** You can fit a demand curve to real conversion data, find the revenue-maximizing price with calculus, compute price elasticity at any point, and forecast the net MRR impact of a price increase — combining new-signup elasticity, existing-customer churn, discounts, grandfathering, and expansion revenue into one model.

You now have a defensible price ($69 for `Growth`) and a defensible reason for it (Lecture 1's WTP ranges, Lecture 2's usage gaps). The question this lecture answers is different: **what happens to revenue if you push that price further?** Not "is $79 acceptable" — you can check that against Lecture 1's curves in five minutes — but "if we raise `Growth` from $69 to $79, what does next quarter's MRR actually do, accounting for the customers who leave, the new customers who don't sign up, and the customers who quietly grow into a higher tier regardless?" That's a forecast, not a guess, and it's built the same way every forecast in this course has been built: from data, in SQL and pandas.

## 1. A demand curve from a real experiment

`price_experiment` holds six months of ScopeIQ's actual signup-page A/B test: 400 prospects were shown each of five prices at random, and you know how many converted:

```sql
SELECT
    price,
    quotes_shown,
    conversions,
    ROUND(100.0 * conversions / quotes_shown, 2) AS conversion_rate_pct
FROM price_experiment
ORDER BY price;
```

```
 price | quotes_shown | conversions | conversion_rate_pct
-------+--------------+-------------+----------------------
    29 |          400 |          88 |                22.00
    35 |          400 |          72 |                18.00
    39 |          400 |          62 |                15.50
    45 |          400 |          48 |                12.00
    55 |          400 |          28 |                 7.00
```

Conversions fall as price rises — unsurprising — but the *shape* of that fall is exactly what you need for two decisions: where's the revenue-maximizing price, and how sensitive is demand at each point.

### 1.1 Fit a linear demand curve

Model conversions as a straight line in price: `Q(p) = a + b·p`, where `Q` is expected conversions (out of 400 quotes) at price `p`. Fit it with ordinary least squares. PostgreSQL has this built in as a regression aggregate — no need to leave SQL:

```sql
SELECT
    REGR_INTERCEPT(conversions, price) AS a,
    REGR_SLOPE(conversions, price)     AS b,
    REGR_R2(conversions, price)        AS r_squared
FROM price_experiment;
```

```
     a      |     b     | r_squared
------------+-----------+-----------
 152.9636   | -2.2996   |  0.9956
```

*(SQLite has no built-in regression aggregates — pull the five rows into pandas and use `numpy.polyfit(prices, conversions, 1)`, which returns the same `[b, a]` pair.)*

An R² of 0.9956 means the straight line explains 99.56% of the variance in conversions across the five tested prices — about as clean as real experiment data gets. The fitted model:

```
Q(p) = 152.9636 − 2.2996 · p
```

Read the slope directly: **every $1 increase in price costs about 2.3 conversions out of 400 quotes shown.**

### 1.2 Find the revenue-maximizing price

Revenue is price times quantity: `R(p) = p · Q(p) = p·(a + b·p) = a·p + b·p²`. That's a downward-opening parabola in `p` (since `b` is negative), and it has one maximum — find it the way you'd find any parabola's vertex, with calculus: set the derivative to zero.

```
R(p) = a·p + b·p²
dR/dp = a + 2b·p
0 = a + 2b·p*
p* = −a / (2b)
```

```python
import numpy as np

a, b = 152.9636, -2.2996
p_star = -a / (2 * b)
q_star = a + b * p_star
r_star = p_star * q_star

print(f"Revenue-maximizing price: ${p_star:.2f}")
print(f"Predicted conversions at p*: {q_star:.1f} / 400 ({100*q_star/400:.1f}%)")
print(f"Predicted revenue per 400-quote cohort: ${r_star:,.2f}")
```

```
Revenue-maximizing price: $33.26
Predicted conversions at p*: 76.5 / 400 (19.1%)
Predicted revenue per 400-quote cohort: $2,543.69
```

Compare that to revenue at the five *actual* tested prices (`R(p) = p × Q(p)` at each):

| price | predicted conversions | predicted revenue |
|------:|----------------------:|-------------------:|
| $29 | 86.3 | $2,501.98 |
| $35 | 72.5 | $2,536.72 |
| **$33.26 (p\*)** | **76.5** | **$2,543.69** |
| $39 | 63.3 | $2,467.89 |
| $45 | 49.5 | $2,226.68 |
| $55 | 26.5 | $1,456.72 |

Two things jump out. First, the revenue-maximizing price ($33.26) is **below** ScopeIQ's actual price ($39) — for *pure signup-page revenue*, ScopeIQ is already priced too high, losing about $76 per 400-quote cohort to a price that converts too few people. Second — and this is the trap to notice before Lecture 3 §2 explains it — **this curve says nothing about whether to raise the price on existing `Growth` customers.** It was built entirely from prospects seeing a price for the very first time. Confusing "the revenue-maximizing price to show a stranger" with "the revenue-maximizing price to charge someone who already depends on your product every day" is one of the most common, most expensive pricing mistakes a growth team makes.

## 2. Elasticity — how sensitive is demand, precisely?

**Price elasticity of demand** is the percentage change in quantity demanded per percentage change in price — a normalized version of the slope that lets you compare sensitivity across products, price ranges, and (crucially, in a moment) customer types, regardless of the units involved.

### 2.1 Arc elasticity — between two observed points

For any two adjacent tested prices, the **arc elasticity** uses the midpoint method (so it doesn't matter which direction you compute it):

```
E = [ (Q2 − Q1) / ((Q1+Q2)/2) ] / [ (P2 − P1) / ((P1+P2)/2) ]
```

```python
import pandas as pd

exp = pd.DataFrame({
    "price":       [29, 35, 39, 45, 55],
    "conversions": [88, 72, 62, 48, 28],
})

for i in range(len(exp) - 1):
    p1, p2 = exp.price[i], exp.price[i+1]
    q1, q2 = exp.conversions[i], exp.conversions[i+1]
    pct_dq = (q2 - q1) / ((q1 + q2) / 2)
    pct_dp = (p2 - p1) / ((p1 + p2) / 2)
    print(f"${p1} → ${p2}: elasticity = {pct_dq/pct_dp:.2f}")
```

```
$29 → $35: elasticity = -1.07
$35 → $39: elasticity = -1.38
$39 → $45: elasticity = -1.78
$45 → $55: elasticity = -2.63
```

Read `|E| > 1` as **elastic** — demand falls *faster*, proportionally, than price rises, so raising price in that range *loses* revenue. `|E| < 1` is **inelastic** — demand barely reacts, so raising price *gains* revenue. Every segment of ScopeIQ's tested range is elastic (`|E|` always above 1), and elasticity gets *more* elastic — demand more sensitive — the higher the price climbs. That's the mechanism behind Section 1.2's result: once you're elastic, every further price increase loses more in volume than it gains in price, which is exactly what pushed the revenue-maximizing price below $39.

### 2.2 Point elasticity — at an exact price, using the fitted curve

Arc elasticity needs two observed points. **Point elasticity** uses calculus on the fitted line to get elasticity at any single price, including ones you never tested:

```
E(p) = (dQ/dp) · (p / Q(p)) = b · (p / (a + b·p))
```

```python
a, b = 152.9636, -2.2996

def point_elasticity(p):
    q = a + b * p
    return b * (p / q)

print(f"E($39) = {point_elasticity(39):.3f}")
print(f"E($45) = {point_elasticity(45):.3f}")
```

```
E($39) = -1.417
E($45) = -2.091
```

At ScopeIQ's current $39 signup price, a 1% price increase costs about 1.42% of conversions — clearly elastic, confirming the same story as the arc elasticities around it.

## 3. New-customer elasticity is not existing-customer elasticity

Here is the distinction that makes the rest of this lecture possible. `price_experiment` measured **first-touch conversion elasticity** — a stranger, seeing a price for the first time, deciding whether to become a customer at all. Raising the `Growth` tier from $69 to $79 for *existing, paying, already-dependent* customers is a completely different decision for them to make:

- They've already built workflows, dashboards, and habits around the product — **switching cost** is real and it wasn't a factor for a prospect who's never used you.
- They've already gotten value they can point to — **loss aversion** works in your favor now, where it worked against you on a cold signup page.
- A price increase on an existing customer usually comes with **advance notice** (a grandfathering period, §4 below) that a first-time prospect never gets — they have time to evaluate rather than bounce on reflex.

Empirically, across SaaS businesses, existing-customer churn elasticity to a price increase is commonly a **fraction** of new-visitor conversion elasticity — not because existing customers don't notice the price, but because the decision they're making ("is this still worth the hassle of switching") is a different, higher-friction decision than the one a prospect makes ("is this worth trying"). For this lecture, ScopeIQ assumes an existing-customer churn elasticity of **−0.35** — explicitly *lower* in magnitude than anything measured in `price_experiment` (−1.07 to −2.63), and explicitly **inelastic** (`|E| < 1`). This number is a stated assumption, not something derived from the data on hand — a real team would validate it with its own grandfathered-price-increase history, or, lacking that, a smaller pilot before rolling a change to the whole base. Flag every extrapolated assumption in a forecast exactly like this, in writing, so whoever reads it downstream knows which numbers are measured and which are estimated.

## 4. Discounts and grandfathering — softening the landing

Two levers make a price increase land without an unnecessary churn spike:

- **Grandfathering** — existing customers keep their old price for a defined window (ScopeIQ uses 60 days' notice) before the new price takes effect. This isn't just goodwill; it's what makes the lower existing-customer elasticity assumption in §3 *valid* — you measured or assumed that elasticity assuming customers get warning and time to react, not a silent bill change.
- **A retention discount** — an alternative offer at the moment of the increase, usually tied to a longer commitment. ScopeIQ's version: existing `Growth` customers can lock in **$759/year** (paid annually) instead of accepting $79/month (=$948/year) — an effective **$63.25/month**, actually *below* their old $69 monthly price, in exchange for an annual commitment instead of month-to-month. This is a deliberate below-old-price sweetener aimed specifically at the customers most likely to churn over the increase, trading a lower price for reduced future churn risk and better cash flow up front.

Both levers reduce the **realized** churn below whatever the raw elasticity assumption alone would predict — which is why a serious price-change forecast models them explicitly instead of assuming every customer either "accepts the new price" or "churns."

## 5. Expansion revenue — the line every naive forecast forgets

**Expansion revenue** is new MRR from *existing* customers, independent of the price change you're modeling — most commonly, a customer's own usage growing past their current tier's cap and them upgrading. Some `Starter` accounts' `mtu` keeps growing every month; some fraction of them cross 3,000 and move to `Growth` on their own, with no sales conversation about the price increase at all. A forecast that only tracks "customers who might churn from the increase" and ignores "customers who were going to expand anyway" understates revenue — and a forecast that credits *all* upgrade revenue to the price change overstates the change's actual effect. Keep the two lines separate:

- **Price-change effect** — the delta from raising `Growth`'s price itself: fewer new signups, some existing churn, higher revenue per retained/new customer.
- **Expansion effect** — the delta from customers moving tiers due to their own growth, which would happen at the *old* price too, just worth less per upgrade.

## 6. The full forecast — putting it together

`subscription_base` holds ScopeIQ's real installed base at the moment of the decision: 1,150 `Starter`, 640 `Growth`, 85 `Scale`. The proposal on the table: raise `Growth` from **$69 → $79** (+14.49%), for new signups immediately and for existing `Growth` customers after the 60-day grandfather window. Forecast the `Growth` tier's MRR 90 days out (one full grandfather window plus one signup cohort), **before expansion revenue and the discount offer — those layer on top in the mini-project.**

**New-signup impact.** `price_experiment` never tested $69 or $79 directly — this is an extrapolation beyond the observed $29–$55 range, so use the *closest-magnitude* arc elasticity as the best available stand-in: the $39→$45 jump (+15.4%) is the closest in percentage size to $69→$79 (+14.5%), with elasticity **−1.78**.

```python
baseline_signups_per_month = 40
pct_price_change = (79 - 69) / 69          # +0.1449
elasticity_new = -1.78

pct_signup_change = elasticity_new * pct_price_change
new_signups_per_month = baseline_signups_per_month * (1 + pct_signup_change)
print(f"Price change: {pct_price_change:.2%}")
print(f"Predicted signup change: {pct_signup_change:.2%}")
print(f"New signups/month at $79: {new_signups_per_month:.1f} (round to 30)")
```

```
Price change: 14.49%
Predicted signup change: -25.79%
New signups/month at $79: 29.7 (round to 30)
```

**Existing-base impact**, using the §3 assumption (elasticity −0.35, applied once as a one-time "shock" churn reaction to the increase, not compounding monthly):

```python
existing_growth_customers = 640
elasticity_existing = -0.35

pct_churn = abs(elasticity_existing * pct_price_change)
churned = round(existing_growth_customers * pct_churn)
retained = existing_growth_customers - churned
print(f"Predicted incremental churn: {pct_churn:.2%} → {churned} customers")
print(f"Retained: {retained} customers")
```

```
Predicted incremental churn: 5.07% → 32 customers
Retained: 608 customers
```

**90-day MRR comparison — with vs. without the price change:**

```python
months = 3

# Without the change (counterfactual): price stays $69
without_existing = 640 * 69
without_new = (40 * months) * 69
without_total = without_existing + without_new
without_customers = 640 + 40 * months

# With the change: $79, grandfathered, existing base takes the elasticity hit
with_existing = 608 * 79
with_new = (30 * months) * 79
with_total = with_existing + with_new
with_customers = 608 + 30 * months

print(f"Without change: ${without_total:,.0f} MRR, {without_customers} customers")
print(f"With change:    ${with_total:,.0f} MRR, {with_customers} customers")
print(f"Net impact:     ${with_total - without_total:+,.0f} ({(with_total-without_total)/without_total:+.2%}), "
      f"{with_customers - without_customers:+d} customers")
```

```
Without change: $52,440 MRR, 760 customers
With change:    $55,142 MRR, 698 customers
Net impact:     +$2,702 (+5.15%), -62 customers
```

**The recommendation writes itself from that last line:** the price increase is net-positive for revenue (+$2,702/mo, +5.15%) even though it *shrinks* the customer count (-62, -8.16%). That's the classic signature of real pricing power — when it's genuine, raising the price grows revenue on a smaller, more valuable base. If the net MRR number had come back negative, that would be equally decisive evidence *against* shipping the change, and just as valuable to know before the fact instead of after. Either way, notice what made this possible: not a stronger opinion about the price, but a forecast built from a measured elasticity, an explicit (and flagged) assumption for the harder-to-observe number, and one clean 90-day comparison — exactly the model Exercise 3, Challenge 2, and the mini-project will have you extend.

## 7. Check yourself

- Why is the revenue-maximizing price from `price_experiment` ($33.26) the wrong number to use for repricing *existing* `Growth` customers?
- What does `R² = 0.9956` tell you about the linear demand fit, and would you trust a fit with `R² = 0.40` the same way?
- In your own words, what does `|E| > 1` mean, and why does it imply that raising price in that range loses revenue?
- Why is existing-customer churn elasticity typically lower in magnitude than new-visitor conversion elasticity? Name two mechanisms.
- What's the difference between a grandfathering period and a retention discount — are they doing the same job?
- Define expansion revenue in one sentence, and explain why crediting it to the price-change forecast would overstate the price change's real effect.
- In the 90-day forecast, why did total MRR go up while total customer count went down — walk through the two numbers that make that possible simultaneously.

If those are automatic, you're ready for a real forecast of your own — Exercise 3 has you fit this same demand curve from scratch, and Challenge 2 has you run the full 90-day model on a different proposed price.

## Further reading

- **PostgreSQL — linear regression aggregates (`REGR_SLOPE`, `REGR_INTERCEPT`, `REGR_R2`):** <https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE>
- **NumPy — `polyfit` (least-squares polynomial fit):** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
- **Investopedia — "Price Elasticity of Demand"** (clear worked-example primer on arc vs. point elasticity): <https://www.investopedia.com/terms/p/priceelasticity.asp>
- **ProfitWell / Price Intelligently — "Grandfathering & price increases"** (search "profitwell how to raise prices without losing customers" — practitioner-level treatment of grandfathering and discount offers).
