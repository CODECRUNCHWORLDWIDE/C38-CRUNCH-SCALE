# Exercise 1 — Size an A/B Test

**Goal:** turn a baseline rate, an MDE, and your α/power choices into a required sample size and a test duration — the calculation every real test needs *before* it launches, done by hand in Python, no black-box calculator.

**Estimated time:** 75 minutes.

## Setup

Create `solutions.py`. Every task is pure Python arithmetic — no database needed for this exercise. Start from the formula in Lecture 2, section 2, but **write the function yourself** from the formula, don't paste the lecture's version verbatim.

```python
import math

def required_n_per_arm(p1, p2, alpha=0.05, power=0.80):
    # z_alpha: 1.959963985 for alpha=0.05 two-tailed
    # z_power: 0.841621234 for power=0.80, 1.281551566 for power=0.90
    ...  # implement the formula from Lecture 2 §2 yourself
```

## The scenario

LoopCart's payment step is getting a new **trust-badge banner** (a small "secure checkout" seal next to the card field) — a much smaller, cheaper change than Express Checkout, but still worth sizing correctly before it ties up engineering and traffic. Current baseline conversion at this step is **8%**. The team wants to detect a **25% relative lift** (8% → 10%).

## Tasks

1. **Size it at the standard settings.** Compute `required_n_per_arm` for baseline 8% → 10%, α = 0.05, power = 80%. Round **up** to the nearest whole visitor. *(Expected: ≈3,215 per arm.)*

2. **Size it at higher confidence in the result.** Recompute with power raised to 90% (everything else unchanged). Report the new required n, and in one sentence explain *why* it's bigger — tie your answer to what power = 90% vs. 80% actually means (Lecture 2, section 1).

3. **Turn Task 1 into a duration.** LoopCart's payment step sees **1,800 eligible sessions per day**, combined across both arms once the test is live. Compute the number of days needed to reach Task 1's total sample size (both arms), rounding up. Then apply the "always run at least one full 7-day cycle" rule from Lecture 2, section 3 — what's the *actual* minimum test duration you'd recommend, and which of the two numbers (the math-only answer or the 7-day floor) is binding?

4. **See how baseline rate changes the answer.** Recompute Task 1's sample size, but imagine LoopCart's baseline at this step were **12%** instead of 8% — same 25% *relative* lift target (12% → 15%), same α/power. Report the new n and explain, in terms of the formula's numerator and denominator, why it comes out smaller than Task 1's even though the variance term (`p̄(1−p̄)`) is actually larger for the 12% scenario.

5. **Contrast with the pilot.** The `checkout_sessions` seed data (from the week README) ran with only 50 sessions per arm and still came back significant, because its baked-in lift (42% → 64%, a 52% relative lift) was enormous. Compute `required_n_per_arm` for that exact 42% → 64% jump and confirm it's comfortably *under* 50. Then state, in one sentence, why a real MDE almost never looks like this one.

## Expected results

- Task 1 → ≈3,215 per arm (6,430 total).
- Task 2 → ≈4,303 per arm — bigger, because higher power means a smaller tolerance for missing a real effect, which costs more data.
- Task 3 → math-only: 6,430 ÷ 1,800/day ≈ 3.6 days → round up to 4 days. But the 7-day floor is **binding** — recommend 7 days regardless, since the math finishes faster than a full week and day-of-week effects (Lecture 3) make anything shorter unreliable.
- Task 4 → ≈2,037 per arm — smaller than Task 1, because the *absolute* gap between 12% and 15% (3 points) is larger than the gap between 8% and 10% (2 points), and the denominator `(p₂−p₁)²` grows faster than the numerator's variance term shrinks.
- Task 5 → well under 50 per arm — this pilot's lift is roughly 10x bigger than the realistic Task 1 scenario, which is why it was legible with so little data.

## Done when…

- [ ] `solutions.py` has a hand-written `required_n_per_arm` function, not copy-pasted, that you can explain line by line.
- [ ] All 5 tasks are computed and printed with labels.
- [ ] You can explain, out loud, why Task 2's answer is bigger than Task 1's.
- [ ] You can explain why Task 4's answer is smaller than Task 1's despite a larger variance term.

## Stretch

- Plot (with `matplotlib`, or just print a small table) required `n` per arm for relative lifts ranging from 5% to 50% in 5-point steps, baseline fixed at 8%. At what lift size does the required sample size drop under 500 per arm?
- LoopCart's data team says they can realistically get 1,800 eligible sessions/day at the payment step. Given that constraint and a hard 3-week (21-day) limit on how long any one test can run, what's the *smallest* relative lift they could reliably detect (α=0.05, power=80%) in that window? (Work backward: 21 days × 1,800/day = total budget; solve the sizing formula for the MDE that budget affords, by trying a few values.)

## Submission

Commit `solutions.py` to your portfolio under `c38-week-08/exercise-01/`.
