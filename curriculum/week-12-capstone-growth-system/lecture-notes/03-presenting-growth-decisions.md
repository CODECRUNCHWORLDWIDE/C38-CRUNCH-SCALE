# Lecture 3 — Presenting Growth Decisions

> **Duration:** ~2 hours. **Outcome:** You can forecast Crunch Boards' revenue with at least two independent methods, explain why they disagree, and assemble the funnel, unit economics, experiment read, and forecast into one decision narrative where every sentence is one query away from proof.

This lecture closes the loop the other eleven weeks opened. It builds the forecast, then it teaches the one skill this whole course has been quietly building toward and never named directly: **turning a pile of correct numbers into a decision someone will actually act on.**

## 1. Three forecasts, same six months of data

Crunch Boards' observed monthly MRR (from `fct_mrr_monthly`, blended across channels):

```
Jan: 256.00   Feb: 599.00   Mar: 826.00   Apr: 954.00   May: 1253.00   Jun: 1878.00
```

Growth isn't smooth — the month-over-month dollar increases are 343, 227, 128, 299, 625. June's jump is nearly double any prior month. A forecast built on this series has to make a call about whether June was the new trend or a one-off spike, and different methods make that call differently without ever saying so out loud. Build three, side by side, so the disagreement is visible instead of hidden inside a single number.

**Method A — linear regression (least squares trend line).** Fits a straight line through all six months, treating every month equally:

```python
import numpy as np

months_idx = np.array([0, 1, 2, 3, 4, 5])
mrr = np.array([256.0, 599.0, 826.0, 954.0, 1253.0, 1878.0])

slope, intercept = np.polyfit(months_idx, mrr, 1)
print(f"slope={slope:.2f}/month  intercept={intercept:.2f}")

for i, name in enumerate(['Jul', 'Aug', 'Sep']):
    x = 6 + i
    print(name, round(slope * x + intercept, 2))
```

```
slope=291.43/month  intercept=232.43
Jul  1981.00
Aug  2272.43
Sep  2563.86
```

**Method B — moving-average dollar growth.** Averages the last **three** month-over-month dollar changes and extends that flat rate forward — deliberately more reactive to recent momentum than Method A:

```python
deltas = [mrr[i] - mrr[i-1] for i in range(1, 6)]     # [343, 227, 128, 299, 625]
avg_delta = sum(deltas[-3:]) / 3                        # last 3: 128, 299, 625
print("avg $ growth (last 3mo):", round(avg_delta, 2))

val = mrr[-1]
for name in ['Jul', 'Aug', 'Sep']:
    val += avg_delta
    print(name, round(val, 2))
```

```
avg $ growth (last 3mo): 350.67
Jul  2228.67
Aug  2579.33
Sep  2930.00
```

**Method C — compounding percentage growth.** Averages the last three month-over-month *percentage* changes and compounds them forward — the most aggressive of the three, because it assumes the *rate* keeps accelerating in dollar terms even as it stays flat in percentage terms:

```python
growth_rates = [mrr[i]/mrr[i-1] - 1 for i in range(1, 6)]   # includes the +49.9% June jump
avg_growth = sum(growth_rates[-3:]) / 3
print("avg %% growth (last 3mo):", round(avg_growth * 100, 2), "%")

val = mrr[-1]
for name in ['Jul', 'Aug', 'Sep']:
    val *= (1 + avg_growth)
    print(name, round(val, 2))
```

```
avg % growth (last 3mo): 32.24 %
Jul  2483.46
Aug  3284.11
Sep  4342.89
```

## 2. Reading the disagreement, not hiding it

| Month | Method A (linear) | Method B (moving avg $) | Method C (compounding %) | Spread |
|---|---:|---:|---:|---:|
| Jul 2025 | $1,981.00 | $2,228.67 | $2,483.46 | $502 |
| Aug 2025 | $2,272.43 | $2,579.33 | $3,284.11 | $1,012 |
| Sep 2025 | $2,563.86 | $2,930.00 | $4,342.89 | $1,779 |

By September the three methods span **$2,563.86 to $4,342.89** — a $1.8k gap, or roughly **70% relative disagreement**, on the exact same six months of input data. That is not a bug in one of the methods. It's the direct, mechanical consequence of what each one assumes:

- **Method A (linear)** assumes growth adds a **constant number of dollars** every month. It's the most conservative because it lets June's outsized jump pull the whole trend line up only a little — one data point among six.
- **Method B (moving average $)** assumes the **recent trailing dollar pace** (weighted toward the last three months, which includes June's spike) continues flat. More reactive than A, still additive.
- **Method C (compounding %)** assumes the **recent trailing percentage pace** continues, compounding — the same growth *rate* applied to an ever-larger base produces ever-larger dollar gains. This is the method most exposed to June being a one-off: if June's 49.9% jump was a fluke (a large Scale-plan customer converting, say), Method C bakes that fluke into every future month at compounding scale.

**The forecast you present is not "the model's number" — it's the number you chose, for a reason you can state.** For Crunch Boards this quarter, the defensible choice is **Method B, with Method A quoted as the floor and Method C flagged as upside-if-June-repeats-but-not-a-plan.** Method A is too conservative — it barely reacts to real, recent acceleration. Method C is too exposed to a single unusually large month. Method B splits the difference using the same trailing-three-months window as C, but in dollar terms, which don't compound a possible fluke. That reasoning — not the arithmetic — is the actual content of a forecast slide. State it explicitly in your own capstone; "the forecast is $2,930 by September" with no stated method or reasoning is not a forecast, it's a guess with a decimal point.

## 3. From honest numbers to an actual recommendation

Lecture 2 left you with three findings that don't resolve themselves:

1. `paid_search` is structurally underwater (LTV:CAC 0.08, no visible retention recovery).
2. `referral` clears LTV:CAC comfortably (2.47:1) but on a small, young sample.
3. `onboarding_checklist_v2` shows a 25-point activation lift that is **not** statistically significant (p ≈ 0.132, n=24).

A stakeholder does not want three separate footnotes. They want one paragraph that turns all three into what happens next. Here is what that paragraph looks like, built entirely from numbers you can re-run:

> **Recommendation: shift next quarter's `paid_search` budget toward `referral`, and extend the `onboarding_checklist_v2` test before generalizing it.**
> `paid_search` costs $1,963.64 to acquire a customer worth $165.93 in five months of observed margin (LTV:CAC 0.08) — every dollar spent there currently returns eight cents, with no sign of the retention curve recovering (Lecture 2, Section 4). `referral` costs $225 per customer and returns $2.47 for every dollar spent, but on only 12 customers, the oldest just five months old — real, but young evidence. Recommend moving half of `paid_search`'s $3,600/month budget to `referral`'s program (bonus payouts scale with volume, so this doubles referral capacity without a new fixed line item), while holding `organic` flat pending its own retention curve maturing past the current 4-month window. Separately: `onboarding_checklist_v2`'s 25-point activation lift is promising but underpowered at n=24 (p≈0.132) — recommend extending the test to the next two full cohorts (~24 more users) before rolling the checklist out to 100% of signups, rather than shipping on today's evidence alone. Revenue forecast for Q3: $2,229–$2,930/month by September under the moving-average method (Section 1), with the linear method ($2,564) as the conservative floor if the `referral` shift takes a month to ramp.

Notice what that paragraph does: **every clause names a number, and every number names the table it came from.** "LTV:CAC 0.08" points at `fct_unit_economics`. "p≈0.132, n=24" points at `fct_experiment_results`. "$2,229–$2,930" points at the forecast comparison in Section 2. A stakeholder — or a skeptical instructor — can ask "where does that come from" about any single sentence and get a `SELECT` in response, not a shrug. **That traceability is the actual deliverable of a growth capstone.** The dashboard is not the deliverable. The paragraph is.

## 4. Anatomy of a decision narrative

Every growth recommendation this course has taught you to build shares the same four-part shape. Use it for the mini-project's final report, and for every growth memo you'll ever write after this course:

1. **The ask, in one sentence.** What do you want the reader to approve or do? (Shift budget. Extend a test. Hold a channel flat.)
2. **The evidence, each claim traced to a table.** Funnel, unit economics, experiment, forecast — in whatever order best supports the ask, not necessarily the order you built them.
3. **The uncertainty, stated plainly.** Where's the sample thin? Where's a formula's assumption broken (Lecture 2, Section 4)? Where do your own forecast methods disagree, and by how much? Hiding this doesn't make the recommendation stronger — it makes it fragile the first time someone asks a follow-up question.
4. **The next checkpoint.** What would change your mind? What number, checked again in a month, confirms or kills the recommendation? ("If `referral`'s next 12 customers show any cancellations, revisit the LTV before scaling further" is a checkpoint. "We'll see how it goes" is not.)

A deck, a memo, a Slack message, or a live walkthrough of your SQL — the medium doesn't matter. The four parts do.

## 5. The same numbers, two audiences

The same four-part narrative changes shape, not substance, depending on who's reading it. Crunch Boards' recommendation from Section 3 looks different in front of a founder than in front of the engineer who'll actually implement the checklist rollout:

**For a founder (decision-focused):** lead with the ask and the dollar stakes, keep evidence to the two or three numbers that actually move the decision, and put every supporting query in an appendix rather than the body. "Shift $1,800/month from `paid_search` to `referral`; hold the checklist rollout at 50% for one more cohort" is the first sentence, not the last one. A founder reading past sentence three without the ask already stated has already stopped reading.

**For an engineer or a fellow analyst (mechanism-focused):** lead with the query and the exact filter conditions, because their first question will be "how did you compute that" and "does it hold up under X edge case" — the same questions this course has trained you to answer, but now they're the *first* thing asked, not a caveat buried in paragraph four. Showing `fct_unit_economics`'s CTEs and the `stg_subscriptions` cutoff filter earns credibility with this audience faster than the recommendation itself does.

**The number of underlying facts never changes between the two versions — only the order and the depth of the trace.** A common capstone mistake is writing one document and hoping it serves both audiences; it usually serves neither well, because a founder skims past your CTEs looking for the ask, and an engineer distrusts a recommendation that never shows its work. The mini-project asks for one `decision-memo.md` — write it for the founder audience, but structure it so an engineer could reconstruct every number from the `metrics.md` and SQL you also ship alongside it. That split (narrative document + traceable technical artifacts) is exactly how real growth teams organize a quarterly review.

## 6. Three ways this exact lecture goes wrong in practice

Three failure modes show up disproportionately often when growth analysts (not just capstone students) move from "correct numbers" to "presented decision." Naming them here is cheaper than discovering them the hard way in your own mini-project:

1. **Precision theater.** Reporting "$2,930.00" instead of "roughly $2.2k–$2.9k, base case ~$2.9k" implies a level of certainty the underlying methods don't have — three methods disagreeing by 70% is not a situation that produces a two-decimal-place answer honestly. Round to the precision your uncertainty actually supports, and say the range out loud.
2. **Burying the lead in a wall of caveats.** The opposite failure: hedging every sentence so thoroughly that the actual recommendation never surfaces. Section 4's four-part shape exists specifically so uncertainty gets its own labeled section — stated once, clearly — instead of qualifying every other sentence into mush. State the ask cleanly first; state the uncertainty cleanly second; don't interleave them until the reader can't find either.
3. **Treating the forecast method choice as a formality.** "We used a linear regression because that's what we learned" is not a reason — Section 2 showed the actual reasoning is about which assumption (constant-dollar growth vs. trailing-pace vs. compounding-rate) best fits *this specific* company's actual trajectory this quarter. A capstone that runs one method because it's the first one taught, without weighing it against the alternatives the way Section 2 did, hasn't actually forecasted anything — it's plugged numbers into a formula.

## 7. Check yourself

- Why do the three forecast methods diverge by roughly $1,800 (70%) by September despite using identical input data?
- In one sentence each, what does Method A assume that Method C doesn't, and vice versa?
- Which forecast method did this lecture recommend for Crunch Boards, and what specific reasoning (not just "it's in the middle") justified that choice?
- Rewrite this sentence to be traceable: "Referral is a great channel, we should do more of it." What number and what table does it need?
- What are the four parts of a decision narrative, and which one is most often skipped by people newer to growth analytics — and why is skipping it the one that gets a recommendation reversed later?
- The recommendation in Section 3 both shifts budget *and* asks for more experiment data before a full rollout. Why is holding two different confidence levels in one recommendation more honest than forcing everything into a single "yes, do it" or "no, don't"?

If those are automatic, you're ready for the exercises, challenges, and mini-project — which is where you build every mart and every piece of this narrative yourself, top to bottom, on `crunch_boards`.

## Further reading

- **PostgreSQL — Window Functions (for trailing-window forecasts):** <https://www.postgresql.org/docs/current/tutorial-window.html>
- **NumPy — `polyfit`:** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
- **pandas — Time series / date functionality:** <https://pandas.pydata.org/docs/user_guide/timeseries.html>
- **Bessemer Venture Partners — State of the Cloud** (forecast/benchmark framing used by real growth teams): <https://www.bvp.com/atlas/state-of-the-cloud>
