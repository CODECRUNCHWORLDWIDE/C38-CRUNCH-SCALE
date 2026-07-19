# Challenge 2 — Catch a Broken Test

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer, but four specific bugs to find.**

## The scenario

LoopCart's growth team posts this update in the company's #wins channel. Your job: read it as carefully as you'd want someone to read *your* test before you shipped anything based on it.

> **🎉 One-Click Reorder is a win — shipping to 100%**
>
> We tested a "Reorder" button on the order-history page for returning customers: one tap re-adds their last order to the cart, no need to search for items again. We announced it to our whole customer list with a "New! Try One-Click Reorder" email the morning the test went live, and started tracking reorder rate (reorder-completions ÷ order-history page views) immediately.
>
> We checked the numbers each morning. By **Wednesday** (day 3 of a planned 2-week test), the lift was already significant, so we called it and pulled the trigger on a full rollout rather than block engineering for another week and a half.
>
> | Variant | Sessions | Reorder completions | Reorder rate |
> |---|---:|---:|---:|
> | Control (old flow) | 1,200 | 168 | 14.00% |
> | Treatment (new button) | 850 | 150 | 17.65% |
>
> z = 2.247, **p = 0.025** — significant at the standard 0.05 bar. Nice lift, and worth noting: our target was a 50/50 split, so a ~59/41 split like this is a little uneven but nothing to worry about at this scale.
>
> We also had eyes on 5 other metrics out of curiosity while we were in there — time-to-reorder, cart size, session length, NPS, and support ticket volume — and **cart size also came back significant (p = 0.04)**, so we're counting that as a bonus finding: One-Click Reorder appears to increase basket size too. Great week for the team.

## Your task

Produce `challenge-02.md` identifying **all four** problems baked into this update — some are stated outright in the numbers, some are implied by how the test was run. For each one:

1. **Name the trap** (use the vocabulary from Lecture 3).
2. **Show the specific evidence** in the memo that reveals it — quote the number or the sentence.
3. **Compute whatever's computable.** At least two of the four problems have a real number you can calculate from data given in the memo (you don't need more data than what's here for those two).
4. **Rate its severity** — is this "the whole result is unusable, full stop," or "a real concern, but here's how a team could still salvage a defensible conclusion"? Defend the rating.

Then write a closing paragraph: if LoopCart's team came to you *before* shipping and asked "can we launch on this?", what's your actual recommendation, and what's the minimum additional work you'd want before revisiting the launch decision?

## Constraints

- Find **four** distinct problems — don't pad the list with the same issue described two ways, and don't stop at the first one you spot (the 59/41 split sentence in the memo is a strong hint, not the whole answer).
- At least one of your four write-ups must include an actual computed number, not just a description (Lecture 3 gave you the exact formula for at least two of the problems present here).
- Don't just say "this test is bad" — for each problem, state what specifically would need to change about *how the test was run* (not just "run it again") to avoid it next time.

## Hints

<details>
<summary>On the uneven split</summary>

The memo dismisses "a little uneven" with no math. Run the actual SRM chi-square (Exercise 3's formula) on 1,200 vs. 850. Compare the result to the 3.841 critical value from Lecture 3, section 2 — is "a little uneven" an accurate description of what you get?

</details>

<details>
<summary>On the multiple metrics</summary>

Six metrics were checked (reorder rate, plus the 5 "out of curiosity" ones) with no stated correction. What's the real chance of at least one false positive across 6 uncorrected checks (Lecture 3, section 4)? Does cart size's p = 0.04 survive a Bonferroni-corrected threshold for 6 comparisons?

</details>

<details>
<summary>On the email blast</summary>

The announcement email went to the "whole customer list," not split by variant, the same morning the test launched. Think through what that does to (a) control-group behavior — could a control-group customer, now aware a reorder button exists, behave differently even without seeing it? — and (b) the shape of the traffic in the first few days specifically. This one connects to two different lecture concepts at once; a strong answer names both.

</details>

## How success is judged

| Signal | Weak submission | Strong submission |
|--------|------------------|--------------------|
| Coverage | Finds 1–2 of the 4 problems | Finds all 4, each correctly named |
| Evidence | Vague ("this seems off") | Quotes the specific number or sentence that reveals each problem |
| Computation | No numbers computed | Correctly computes the SRM chi-square and/or the multiple-comparisons math |
| Severity judgment | Treats every problem as equally fatal (or equally minor) | Distinguishes which problems make the result unusable vs. which are real but survivable with more rigor |
| Recommendation | "Don't ship" with no path forward | Concrete next steps: what to fix, what to re-run, what would need to be true to revisit the launch call |

## Submission

Commit `challenge-02.md` to your portfolio under `c38-week-08/challenge-02/`.
