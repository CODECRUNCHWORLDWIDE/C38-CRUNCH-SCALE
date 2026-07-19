# Mini-Project — Design, Size, and Analyze an A/B Test End to End

> Take LoopCart's next growth idea from a one-sentence pitch all the way to a ship/no-ship recommendation: write the brief, size the test *before* you see any data, generate the (seeded, reproducible) outcome data with the provided script, then analyze it completely in SQL and pandas — significance, confidence interval, guardrail, and a sample-ratio-mismatch check — and defend a real decision. This is the week's capstone: every skill from Lectures 1–3 and Exercises 1–3, applied once, start to finish, on a result you haven't seen the answer to yet.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

Every exercise this week handed you numbers that were already computed, or a dataset that was already run. Real experimentation work starts before any of that — someone has to write the brief, commit to a sample size *before* looking at a single row of outcome data, and only then find out what happened. That order matters enormously: if you size the test **after** generating the data, you'll unconsciously let the result influence what MDE you claim you were always targeting. This mini-project enforces the correct order structurally — you cannot write Step 2 correctly without doing Step 1 first, on faith.

---

## Deliverable

A directory in your portfolio `c38-week-08/mini-project/` containing:

1. `brief.md` — your experiment brief (hypothesis, primary metric, guardrail, randomization unit), written in the shape of Lecture 1, section 7.
2. `sizing.py` — your sample-size calculation, run **before** you generate the dataset, with your stated baseline rate and MDE assumptions.
3. `generate_data.py` — the provided generator script (below), unmodified except for adding your own file paths if needed. **Do not change the seed** — a fixed seed is what makes your results checkable and everyone's dataset comparable.
4. `analysis.py` (or a notebook) — your full analysis: SRM check, primary-metric significance + CI, guardrail check, and a reconciliation between your Step 2 sizing and the actual sample size you got.
5. `decision.md` — your ship/no-ship recommendation, defended in writing (see "The decision" below).

Everything runs in pandas; use SQL (loading the generated CSV into a SQLite or Postgres table) for any step where a `GROUP BY` is more natural than pandas — your choice, both are acceptable this week, per the course's data-tooling rule (SQL and/or pandas, never a spreadsheet).

---

## The brief (write this first — 20 min)

LoopCart's growth team has a new idea: a **"Free shipping over $50" banner** on the cart page, shown only when the cart subtotal is under $50, nudging shoppers toward the threshold. Hypothesis to formalize in `brief.md`:

> If we show a free-shipping-threshold banner on carts under $50, cart→purchase conversion rate will increase, because the banner gives price-sensitive shoppers a concrete, achievable reason to complete checkout instead of abandoning.

Write the complete brief — primary metric (precise numerator/denominator), at least one guardrail (think about what a threshold banner could do *besides* raise conversion — what's the specific failure mode of "nudge people toward a spending target"?), and randomization unit, exactly like Lecture 1, section 7's finished example.

## Step 1 — Size the test before you see any data (30 min)

State your assumptions in `sizing.py` as comments, then compute:

- **Baseline conversion rate: 18%** (LoopCart's current cart→purchase rate — use this exact number, it matches the dataset you'll generate).
- **Target MDE: a 15% relative lift** (use this exact number too).
- α = 0.05, power = 80% (the course's standard settings, per Lecture 2).

Run Exercise 1's sizing formula and report the required sample size **per arm**. Write one sentence in `sizing.py`'s comments committing to what you'll do if the actual dataset turns out smaller than this number — decide that *now*, before you know whether it's true, not after.

## Step 2 — Generate the dataset (10 min)

Run this script exactly as given. **The seed is fixed at `2026` — do not change it.** This isn't your data to invent; it's LoopCart's pilot, already run, and your job starts at analyzing it, the same way it would if you joined the growth team on the Monday after the test ended.

```python
# generate_data.py
import numpy as np
import pandas as pd

rng = np.random.default_rng(2026)   # DO NOT CHANGE — fixed for reproducibility
N_TOTAL = 5000

# --- assignment: a true random 50/50 split (not forced-exact — real
#     randomizers produce natural imbalance; you're checking whether
#     THIS imbalance is within normal noise or an actual SRM) ---
variant = rng.choice(['control', 'treatment'], size=N_TOTAL, p=[0.5, 0.5])

# --- outcome: conversion, secretly variant-dependent (you don't get
#     told the true rates — that's what your analysis is for) ---
p1, p2 = 0.18, 0.18 * 1.15   # NOTE: these happen to match Step 1's assumptions.
                              # That's intentional — figure out during analysis
                              # whether the dataset actually behaves that way.
is_treat = (variant == 'treatment')
p_arr = np.where(is_treat, p2, p1)
converted = rng.binomial(1, p_arr)

# --- covariates, independent of variant (not confounds — safe to ignore
#     unless your SRM/segment-balance check says otherwise) ---
device = rng.choice(['mobile', 'desktop'], size=N_TOTAL, p=[0.62, 0.38])
country = rng.choice(['USA', 'UK', 'Canada', 'Germany', 'Australia', 'Brazil'],
                      size=N_TOTAL, p=[0.40, 0.15, 0.10, 0.15, 0.10, 0.10])

# --- guardrail outcome: order value, only for converted sessions,
#     mildly variant-dependent ---
order_value = np.full(N_TOTAL, np.nan)
conv_idx = converted == 1
n_conv = conv_idx.sum()
ov = np.where(
    is_treat[conv_idx],
    rng.gamma(shape=6, scale=13.3, size=n_conv),   # treatment
    rng.gamma(shape=6, scale=14.0, size=n_conv),    # control
)
order_value[conv_idx] = ov

# --- cosmetic fields (not random — safe to include without affecting
#     the statistics above) ---
session_id = np.arange(1, N_TOTAL + 1)
visitor_id = 10000 + session_id
day_offset = session_id % 14   # spreads sessions evenly across a 14-day pilot

df = pd.DataFrame({
    'session_id': session_id,
    'visitor_id': visitor_id,
    'variant': variant,
    'device': device,
    'country': country,
    'day_offset': day_offset,
    'converted': converted.astype(bool),
    'order_value_usd': order_value,
})

df.to_csv('assignments.csv', index=False)
print(df.groupby('variant').size())   # sanity check only — do NOT peek at conversion here
```

Run it once. You should see two arm sizes that are close to, but not exactly, 2,500 each — that's expected from a true random split, not a bug.

## Step 3 — Analyze it completely (60 min)

In `analysis.py`, in this exact order (Lecture 3, section 6's checklist):

1. **SRM check first.** Run the chi-square goodness-of-fit test (Exercise 3) on the actual arm sizes against the designed 50/50 split. State clearly whether this passes before doing anything else.
2. **Primary metric.** Compute the conversion rate per arm, the two-proportion z-test, the p-value, and the 95% confidence interval for the difference (Lecture 2, sections 4–5).
3. **Guardrail.** Compute average order value per arm (converted sessions only) and the relative change. State whether it breaches your `brief.md` threshold.
4. **Reconcile with Step 1.** Compare the actual arm sizes to your Step 1 required sample size. Are you adequately powered for the MDE you committed to? If not, say so explicitly — this changes how much weight the p-value can bear.
5. **Segment sanity check.** Break the primary metric out by `device`, the same way Lecture 3, section 5 did for Simpson's paradox. Confirm the effect direction is consistent across segments (or report if it isn't).

## The decision (30 min, `decision.md`)

Write LoopCart's ship/no-ship recommendation — for a real Head of Growth, not a professor. Your write-up must address, explicitly:

1. **The call: ship, don't ship, or extend the test — and why**, referencing the actual p-value, CI width, and guardrail result together, not any one number in isolation.
2. **The power reconciliation from Step 3.4** — if the dataset came in short of your Step 1 sizing, say what that does to your confidence in the call, regardless of which way the point estimate leans.
3. **One thing you'd want to monitor for the first 30 days post-launch even after your decision** (tie this to a specific risk from Lecture 3 — novelty effect, a guardrail that needs a longer window than the test had, or something else you noticed).

There is no single correct call here — a defensible "ship, but monitor the guardrail closely" and a defensible "extend two more weeks to reach the pre-registered sample size" can both be strong answers, if the reasoning is sound and actually uses the numbers you computed.

---

## Rules

- **No spreadsheets, anywhere, at any point.** The generated CSV gets loaded into pandas and/or SQL, never opened in a spreadsheet to "eyeball it."
- **Every number in `decision.md` must trace back to a computed value in `analysis.py`** — no hand-waved percentages.
- **Do the sizing (Step 1) before generating the data (Step 2), and do not revise your sizing assumptions after seeing the analysis.** This ordering is the entire point of the exercise — grading checks that `sizing.py` doesn't quietly reference numbers from `analysis.py`.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Brief quality | 15% | Precise primary metric, a guardrail genuinely specific to threshold-banner risk, justified randomization unit |
| Sizing discipline | 20% | Correct formula application, done *before* generating data, assumptions stated and untouched afterward |
| Analysis completeness | 30% | All 5 steps present and correct: SRM, significance, CI, guardrail, power reconciliation, segment check |
| Decision quality | 25% | Recommendation genuinely synthesizes p-value + CI + guardrail + power, not just "p<0.05 so ship" |
| Documentation & honesty | 10% | Clear, numbers-traced write-up; states real uncertainty rather than false confidence |

---

## Reflection (part of `decision.md`, ~200 words)

1. Did your Step 1 sample size match what the dataset actually delivered? If not, how did that change how much you trusted the Step 3 p-value?
2. What would you have concluded if you'd skipped the SRM check and gone straight to the primary metric? Would it have changed your answer, or just your confidence in it?
3. This dataset's guardrail (order value) turned out however it turned out. If it had *breached* your threshold instead, would that alone be enough to kill an otherwise-significant primary-metric win? Why or why not?
4. Name one thing about this whole process that would be genuinely harder with real production data than with this clean, seeded dataset — and what you'd do about it.

---

## Why this matters

Every skill in this course up to this point has been about *measuring* what's true. This week is the first time the course asks you to *change* something and prove the change caused the result — the actual job of a growth or product analytics team, and the single hardest thing to get right under real pressure to ship. Getting the order right (brief → size → run → analyze → decide, never reordered, never skipped) is worth more than any individual formula in this project, because it's the discipline that keeps you honest exactly when a stakeholder is most eager for you not to be.

When done: push, then take the [quiz](../quiz.md) and start Week 9 — Pricing, packaging & price sensitivity, where this exact same machinery gets pointed at a much more contentious question: what will customers actually pay.
