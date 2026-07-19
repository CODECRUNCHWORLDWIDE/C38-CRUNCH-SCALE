# Challenge 2 — Measure Campaign Lift

**Goal:** Load the (simulated) outcomes of the win-back journey you designed in Challenge 1, compute the observed lift, test its significance, and calculate the sample size that would actually settle the question.

**Estimated time:** 90 minutes.

## Context

Three months have "passed" since Challenge 1's journey launched. Here's what actually happened, generated the same way this week's whole dataset was — deterministically, from a fixed seed, so your numbers match this document's exactly. In the real world this data would come from your product/billing warehouse three months from now; here, it comes from a second seeded script that plays out a specific (but hidden-until-you-calculate-it) treatment effect against the `churned_in_window` outcome your Week 10 model already knows for every customer in this synthetic company.

## Part A — Load the outcomes

Save and run this script (it depends on `churn_scored.csv` from Exercise 3 and the exact split salt from Challenge 1):

```python
import pandas as pd, numpy as np, hashlib

df = pd.read_csv("churn_scored.csv")
seg = df[df["action"] == "winback_email_discount"].copy()

def split_group(uid):
    h = int(hashlib.md5(f"crunch-flow-winback-2025-09|{uid}".encode()).hexdigest(), 16)
    return "treatment" if (h % 10) < 5 else "holdout"

seg["group"] = seg["user_id"].apply(split_group)
seg = seg.sort_values("user_id").reset_index(drop=True)

TRUE_SAVE_RATE = 0.45   # hidden effect size — DO NOT look at this until Task 5
rng = np.random.default_rng(42)
outcomes = []
for _, row in seg.iterrows():
    would_churn = row["churned_in_window"] == 1
    if row["group"] == "holdout":
        outcomes.append(1 if would_churn else 0)
    else:
        if would_churn:
            outcomes.append(0 if rng.random() < TRUE_SAVE_RATE else 1)
        else:
            outcomes.append(0)
seg["actual_churned_90d"] = outcomes
seg.to_csv("winback_outcomes.csv", index=False)
print(seg["group"].value_counts())
```

*(Expected: `treatment` **30**, `holdout` **28** — matching Challenge 1, Part C.)*

## Tasks

1. **Group-level outcome table.** `groupby("group")` on `n`, `sum(actual_churned_90d)`, and the resulting churn rate. *(Expected: holdout **16/28 = 57.1%**, treatment **13/30 = 43.3%**.)*

2. **Lift.** Compute absolute lift (percentage points) and relative lift (%) from Task 1. *(Expected: **13.8 pp** absolute, **24.2%** relative.)*

3. **Significance test.** Run a two-proportion z-test (`statsmodels.stats.proportion.proportions_ztest`) on the counts from Task 1. *(Expected: **z ≈ −1.05**, **p ≈ 0.293**.)* State, in one sentence, whether this result is statistically significant at the conventional 5% threshold — and don't soften the answer.

4. **What p = 0.293 does and doesn't mean.** In 3–4 sentences, explain what you can and cannot conclude from this p-value. Specifically address: does a non-significant result mean the campaign definitely didn't work? Does the 13.8-point observed lift mean it definitely did?

5. **Reveal and reconcile.** Now look at `TRUE_SAVE_RATE = 0.45` in the script. In the simulation, the journey was designed to save 45% of customers who would otherwise have churned. Compare that ground truth to your Task 2 relative lift (24.2%). Why don't they match exactly, even though the simulation's true effect is known? (Hint: relative lift is measured against the holdout's *realized* churn rate, a single noisy draw from the same underlying probabilities — not against the true population parameter.)

6. **Power calculation.** Using `statsmodels.stats.power.NormalIndPower` and `proportion_effectsize`, compute the sample size **per group** needed to detect the observed 57.1%-vs-43.3% difference at 80% power and 5% significance. *(Expected: **≈ 205 per group**, roughly **410 total** — about 7× the current segment size.)*

7. **A real recommendation.** Write a half-page recommendation, as if to Crunch Flow's Head of Growth, covering: (a) what you observed, (b) whether you can act on it today, (c) at least two concrete options for getting a trustworthy answer (e.g., running the journey continuously and re-analyzing quarterly as the segment naturally grows past 205/group; loosening the significance threshold if the cost of a false positive is genuinely low; combining this evidence with a secondary metric like support-ticket volume). Do not recommend "just wait and see" without a number attached to it.

## Done when…

- [ ] Tasks 1–3 reproduce the exact numbers given.
- [ ] Task 4's explanation correctly distinguishes "not significant" from "definitely no effect."
- [ ] Task 5 correctly explains why 45% (true) ≠ 24.2% (observed relative lift), referencing sampling noise.
- [ ] Task 6's power calculation lands within ~10% of 205 per group.
- [ ] Task 7's recommendation names at least two concrete, numbered paths forward — not just "run it longer."

## Stretch

- Re-run the outcome simulation with a different `TRUE_SAVE_RATE` (try `0.15`, a much smaller true effect) and recompute the p-value and required sample size. How much bigger does the test need to be to detect a smaller true effect at the same confidence?
- Compute a 95% confidence interval on the lift (not just a point estimate) using `statsmodels.stats.proportion.proportion_confint` on each group separately, and report the interval on the *difference*. Does the interval include zero? Does that match your Task 3 conclusion?
- If Crunch Flow ran this exact journey every month indefinitely (not just once), sketch — in words, no code required — how you'd design a continuous experimentation program (Week 8) so the holdout group doesn't have to stay small forever.

## Submission

Commit `challenge-02.md` (your write-up, Tasks 4/5/7) plus your analysis script to your portfolio under `c38-week-10/challenge-02/`.
