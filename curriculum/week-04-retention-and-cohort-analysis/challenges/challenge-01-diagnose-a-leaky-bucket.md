# Challenge 1 — Diagnose a Leaky-Bucket Product

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

You're the analyst at Crunch. The VP of Growth pulls up the top-line unbounded retention triangle from Lecture 1 and is thrilled: month-1 retention climbed from 16.7% (Feb cohort) to 66.7% (May cohort) — a **4×** improvement in three months. She wants to tell the board "retention is fixed." Your job, before that slide goes out, is to find out whether it's *actually* fixed, or whether the headline number is hiding a bucket that's still leaking somewhere underneath it.

You already know two things the VP's slide doesn't show: the product has a `self_serve` segment (36 of 48 users — the majority) and a `sales_assisted` segment (12 of 48). And an onboarding change shipped 2025-04-01, splitting the four cohorts into a "pre" era (Feb, Mar) and a "post" era (Apr, May).

## Your task

Write `challenge-01.md` containing:

1. **A segment-level retention breakdown** — rebuild the triangle (or the eligible-adjusted survival curve from Lecture 3) split by `segment`. Show your SQL.
2. **A finding, stated in one paragraph**: is the "4× improvement" real for both segments, or is it concentrated in one of them? Is there a segment where the bucket is still leaking regardless of the onboarding change?
3. **A quantified answer**: pick the single number you'd put on a slide instead of (or alongside) the blended 16.7%→66.7% figure, and justify why it's more honest.
4. **A next investigation**: name one thing you don't have in this dataset that you'd want before recommending a fix (e.g., a specific onboarding step, time-to-first-value, support ticket volume) — and why that's the natural next question once you've located *which* segment is leaking.

## Constraints

- Use both `users` and `events`. You may reuse and adapt SQL from the lectures, but the segment split must be your own query, not a copy-paste of the blended one.
- State your assumptions where the data is ambiguous (e.g., how you're treating a segment/cohort pair with a very small sample).
- Every claim about "improvement" or "leak" needs a number attached to it — no adjectives without evidence.

## Hints

<details>
<summary>On where to start</summary>

Start from the segment-level survival table built in Lecture 3 §2 — it's already eligible-adjusted, so you won't accidentally divide by cohorts too young to have reached a given month. Look at `self_serve` alone: does its month-1 retention rise the same way the blended number does? Compare that rise to `sales_assisted`'s.

</details>

<details>
<summary>On small samples</summary>

By month 3–4, some segment/cohort combinations have single-digit eligible users. A jump from 77.8% to 33.3% between month 2 and month 3 in a 6-to-9-person group can be one or two people's behavior, not a trend. Say so explicitly if you lean on a late-month cell for your argument — don't build your headline finding on the noisiest part of the curve.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| SQL | Reuses the blended triangle unchanged | Builds a genuine segment-level breakdown |
| Finding | Repeats the VP's 4× claim at face value | Locates exactly which segment the improvement is (and isn't) concentrated in |
| Small-sample awareness | Reports every percentage with equal confidence | Flags where n is too small to trust |
| Recommendation | "Retention improved, good job" | Names a specific, falsifiable next question |

## Submission

Commit `challenge-01.md` to your portfolio under `c38-week-04/challenge-01/`.
