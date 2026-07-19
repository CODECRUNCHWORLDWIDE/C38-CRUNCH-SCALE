# Challenge 2 — Compare Retention Across Segments

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch has two ways people get onboarded: `self_serve` (sign up, figure it out yourself — 36 of 48 users) and `sales_assisted` (a human walks you through setup — 12 of 48 users). Someone on the leadership team has a hunch: "sales-assisted customers stick around better, so we should put more of the acquisition budget into a sales-assisted motion, even though it's more expensive per signup." Your job is to build the actual comparison — carefully — and say whether the data supports that call, and how strongly.

## Your task

Write `challenge-02.md` containing:

1. **Two retention comparisons, built two different ways:**
   - **Per-cohort, per-segment.** For each of the four cohorts, break month-1 retention out by segment (8 numbers total: 4 cohorts × 2 segments).
   - **Pooled across cohorts, per segment.** Using the eligible-adjusted approach from Lecture 3, compute each segment's retention curve blended across all applicable cohorts, at every `month_number` you have data for.
   
   Show both queries and both result sets.

2. **A written comparison of the two views.** Do they tell the same story? Where do they disagree, and *why* would two correct queries about "the same" question give different-looking numbers? (Hint: think about what each approach's sample size looks like, cell by cell, and what each one is and isn't controlling for.)

3. **A stickiness cross-check.** Pull in the DAU/MAU stickiness numbers by segment from Exercise 3. Does the stickiness gap between segments point the same direction as the retention gap? If a stakeholder only had one of the two metrics, could they be misled?

4. **A recommendation with a stated confidence level.** Does the data support "invest more in sales-assisted onboarding"? State your recommendation, and separately state how confident you are and what would raise or lower that confidence (e.g., more users per segment, more months of history, a controlled test rather than an observational comparison).

## Constraints

- You must show numbers from **both** comparison methods (per-cohort-per-segment, and pooled). A submission with only one is incomplete — the whole point of the challenge is noticing they can look different.
- Segment is not randomly assigned — sales-assisted customers likely differ from self-serve customers in ways beyond just "got help onboarding" (company size, budget, intent). Address this directly: is "sales-assisted causes better retention" the only explanation consistent with your numbers, or is there at least one confound you can name?
- Cite specific counts (not just percentages) anywhere a sample is small enough that the percentage alone could mislead.

## Hints

<details>
<summary>On why the two comparison methods can disagree</summary>

Per-cohort-per-segment cells are small — 9 `self_serve` and 3 `sales_assisted` users per cohort. A single user moving from "churned" to "retained" swings a 3-person cell by 33 points and a 9-person cell by 11 points. The pooled, eligible-adjusted view (Lecture 3 §2) has much larger denominators (up to 36 for `self_serve`, up to 12 for `sales_assisted`) and is the more stable number to actually act on — but it blends cohorts together, so it can't show you *whether the gap is closing over time* the way the per-cohort view can. Neither view is "wrong"; they answer different questions.

</details>

<details>
<summary>On confounding</summary>

`sales_assisted` isn't randomly assigned in this dataset (or in most real companies) — it usually correlates with deal size, company size, or how much a prospect already wanted the product before they ever signed up. A real investigation would want to know: are sales-assisted customers retaining better *because* of the onboarding help, or because they were already more likely to stick around for reasons unrelated to onboarding (bigger budget, more organizational buy-in)? You can't fully resolve this with observational data alone — say so, and name what a cleaner test would look like. (This is exactly the kind of question Week 8's experimentation unit exists to answer properly.)

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Both views built | Only the easier one | Both, with correct SQL for each |
| Reconciling the disagreement | Ignores that they differ, or picks whichever is more convenient | Explains *why* they can differ and which to trust for which question |
| Confounding | Treats segment as if it were randomly assigned | Names at least one plausible confound and its implication |
| Recommendation | Flat "yes, invest more" or "no, don't" | A recommendation with an explicit, reasoned confidence level |

## Submission

Commit `challenge-02.md` to your portfolio under `c38-week-04/challenge-02/`.
