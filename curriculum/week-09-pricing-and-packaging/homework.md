# Week 9 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on SQL/pandas, a written explanation, and a build-your-own-dataset task. Commit each.

All queries run against `wtp_survey`, `accounts_usage`, `price_experiment`, and `subscription_base` from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Rebuild every PSM curve value by hand, twice (60 min)

Confidence in the Van Westendorp method comes from doing the arithmetic without a query doing it for you.

1. Pick any five prices between $10 and $150 of your choosing (not the ones used in the lecture). For each, compute all four cumulative percentages (TC, Bargain, Getting Expensive, TE) two ways: (a) a SQL query against `wtp_survey`, (b) counting by hand from a printed/sorted list of the four columns.
2. Confirm your two methods agree exactly for all five prices, all four curves (20 numbers total).
3. Using only your five hand-computed points (no full price grid), estimate — by eye, with linear interpolation between your nearest two points — where TC and TE would cross. How close is your rough estimate to the exact OPP ($60.50) from Lecture 1?

**Deliver** `psm-by-hand.md` with your five prices, both sets of 20 numbers, and your OPP estimate with the interpolation shown.

---

## Problem 2 — Twenty warm-up queries (75 min)

Write and run each against the Week 9 seed. Put them in `warmups.sql` with a `-- N` comment and the answer beneath each.

1. All `agency`-segment respondents whose `too_cheap` is above $80.
2. The single respondent with the highest `too_expensive` value, and their segment.
3. Average `cheap` (bargain) value, by segment.
4. Every `accounts_usage` row with `mtu` between 10,000 and 50,000.
5. The three accounts with the lowest `mtu`, and their segment.
6. Count of accounts per segment where `mtu` exceeds their segment's own average `mtu`.
7. `price_experiment` rows where the conversion rate exceeds 15%.
8. The single highest-revenue row in `price_experiment` (`price * conversions`), computed in the query itself.
9. Total customers across all three tiers in `subscription_base`.
10. `subscription_base` MRR per tier (`customer_count * current_price`), ordered highest to lowest.
11. Every `wtp_survey` respondent whose `expensive` value is more than double their `too_cheap` value.
12. The `team`-segment respondent with the narrowest gap between `too_cheap` and `too_expensive` (most price-sensitive individual in that segment).
13. `accounts_usage` accounts joined to `wtp_survey` where `mtu` is in the top 5 overall, with their `too_expensive` value alongside.
14. Every `Growth`-eligible account (`mtu <= 30000`) that is currently `agency`-segment, if any. *(Trick question — check your understanding of why this should return 0 rows.)*
15. The average `too_expensive` across the entire `wtp_survey` table (all 30, blended) — compare it by eye to the blended PME from Lecture 1 ($68). Are they the same number? Should they be?
16. `subscription_base`'s combined `monthly_signups` across all three tiers.
17. Every `indie`-segment account with `mtu` above 2,000 (candidates closest to the `Starter` ceiling).
18. The dollar gap between the highest and lowest `current_price` in `accounts_usage`. *(Trick question — what does the answer tell you about the current flat-pricing scheme?)*
19. Count of `price_experiment` rows where `conversions / quotes_shown` is below 10%.
20. Every `team`-segment respondent whose `cheap` value exceeds $50.

---

## Problem 3 — Explain elasticity in plain English (45 min)

In `elasticity-writeup.md`, answer in prose (no more than 400 words total):

1. A colleague says "demand went down 10% when we raised the price 10%, so elasticity is bad news, full stop." Is their arithmetic right? Is their conclusion right? Explain using the exact definition of elasticity from Lecture 3 §2.
2. Explain, without using the word "elastic" or "inelastic" once, what it means for a 1% price increase to lose more than 1% of your customers versus less than 1%.
3. Why does the revenue-maximizing price sit exactly where `|E| = 1`? You don't need calculus for the explanation — reason about what happens to revenue on either side of that point.
4. Give a real-world example (not ScopeIQ) of a product you'd expect to have inelastic demand, and one you'd expect to have elastic demand. Justify each in one sentence.

---

## Problem 4 — Design your own Van Westendorp survey (60 min)

Practice the full instrument, not just the analysis, on a product you actually understand.

1. Pick a real (or realistic) product or service you could plausibly price — an app, a freelance service, a subscription box, anything with a $5–$500 range.
2. Write the exact four PSM questions for that product, adapted from Lecture 1 §2's wording.
3. "Survey" 8 people you know (friends, classmates, family — this doesn't need IRB approval, just real answers) with your four questions. If you can't reach 8 people, you may invent 8 *plausible* respondents instead — note clearly which you did.
4. Load their answers into a table (`CREATE TABLE my_survey (...)`, matching `wtp_survey`'s shape) and compute the blended PMC, PME, and OPP.
5. Write two sentences: does the OPP feel right to you, given what you know about the product and the people you asked? If not, what do you think went wrong — a bad question, too few respondents, or a genuinely surprising result?

**Deliver** `my-survey.sql` (the `CREATE TABLE`, the inserts, and your PMC/PME/OPP queries) and `my-survey-writeup.md` (steps 1–2 and 5).

---

## Problem 5 — Stress-test the mini-project's riskiest assumption (60 min)

Pick **one** assumption from the mini-project model (the 25% discount take-rate, the 3% monthly expansion rate, or the −0.35 existing-customer churn elasticity for `Growth`) and sensitivity-test it.

1. Recompute the mini-project's headline net-impact number (scenario 4 vs. scenario 2) at three values of your chosen assumption: the value used in the mini-project, one meaningfully lower, and one meaningfully higher (e.g., for the discount take-rate: 10%, 25%, 40%).
2. Plot or tabulate how the headline $ and % impact change across your three scenarios.
3. In two sentences, state whether the recommendation (ship the price increase) would change at your "meaningfully lower" value — i.e., is the recommendation robust to being wrong about this one number, or does it flip?

**Deliver** `sensitivity-test.py` and a short `sensitivity-writeup.md` with your table and your two-sentence conclusion.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 60 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
