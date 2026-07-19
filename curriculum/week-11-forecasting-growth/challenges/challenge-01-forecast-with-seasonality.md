# Challenge 1 — Forecast Revenue with Seasonality

**Goal:** Decide, with evidence and honesty about its limits, whether and how to bake a suspected seasonal pattern into a forecast you only have one year of data to support.

**Estimated time:** 60 minutes.

## The situation

Crunch Flow's `new_mrr` dipped hard in August 2025 — $2,600, down from June's $4,000 and July's $3,300, before rebounding to $4,300 in September. The linear-trend fit from Lecture 3 gives August a residual of roughly **−$1,875**, its worst miss of the year by a wide margin. The obvious read: **summer slowdown** — a well-known pattern in B2B SaaS, where buyer decision cycles slow down in July/August as budget-holders take vacation and deals stall.

Finance has heard this explanation before and wants it in the forecast: *"Just knock August 2026 down like last August was down, and adjust the rest of the year to match."* Your job this week isn't to say yes or no on faith — it's to work out what the data can and can't tell you, and build a forecast that reflects the honest answer.

## The core tension

You have **exactly one year of monthly data.** A repeating seasonal pattern, by definition, is something that recurs across *multiple* cycles — you cannot statistically distinguish "August is always weak" from "August 2025 happened to be weak for a one-off reason" (a sales rep on leave, a delayed renewal, a single lost deal) with only one observation of "August." This is the central judgment call of the challenge: **how do you responsibly use a signal you can't yet prove is real?**

## Tasks

1. **Quantify the anomaly.** Using the linear trend fit from Lecture 3 (`ending_mrr` vs. month index), compute how far August's actual `ending_mrr` sits below its fitted value, in dollars and as a percentage of the fitted value. Do the same for `new_mrr` against its own trend line. Which measure shows a bigger deviation, and does that make sense given `new_mrr` is one input that feeds into `ending_mrr` alongside three other bridge components?

2. **Build the case *for* treating it as seasonal.** List every piece of evidence in the data (and in the business context given in the README) that supports "this is a real, repeating pattern" — not just the one dip, but anything else consistent with a summer-slowdown story (e.g., where else in the year does the data show a rebound-after-dip shape, and does that fit a "budget cycle" narrative or contradict it?).

3. **Build the case *against*.** List every reason a rigorous analyst should be skeptical of calling this "seasonal" off one data point. What would a single unusual event (one lost deal, one rep's parental leave, one delayed contract signature) look like in this data, and how would you distinguish that from a true seasonal pattern using only what you have?

4. **Decide on a policy — and write it down as a rule, not a one-off tweak.** Choose one of (a) apply no seasonal adjustment at all and let the trend/rate-based forecast stand, (b) apply a **partial, explicitly-dampened** adjustment (e.g., discount the observed dip by 50% to reflect your uncertainty that it will repeat), or (c) apply the full observed dip. Whichever you choose, state it as a rule you could apply consistently (e.g., "August 2026's `new_mrr` growth assumption is reduced by X% relative to the base-case trend value, because…") — not as a manually-typed override for one month only.

5. **Apply it and measure the impact.** Rebuild the base-case bridge forecast (Exercise 3's method) with your Task 4 policy applied to August 2026 (and, if your policy affects it, July 2026 too). Report the new Q3 2026 ending MRR next to the unadjusted base case from Exercise 3. How much did your policy move the number?

6. **Say what would resolve the uncertainty.** What specific additional data — not more analysis of the same twelve months, but genuinely new data — would let you confidently confirm or reject the seasonal hypothesis? How many more months would you need, at minimum, to see a *second* August and call the pattern confirmed?

## What a strong submission looks like

- Task 1's numbers are computed, not eyeballed, and the "does that make sense" question is actually answered with reasoning about how the bridge components interact.
- Tasks 2 and 3 are genuinely balanced — a submission that only builds the case *for* seasonality (because that's the "interesting" answer) or only argues against it (because that's the "safe" answer) hasn't done the job. The tension is the point.
- Task 4's policy is a **rule**, stated in a sentence a colleague could apply to a different month without asking you what you meant.
- Task 5 shows the actual before/after numbers — not just "it went down."
- Task 6 names a concrete, falsifiable condition ("if August 2026's actual new_mrr comes in within 10% of trend, treat this as a one-off and drop the adjustment for August 2027"), not a vague "we'll know more with time."

## Submission

`challenge-01.md` with your reasoning for each task, plus the queries/code you ran, in your portfolio under `c38-week-11/challenge-01/`.
