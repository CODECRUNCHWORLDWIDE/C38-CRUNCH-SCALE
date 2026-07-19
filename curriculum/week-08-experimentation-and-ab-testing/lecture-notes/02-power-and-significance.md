# Lecture 2 — Power, Sample Size, and Significance

> **Duration:** ~2 hours. **Outcome:** You can compute how many visitors and how many days a test needs before it launches, and you can compute — and correctly interpret — a two-proportion z-test and a 95% confidence interval once it's done, in SQL and in pandas.

Lecture 1 gave LoopCart's test a hypothesis, a primary metric, a guardrail, and a randomization unit. This lecture gives it the two things that turn "we ran a test" into "we can trust what the test said": a sample size computed *before* launch, and a significance read computed correctly *after*.

## 1. Four numbers that determine everything

Before you can size a test, you need to commit to four numbers. Every one of them is a judgment call made in advance — that's the whole point.

| Symbol | Name | What it means | Typical value |
|--------|------|----------------|----------------|
| **p₁** | Baseline rate | Your primary metric's current value, under control | Measured from historical data |
| **MDE** | Minimum detectable effect | The smallest lift you actually care about catching | A business decision, not a stats one |
| **α** (alpha) | Significance level | Your tolerance for a **false positive** — calling a real-nothing a win | 0.05 (5%) |
| **1 − β** (power) | Statistical power | Your tolerance for a **false negative** — missing a real effect | 0.80 (80%) |

**α and β are two different mistakes, and they trade off against each other:**

- A **Type I error** (false positive, controlled by α) is shipping a change that does nothing, because random noise happened to look like a win. α = 0.05 means: if the true effect is exactly zero, you'll still see "significant" 5% of the time, by chance alone, every time you run this test.
- A **Type II error** (false negative, controlled by β) is killing a change that actually works, because your test didn't have enough visitors to tell the real effect apart from noise. Power = 80% means: if the true effect is exactly your MDE, you'll correctly detect it 80% of the time.

You cannot drive both errors to zero with a fixed sample size — the only lever that shrinks both at once is **more data**. Sizing a test is choosing how much of that lever you're willing to pull.

**MDE deserves its own sentence, because it's the number teams most often skip.** The MDE is not "the effect we hope to see" — it's "the smallest effect that would still be worth the engineering cost of building and maintaining this feature." A checkout redesign that lifts conversion by 0.1% (relative) probably isn't worth shipping even if you could prove it's real — so there's no reason to size a test that could detect something that small. Pick an MDE that reflects the actual decision threshold, and the required sample size will reflect it too.

## 2. The sample size formula, derived

For a **two-proportion test** — comparing a conversion rate between two groups — the standard sample-size formula per arm is:

```
              2 · p̄ · (1 − p̄) · (z_α/2 + z_β)²
n_per_arm  =  ────────────────────────────────
                          (p₂ − p₁)²
```

Where `p̄ = (p₁ + p₂) / 2` (the average of the two rates), and `p₂ = p₁ × (1 + relative MDE)` if your MDE is stated as a relative lift.

**Where this comes from, in one paragraph:** both groups' conversion counts are approximately normally distributed (for reasonably large n, by the Central Limit Theorem), each with variance `p(1-p)/n`. The difference of two independent normals is itself normal, with variance equal to the sum of the two variances. `z_α/2` is how many standard deviations out you need to go to be 95% confident you're not looking at noise; `z_β` is how many more you need so that a *true* effect of size MDE clears that bar 80% of the time, not just 50%. Squaring and rearranging for `n` gives the formula above. You don't need to re-derive this every time — you do need to understand that it's not a magic incantation, it's "how much variance can I tolerate divided by how big a signal am I looking for," squared.

**The two `z` values, for the settings almost everyone uses:**

| Setting | z-value |
|---------|---------|
| α = 0.05, two-tailed → z_α/2 | 1.96 |
| Power = 80% → z_β | 0.84 |
| Power = 90% → z_β | 1.28 |

## 3. Worked example: sizing LoopCart's next test

Forget the pilot's huge 42% → 64% jump for a moment — that was sized to be legible by hand, not realistic. Here's a **realistic** ask: LoopCart's baseline checkout conversion is **18%**, and the team wants to detect a **15% relative lift** (18% → 20.7%) with the standard α = 0.05, power = 80%.

```python
import math

def required_n_per_arm(p1, p2, alpha=0.05, power=0.80):
    z_alpha = 1.959963985   # z for alpha=0.05, two-tailed
    z_power = 0.841621234   # z for power=0.80
    p_bar = (p1 + p2) / 2
    return 2 * p_bar * (1 - p_bar) * (z_alpha + z_power) ** 2 / (p2 - p1) ** 2

p1 = 0.18
p2 = 0.18 * 1.15          # 15% relative lift
n = required_n_per_arm(p1, p2)
print(f"required n per arm: {n:.0f}")   # ≈ 3,361
```

**≈3,361 visitors per arm — 6,722 total** — just to reliably detect a 15% relative lift on an 18% baseline. Compare that to the pilot's 50-per-arm, which only worked because its baked-in lift (52% relative) was enormous. This is the single most common way real experimentation programs get into trouble: someone runs a test at whatever traffic happens to be available, gets a "not significant" result, and concludes the change doesn't work — when the honest conclusion is "we never had enough power to know."

**Turning a sample size into a duration** just needs your daily eligible traffic:

```python
daily_cart_sessions = 2400     # both arms combined
total_needed = n * 2
days_needed = total_needed / daily_cart_sessions
print(f"days needed: {days_needed:.1f}")   # ≈ 5.6 days → round UP to 6 full days minimum
```

**Two rules for rounding a duration, both non-negotiable:** round the day count **up**, never down or to the nearest — a test that ends 3 hours short of its target is underpowered, full stop. And run **at least one full 7-day cycle** regardless of what the math says, even if the sample-size math finishes in 3 days — Lecture 3 explains exactly why day-of-week effects make anything shorter unreliable.

## 4. Reading a finished test: the two-proportion z-test

Once a test has run its full pre-registered duration, you compute significance. This is the formula that answers: **"is the observed difference bigger than what pure chance could plausibly produce?"**

```
                p̂₂ − p̂₁
z  =  ──────────────────────────────────────
       √( p̄(1 − p̄) · (1/n₁ + 1/n₂) )
```

`p̂₁`, `p̂₂` are the *observed* rates in each arm; `p̄` is the **pooled** rate across both arms (used only for the significance test itself, under the null-hypothesis assumption that both arms truly have the same rate). This is the same shape of formula as the sizing formula in section 2 — sizing asks "how big does n need to be for a given effect," significance asks "given this n and this effect, how many standard deviations from zero am I."

**Compute it against the LoopCart pilot** (`checkout_sessions` from the [week README](../README.md)) in SQL first, to get the raw counts:

```sql
SELECT
    variant,
    COUNT(*)                                   AS sessions,
    COUNT(*) FILTER (WHERE converted)          AS conversions,
    ROUND(COUNT(*) FILTER (WHERE converted)::NUMERIC / COUNT(*), 4) AS rate
FROM checkout_sessions
GROUP BY variant
ORDER BY variant;
```

```
 variant   | sessions | conversions | rate
-----------+----------+-------------+------
 control   |    50    |     21      | 0.42
 treatment |    50    |     32      | 0.64
```

(SQLite: replace `::NUMERIC` with `1.0 *`, e.g. `1.0 * COUNT(*) FILTER (WHERE converted) / COUNT(*)`.)

Then the test itself, in Python:

```python
import math
from math import erf

n1, x1 = 50, 21    # control: sessions, conversions
n2, x2 = 50, 32    # treatment: sessions, conversions

p1, p2 = x1 / n1, x2 / n2
p_pool = (x1 + x2) / (n1 + n2)
se_pool = math.sqrt(p_pool * (1 - p_pool) * (1 / n1 + 1 / n2))
z = (p2 - p1) / se_pool
p_value = 2 * (1 - 0.5 * (1 + erf(abs(z) / math.sqrt(2))))   # two-tailed

print(f"p1={p1:.2%}  p2={p2:.2%}  z={z:.3f}  p-value={p_value:.4f}")
# p1=42.00%  p2=64.00%  z=2.204  p-value=0.0275
```

**p-value = 0.0275.** In plain language: *if Express Checkout truly did nothing at all* (the null hypothesis), a gap this large or larger between two 50-person samples would happen by pure chance about 2.75% of the time. That's below the pre-registered α = 0.05, so this result clears the significance bar.

## 5. The confidence interval — the number the p-value hides

A p-value answers one narrow question: "is this distinguishable from zero?" It says **nothing** about how big or how precisely known the effect is. That's what a **confidence interval (CI)** is for, and a serious result report always includes one.

```python
se_unpooled = math.sqrt(p1 * (1 - p1) / n1 + p2 * (1 - p2) / n2)  # NOT pooled — CI uses each arm's own variance
diff = p2 - p1
ci_low = diff - 1.96 * se_unpooled
ci_high = diff + 1.96 * se_unpooled
print(f"observed lift: {diff:+.1%}  95% CI: [{ci_low:+.1%}, {ci_high:+.1%}]")
# observed lift: +22.0%  95% CI: [+2.9%, +41.1%]
```

Note the CI uses `p1(1-p1)/n1 + p2(1-p2)/n2` — **each arm's own observed variance**, not the pooled one. Pooling assumes the null hypothesis (both rates equal) is true, which is exactly what you're trying to estimate around here, not assume.

**Read that interval honestly: [+2.9 percentage points, +41.1 points].** The point estimate is a dramatic +22 points, but the interval is enormous — with only 50 sessions per arm, the *true* lift could plausibly be as small as +2.9 points (barely worth shipping) or as large as +41 (huge). "Statistically significant" tells you the interval doesn't cross zero. It tells you nothing about whether the interval is narrow enough to act on with confidence. **A wide CI with a real (nonzero) lower bound means "probably worked, but we don't yet know by how much" — that is not the same claim as "we know it worked by a lot."** Whenever anyone reports a test result to you with a p-value and no interval, ask for the interval before you believe the headline.

## 6. Checking the guardrail too

Significance testing isn't just for the primary metric — LoopCart's brief also committed to an AOV guardrail. Compute both sides:

```sql
SELECT
    variant,
    ROUND(AVG(order_value_usd)::NUMERIC, 2) AS aov,
    COUNT(*) FILTER (WHERE converted)       AS conversions
FROM checkout_sessions
WHERE converted
GROUP BY variant;
```

```
 variant   |  aov   | conversions
-----------+--------+-------------
 control   |  78.00 |     21
 treatment |  70.31 |     32
```

AOV fell from $78.00 to $70.31 — a **9.9% relative drop**, under the pre-committed 10% guardrail threshold from Lecture 1, but close enough to watch, not ignore. Before calling the guardrail "fine," check the metric that actually pays the bills — **revenue per cart session**, converted or not:

```sql
SELECT
    variant,
    ROUND(SUM(COALESCE(order_value_usd, 0))::NUMERIC / COUNT(*), 2) AS revenue_per_session
FROM checkout_sessions
GROUP BY variant;
-- control: 32.76   treatment: 45.00   (+37.4% relative)
```

Even with AOV down almost 10%, revenue per session is **up 37%**, because the conversion-rate gain more than offsets the smaller average order. This is exactly why a primary metric and a guardrail can each look concerning in isolation but tell a coherent, good story together — and why you check both, always, before writing "ship it."

## 7. Check yourself

- In your own words, what does α = 0.05 control, and what does power = 80% control? Which one governs the risk of shipping a dud; which governs the risk of killing something real?
- Why does a smaller MDE require a *larger* sample size, not a smaller one?
- LoopCart's realistic sizing example needed ≈3,361 per arm for a 15% relative lift on an 18% baseline. Would detecting a 25% relative lift need more or fewer visitors? Why?
- What's the difference between the pooled standard error (used for the z-test) and the unpooled standard error (used for the CI)? Why does each use the assumption it uses?
- The pilot's result was "significant" with a CI of [+2.9%, +41.1%]. Explain, to someone who thinks a p-value alone is enough, why that CI matters just as much as the 0.0275.
- Why should a test's duration always round *up* to a full number of days, and never end early even if the running z-test crosses significance sooner?

Lecture 3 is where a test that looks exactly this clean turns out to be lying — five specific ways, each with its own SQL detector.

## Further reading

- **Evan Miller — "How Not To Run an A/B Test"**: <https://www.evanmiller.org/how-not-to-run-an-ab-test.html>
- **Evan Miller — Sample Size Calculator** (useful for sanity-checking your own hand-rolled formula): <https://www.evanmiller.org/ab-testing/sample-size.html>
- **Kohavi, Tang, Xu — *Trustworthy Online Controlled Experiments*, Ch. 4 (Metrics) and Ch. 19 (Power)**
- **Python `math` module docs** (for `erf`, used to turn a z-score into a p-value without a stats library): <https://docs.python.org/3/library/math.html#math.erf>
