# Challenge 2 — Defend the Growth Plan

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

You have a founder's attention for fifteen minutes. They've seen Lecture 3's three forecast methods disagree by 70% on September's MRR and they want to know two things: **which number do you actually believe, and where should next quarter's marketing dollars go.** This challenge is that fifteen minutes, written down. It is graded as a decision memo a real founder could act on tomorrow morning — not as a bigger SQL script.

## Your task

Produce a `challenge-02.md` containing:

1. **Your own forecast reconciliation.** Reproduce Lecture 3's three methods (linear, moving-average $, compounding %) yourself in Python, then add a **fourth method of your own choosing** — a weighted blend of two existing methods, a simple exponential-smoothing model, a scenario range (best/base/worst case with named assumptions), or another defensible approach. State the formula or logic for your fourth method in plain language before showing the code.
2. **A single chosen number.** State, in one sentence, which forecast (of your four) you'd put in a board deck for September 2025 MRR, and defend the choice using the same reasoning style as Lecture 3, Section 2 — not "it felt right," but "here's what this method assumes, here's why that assumption is more defensible than the alternatives' assumptions, for *this* dataset."
3. **A channel budget recommendation**, built on `fct_unit_economics` — how would you actually reallocate Crunch Boards' $6,450/month total marketing spend (`paid_search` $3,600 + `organic` $2,400 + `referral` $450) across the three channels next quarter? Give specific dollar amounts per channel, not just "shift some money to referral." Justify against CAC, LTV:CAC, and the sample-size caveats from Lecture 2, Section 4 — including whether `referral`'s young, small sample changes how aggressively you'd scale it versus how confidently you'd state its LTV.
4. **A stress test.** Pick **one** assumption underlying your recommendation (the 75% gross margin, the retention curve holding past age 4, `referral`'s zero observed churn, or one of your own) and show, with actual numbers, what happens to your recommendation if that assumption is wrong in the unfavorable direction. Does your recommendation survive, or does it flip?
5. **The memo itself** — 150–250 words, following the four-part decision-narrative shape from Lecture 3, Section 4 (ask, evidence, uncertainty, checkpoint), written as if you were sending it to Crunch Boards' founder today.

## Constraints

- Every dollar figure in your memo must trace to a table or a query you show in this challenge or an earlier exercise — no unsourced numbers.
- Your stress test must use real recomputed numbers, not a hand-waved "this would probably still hold."
- The memo (Task 5) is the graded deliverable everything else supports — a technically perfect forecast with no memo, or a memo with no traceable numbers behind it, both fail this challenge.

## Hints

<details>
<summary>On choosing a fourth forecast method</summary>

A **scenario range** is often the most honest choice for a dataset this size and this volatile (recall June's MRR jump was nearly double any prior month): instead of one number, give "base case: Method B's $2,930, if June's acceleration holds; downside: Method A's $2,564, if June was a one-off; upside: Method C's $4,343, if the acceleration itself is accelerating." Naming three numbers with named conditions is frequently more useful — and more honest — to a founder than a single fabricated-precision figure like "$2,847.12."

</details>

<details>
<summary>On the stress test</summary>

The gross margin assumption is a good one to pressure-test because it's a single input that touches every LTV number at once: drop it from 75% to 60% (a real range for an early-stage SaaS still investing in support/infra) and recompute all three `ltv_cohort_sum` figures. Does `referral` still clear 1:1 LTV:CAC? Does `paid_search` get *more* underwater, or does the gap barely move because it was already so far below CAC that a margin change doesn't matter at that scale? That last question — "does this assumption change actually move the decision, or just the decimal places" — is the whole point of a stress test.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Fourth method | A trivial variant of an existing method with no new reasoning | A genuinely different logic (scenario range, smoothing, weighted blend) with its assumption stated plainly |
| Chosen forecast | "I'll go with the middle one" | Names the specific assumption trade-off that makes one method more defensible *for this dataset* |
| Budget recommendation | "Spend more on referral" | Specific dollar reallocation, three channels, tied to CAC/LTV:CAC/sample-size numbers |
| Stress test | Asserts robustness without recomputing | Real recomputed numbers under the stressed assumption, and an honest answer about whether the recommendation flips |
| The memo | Restates numbers with no narrative arc | Ask → evidence → uncertainty → checkpoint, 150–250 words, every number traceable |

## Submission

Commit `challenge-02.md` and any supporting Python (forecast code, stress-test recomputation) to your portfolio under `c38-week-12/challenge-02/`.
