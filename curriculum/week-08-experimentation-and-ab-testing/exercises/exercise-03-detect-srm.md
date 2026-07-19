# Exercise 3 — Detect Sample-Ratio Mismatch

**Goal:** implement the chi-square goodness-of-fit check from Lecture 3, section 2 yourself, run it against a healthy split, a badly broken one, and an intentionally *uneven* design — and see, directly, how sample size changes what counts as detectable.

**Estimated time:** 60 minutes.

## Setup

Create `srm_check.py` with a hand-written function:

```python
import math
from math import erf

def srm_chi2(observed_a, observed_b, expected_ratio_a=0.5):
    """Chi-square goodness-of-fit test for a 2-arm split against an expected ratio.
    Returns (chi2, p_value)."""
    ...  # implement from Lecture 3 §2 yourself — don't paste it
```

Reminder: `chi2` for a 1-degree-of-freedom test relates to a z-score by `z = sqrt(chi2)`, so you can reuse the same `erf`-based p-value conversion from Lecture 2/3's z-tests. The critical value to know by heart: **3.841** is the df=1 threshold for α=0.05 — any `chi2` above that is a significant mismatch.

## Tasks

1. **Baseline sanity check — the actual pilot.** Query `checkout_sessions` for the real control/treatment counts and run your `srm_chi2` on them. Confirm it's healthy. *(Expected: 50/50, chi2 = 0.0 — a perfectly balanced split, as good as SRM checks get.)*

2. **The broken "Sticky Cart Reminder" test.** A different LoopCart test — a reminder banner shown to visitors who abandon a full cart — was designed as a clean 50/50 split. After a caching bug during a mid-test redeploy, the actual observed counts came back: **control 2,142, treatment 1,858** (4,000 total). Run `srm_chi2(2142, 1858)`. Is this a red alert? What's the p-value telling you about how likely a fair 50/50 randomizer is to produce a split this skewed by chance? *(Expected: chi2 ≈ 20.164, p ≈ 7.1e-06 — an unambiguous SRM.)*

3. **An intentionally uneven design.** Not every test targets 50/50 — a riskier change is sometimes rolled out to a smaller slice on purpose to limit blast radius. LoopCart deliberately designed a **90/10** split (90% control, 10% treatment) for a payment-provider migration test. After running, observed counts are **control 888, treatment 112** (1,000 total). Run `srm_chi2(888, 112, expected_ratio_a=0.9)` — note the different `expected_ratio_a`. Is this healthy or broken? *(Expected: chi2 = 1.6, well under 3.841 — healthy. An "imbalanced-looking" 888/112 split is completely fine when 90/10 was the actual design; the SRM check is always against the *designed* ratio, never a reflexive 50/50 assumption.)*

4. **Same skew, two sample sizes.** Compute `srm_chi2` for two scenarios that have the **exact same proportional skew** (51.5% / 48.5%, both meant to be 50/50):
   - (a) `observed_a=1030, observed_b=970` (n=2,000)
   - (b) `observed_a=10300, observed_b=9700` (n=20,000)

   Report both `chi2` values. One crosses the 3.841 threshold and one doesn't, even though the *proportions* are identical. Explain, in your own words, why — tie your answer to how the SRM chi-square statistic scales with total `n` (compare this to how sample size affects statistical *power* in Lecture 2 — it's the same underlying idea applied to a different test).

## Expected result (spot checks)

- Task 1 → chi2 = 0.0 (perfectly healthy).
- Task 2 → chi2 ≈ 20.164, p ≈ 7.1×10⁻⁶ (clear SRM — stop and find the bug before reading any other result from this test).
- Task 3 → chi2 = 1.6 (healthy, against the correct 90/10 expectation).
- Task 4 → (a) chi2 = 1.8 (not significant), (b) chi2 = 18.0 (significant) — identical skew, different verdict, purely because of `n`.

## Done when…

- [ ] `srm_check.py` has a hand-written `srm_chi2` function you can explain, including why `z = sqrt(chi2)` lets you reuse the z-to-p-value conversion.
- [ ] All 4 tasks are computed with labeled output.
- [ ] You can explain why Task 3's 888/112 split is *not* a red flag, when Task 2's 2142/1858 split — a much less extreme-looking ratio — is.
- [ ] You can explain Task 4's result without saying anything vague like "more data is more accurate" — the answer should specifically reference how the chi-square statistic's magnitude scales with `n` for a fixed proportional skew.

## Stretch

- At what total `n` does the 51.5%/48.5% skew from Task 4 first cross the 3.841 threshold? (Binary-search it, or solve algebraically — the chi2 statistic scales linearly with `n` for a fixed proportional split.)
- Write a one-paragraph incident note, as if for LoopCart's experimentation platform team, explaining what an SRM chi2 = 20.164 means, why the primary-metric result from that test must be discarded (not "adjusted"), and two concrete places you'd look first for the bug (tie this back to Lecture 3, section 2's list of common real-world causes).

## Submission

Commit `srm_check.py` to your portfolio under `c38-week-08/exercise-03/`.
