# Lecture 3 — Experimentation Traps

> **Duration:** ~2 hours. **Outcome:** You can name, detect with a query, and defend against the five most common ways a "clean" A/B test result is actually lying: peeking, sample-ratio mismatch, novelty effects, multiple comparisons, and Simpson's paradox.

A test can have a perfect hypothesis, a correctly computed sample size, and a p-value under 0.05 — and still be wrong. Every trap in this lecture produces output that *looks* like a normal, well-behaved result. That's what makes them dangerous: nothing about the number itself tells you it's broken. You have to know to check.

## 1. Peeking — checking before you're done, then stopping when you like what you see

**The trap:** you pre-registered a 7-day test at α = 0.05. On day 3 you get curious and run the significance query. It's not significant, so you keep going. On day 6, it crosses p < 0.05. You stop the test and ship.

**Why this is broken:** α = 0.05 is a promise about *one* look at the data — "if there's truly no effect, I'll falsely call it significant 5% of the time." Every additional look is another chance for noise to cross the line, and the promise was never about "at least one of several looks." Run the LoopCart pilot's actual day-by-day trajectory (cumulative sessions and z-test, re-run each day as more of the week's data arrives):

```sql
SELECT
    DATE(assigned_at)                          AS cutoff_day,
    variant,
    SUM(COUNT(*)) OVER (PARTITION BY variant ORDER BY DATE(assigned_at))                             AS cum_sessions,
    SUM(COUNT(*) FILTER (WHERE converted)) OVER (PARTITION BY variant ORDER BY DATE(assigned_at))     AS cum_conversions
FROM checkout_sessions
GROUP BY DATE(assigned_at), variant
ORDER BY cutoff_day, variant;
```

Feed each day's cumulative counts through the z-test from Lecture 2 and the p-value trajectory looks like this:

| Cutoff day | n (control / treatment) | z | p-value |
|-----------:|:------------------------|-----:|--------:|
| Jun 1 | 7 / 7 | 0.53 | 0.593 |
| Jun 2 | 15 / 14 | 0.92 | 0.356 |
| Jun 3 | 22 / 22 | 1.21 | 0.228 |
| Jun 4 | 29 / 29 | 1.58 | 0.115 |
| Jun 5 | 36 / 36 | 1.89 | 0.059 |
| Jun 6 | 43 / 43 | 2.16 | **0.031** |
| Jun 7 | 50 / 50 | 2.20 | **0.028** |

A team peeking daily and stopping at the first crossing would ship on **June 6**, a day early, having watched the p-value approach the line for five straight days first. In this particular dataset the effect is genuinely real and large, so the early stop happens to land on a true positive — but the *policy* of "check daily, stop at first crossing" is broken regardless of whether this specific run got lucky, because it changes your actual false-positive rate. **In real (noisier) data, p-value trajectories bounce up and down constantly rather than sliding smoothly toward zero** — a curve that dips under 0.05 on day 4 by chance and climbs back above it by day 7 is completely ordinary under the null hypothesis, and "stop at first crossing" would ship on day 4's noise every time it happened to dip.

**The size of the problem, in one number:** if a metric has truly zero effect and you check it once a day for 6 days at α = 0.05 each time, the chance that *at least one* of those 6 checks shows "significant" purely by chance is:

```
1 − (1 − 0.05)⁶ = 1 − 0.95⁶ ≈ 0.265   (26.5%, not 5%)
```

**The defense:** commit to the sample size and duration *before* the test starts (Lecture 2), and only look at the result once, at the end. If your team genuinely needs to monitor a live test (for safety — a guardrail crashing, say), use a **sequential testing method** designed for repeated looks (e.g., always-valid p-values, group sequential designs) — these exist specifically because "just don't look" isn't realistic for a launch a business is watching closely. What's never acceptable is applying a fixed-horizon test's α = 0.05 threshold to a result you peeked at more than once.

## 2. Sample-ratio mismatch (SRM) — the randomizer itself is broken

**The trap:** you designed a clean 50/50 split. But the *actual* traffic that landed in each arm isn't close to 50/50 — maybe control got 2,142 sessions and treatment got 1,858, out of 4,000 total. Before you trust a single significance number from this test, you have to ask: **is the assignment mechanism itself broken?**

**Why this happens in real systems, constantly:** a redirect that's slightly slower for one variant and times out more often on flaky mobile connections; a caching layer that pins some visitors to a stale variant after a redeploy; a bot-filtering rule that happens to catch one variant's URL pattern more than the other's; an app-store review gate that delays treatment rollout to a slice of users. None of these are statistical problems — they're plumbing bugs — but they poison every statistical conclusion downstream, because the two groups are no longer comparable.

**The detector: a chi-square goodness-of-fit test** against the *expected* split.

```python
import math
from math import erf

n_control, n_treatment = 2142, 1858
total = n_control + n_treatment
expected = total / 2   # expected count per arm under a true 50/50 split

chi2 = (n_control - expected) ** 2 / expected + (n_treatment - expected) ** 2 / expected
z = math.sqrt(chi2)                                    # chi-square(df=1) ≡ z² under the null
p_value = 2 * (1 - 0.5 * (1 + erf(abs(z) / math.sqrt(2))))

print(f"chi2={chi2:.3f}  p-value={p_value:.2e}")
# chi2=20.164  p-value=7.1e-06
```

A p-value of 0.0000071 says: a split this skewed from 50/50, on 4,000 sessions, would essentially never happen from a genuinely fair 50/50 randomizer. This is a **red alert**, not a borderline judgment call — SRM this severe means **stop, do not read the primary metric, go find the plumbing bug.** Any "the treatment won!" conclusion drawn from an SRM'd test is not analyzing an experiment — it's analyzing whatever selection bias caused the mismatch in the first place (the two groups likely differ systematically in ways that have nothing to do with your feature).

**The rule:** run the SRM check **first**, before looking at any other result, every single time. It's one query and it's the cheapest insurance this entire lecture offers.

## 3. Novelty (and primacy) effects — the honeymoon period lies

**The trap:** a redesigned checkout gets a burst of curiosity clicks in week 1 — people notice the new page and explore it, independent of whether it's actually better. That burst inflates the treatment effect early and fades as it becomes routine. The mirror-image problem, **primacy effect**, is the opposite: a change that's genuinely better takes a week or two to earn trust (returning users have to unlearn the old flow), so early results *understate* the true effect.

**Why this matters for LoopCart specifically:** the pilot only ran one week. If Express Checkout's real week-1 lift is partly curiosity rather than durable preference, a full rollout — measured in week 4, once novelty has worn off — could show a smaller effect than the pilot promised. The reverse is also possible: if some of the win comes from returning users needing time to build trust in a new flow, a one-week pilot could *understate* the long-run effect.

**The detector:** segment the treatment effect **by visitor tenure or by time since the test started**, and watch whether it decays (novelty) or grows (primacy) instead of holding steady:

```sql
SELECT
    variant,
    CASE WHEN assigned_at < '2026-06-04' THEN 'first_half' ELSE 'second_half' END AS test_period,
    COUNT(*)                                                                      AS sessions,
    COUNT(*) FILTER (WHERE converted)                                             AS conversions,
    ROUND(COUNT(*) FILTER (WHERE converted)::NUMERIC / COUNT(*), 3)               AS rate
FROM checkout_sessions
GROUP BY variant, test_period
ORDER BY variant, test_period;
```

If treatment's lift over control is much bigger in `first_half` than `second_half`, that's the novelty-effect signature. A single-week pilot literally cannot distinguish "genuine, durable +22 points" from "a first-week curiosity bump that will fade" — which is exactly why a real launch decision for a big UI change should extend a pilot's promising early read with a longer confirmation window (or a **holdback**: keep a small % on the old experience even after a "winning" launch, specifically to keep measuring whether the win holds up over months, not just the pilot's window).

## 4. Multiple comparisons — testing everything, correcting for nothing

**The trap:** the primary metric (conversion) came back significant, so on the way to writing up the result you also check 4 more metrics out of curiosity — time-on-page, cart-abandon rate, mobile-only conversion, and a "power users" subgroup — and one of them, mobile-only conversion, also clears p < 0.05. You report it as a second win.

**Why it's broken:** each individual test at α = 0.05 has a 5% false-positive rate *on its own*. Run 5 independent tests where nothing is actually true, and the chance that **at least one** shows "significant" by chance is:

```
1 − (1 − α)^k
```

```python
for k in [1, 3, 5, 10]:
    print(k, 1 - (1 - 0.05) ** k)
# 1  → 5.0%
# 3  → 14.3%
# 5  → 22.6%
# 10 → 40.1%
```

Test 5 metrics with no correction and your *real* chance of a spurious "win" somewhere in the batch is **22.6%**, not 5%. Subgroups make this worse fast — "mobile," "desktop," "USA," "returning visitors," "power users" is 5 more comparisons before you've even started on secondary metrics, and subgroup cuts are usually the least defensible of all (see section 5).

**The defense, in order of how often each is actually used:**

- **Designate exactly one primary metric before the test** (Lecture 1) and treat everything else as **informational**, never as a second ship/no-ship signal on its own.
- If you truly need several confirmatory metrics to all matter, apply a correction — the simplest is the **Bonferroni correction**: divide α by the number of comparisons (5 metrics → use α = 0.01 for each, not 0.05) before calling any of them significant.
- **Report subgroup results as hypothesis-generating, not hypothesis-confirming.** "Mobile looked stronger than desktop" is a reason to design a *follow-up* test focused on mobile — it is not, by itself, evidence that mobile users specifically benefit.

## 5. Simpson's paradox — the aggregate reverses the subgroups

**The trap, and the most counterintuitive one in this lecture:** a result can be **worse for treatment than control overall**, while being **better for treatment in every single subgroup**. This isn't a rare statistical curiosity — it happens whenever a confounding variable is unevenly distributed between the two arms.

Here's a real shape it takes, from a checkout test at a different company we'll call **NimbleCart**, where a bug caused mobile app users to be under-enrolled in the treatment arm relative to desktop:

| Segment | Control (n, conversions, rate) | Treatment (n, conversions, rate) |
|---------|:---|:---|
| Mobile | 180, 144, **80.0%** | 20, 17, **85.0%** |
| Desktop | 20, 4, **20.0%** | 180, 45, **25.0%** |
| **Overall** | **200, 148, 74.0%** | **200, 62, 31.0%** |

Look closely: treatment beats control on mobile (85.0% > 80.0%) **and** on desktop (25.0% > 20.0%) — treatment is better in *every* subgroup. But overall, treatment's 31.0% looks catastrophically worse than control's 74.0%. Nothing about the arithmetic is wrong. What's wrong is that **control's traffic is 90% mobile (the easy-to-convert segment) while treatment's traffic is 90% desktop (the hard-to-convert segment)** — the two arms aren't comparable populations, because whatever assignment bug happened, it correlated with device.

**This is functionally an SRM problem hiding one layer deeper** — the *overall* split might even look fine (200 vs. 200!), while the split *within a segment that matters* is badly broken. That's exactly why an SRM check on total counts alone isn't sufficient by itself for anything you plan to slice by segment later.

**The detector:** always compute the primary metric **both overall and broken out by any variable that plausibly correlates with the randomization mechanism** (device, country, signup channel, traffic source) — and check that each subgroup's *share of traffic* is close to the same between arms, not just that the metric direction agrees:

```sql
SELECT
    device,
    variant,
    COUNT(*)                                                        AS sessions,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (PARTITION BY variant), 1) AS pct_of_arm
FROM checkout_sessions
GROUP BY device, variant
ORDER BY device, variant;
```

If `pct_of_arm` for `mobile` is, say, 62% in control and 24% in treatment, that mismatch — not the headline metric — is the first thing to investigate. **The general defense:** never trust an aggregate number from a randomized test without also checking that the randomization actually balanced the segments you'd want to slice by later. When it didn't, the fix is almost never "reweight and recompute" (that reintroduces the exact bias you're trying to escape) — it's "find and fix the assignment bug, then re-run the test."

## 6. The five-trap checklist

Before reading a single significance number from any test, in this order:

1. **SRM** — does the overall split match the design? (section 2)
2. **Segment balance** — does the split match the design *within* the segments you care about? (section 5)
3. **Peeking discipline** — was this read taken once, at the pre-registered horizon, or was the test stopped early because it "looked done"? (section 1)
4. **Novelty/primacy** — does the effect hold steady across the test window, or is it decaying/growing? (section 3)
5. **Comparison count** — how many metrics and subgroups were checked, and was the primary metric's significance threshold the only one treated as a ship/no-ship signal? (section 4)

A result that survives all five is one you can actually act on. A result that skips straight to "p < 0.05, ship it" has survived none of them.

## 7. Check yourself

- Why does checking a test's p-value every day and stopping at first significance change the *real* false-positive rate, even though each individual check still uses α = 0.05?
- What's the difference between an SRM check on total counts and a segment-balance check? Give an example where the first passes and the second fails.
- LoopCart's pilot ran one week. What specific evidence would tell you the +22-point lift is a novelty effect rather than durable?
- Compute, without looking it up again, the chance of at least one false positive across 3 uncorrected comparisons at α = 0.05.
- In the NimbleCart example, treatment wins every subgroup but loses overall. What's the confounding variable, and how did it get unevenly distributed between arms?
- Why is "reweight the subgroups after the fact and recompute" usually the wrong fix for a Simpson's-paradox-driven result?

You now have the full toolkit: design (Lecture 1), sizing and significance (Lecture 2), and the five traps that invalidate a result that looks clean (this lecture). The exercises put all three to work on real numbers; the mini-project has you run the whole pipeline yourself, end to end.

## Further reading

- **Kohavi, Tang, Xu — *Trustworthy Online Controlled Experiments*, Ch. 3 (SRM) and Ch. 20 (peeking / sequential testing)**
- **Fabijan et al. — "Diagnosing Sample Ratio Mismatch in Online Controlled Experiments"** (Microsoft/Booking.com research paper on SRM in production systems): search "Fabijan Diagnosing Sample Ratio Mismatch" — widely available as a free PDF.
- **Wikipedia — Simpson's Paradox** (the classic UC Berkeley admissions and kidney-stone-treatment examples, both public-domain statistics facts): <https://en.wikipedia.org/wiki/Simpson%27s_paradox>
- **PostgreSQL — Window functions** (used for the cumulative peeking-trajectory query): <https://www.postgresql.org/docs/current/tutorial-window.html>
