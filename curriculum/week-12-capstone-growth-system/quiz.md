# Week 12 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before considering this course complete. A mix of multiple-choice and short "what does this mean" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** `raw_subscriptions` has 35 rows; `stg_subscriptions` has 30. What explains the difference?

- A) Five rows are duplicate data-quality errors that staging correctly removed.
- B) Five subscriptions were `created` after the 2025-06-30 analysis cutoff, so `stg_subscriptions` excludes them by design.
- C) `stg_subscriptions` only keeps `active` subscriptions.
- D) `raw_subscriptions` has five orphaned rows with no matching user.

---

**Q2.** Why does the cutoff filter (`WHERE created <= DATE '2025-06-30'`) belong in `stg_subscriptions` rather than in every downstream mart query?

- A) It doesn't matter where it lives, as long as it's somewhere.
- B) Filters run faster in staging than in marts.
- C) Putting it in one place means every downstream mart inherits the correct cutoff automatically, and moving the cutoff later only requires one change.
- D) Raw tables can't have `WHERE` clauses applied to them.

---

**Q3.** Reading `fct_acquisition` by channel: `paid_search` has the most signups (30) but the lowest conversion rate (36.7%); `referral` has the fewest signups (18) but the highest conversion rate (66.7%). What's the correct takeaway?

- A) `paid_search` is definitely the best channel because it has the most signups.
- B) Volume and quality of signups can point in opposite directions, and a channel decision needs both, not just signup count.
- C) `referral`'s numbers must be wrong since they're so much better.
- D) Conversion rate doesn't matter if signup volume is high enough.

---

**Q4.** `referral`'s CAC ($225) is structurally different in *kind*, not just size, from `paid_search`'s ($1,963.64). Why?

- A) `referral` spend is a flat monthly budget just like the other two channels.
- B) `referral` spend scales with signup volume (a per-referral bonus plus a small fixed admin fee), while `paid_search` and `organic` are flat monthly budgets regardless of volume.
- C) `referral` has no marketing spend at all.
- D) There is no real difference — the numbers just happen to be smaller.

---

**Q5.** At a 75% gross margin, `referral`'s reciprocal-formula LTV (`margin ÷ avg monthly churn`) is undefined (division by zero). What is the correct response?

- A) Report the LTV as literally infinite in `metrics.md`.
- B) Conclude `referral` customers never churn, ever.
- C) Report the cohort-sum LTV ($555.60) instead, and flag that the reciprocal formula doesn't apply yet because zero churn has been *observed* in a small (12-customer), young sample — not because referral customers are immortal.
- D) Throw out `referral`'s LTV entirely since one formula failed.

---

**Q6.** `organic`'s retention drops from 83.3% at age 3 to 33.3% at age 4. Why should this specific data point be treated with more caution than `paid_search`'s consistent age-2 drop (77.8% → 44.4%)?

- A) It shouldn't — both drops are equally trustworthy.
- B) The age-4 `organic` point is built on only 3 observed customers, so one cancellation swings the percentage dramatically; `paid_search`'s drop is backed by 9 customers and repeats at later ages too.
- C) `organic` customers are inherently less predictable than `paid_search` customers.
- D) Age-4 data is always wrong regardless of sample size.

---

**Q7.** `onboarding_checklist_v2` shows control at 66.7% activation and treatment at 91.7% (n=12 per arm), with a two-proportion z-test giving p≈0.132. What's the correct one-sentence read?

- A) The checklist clearly works — ship it to 100% of new signups immediately.
- B) The checklist clearly doesn't work — the lift is fake.
- C) The lift is promising and directionally consistent but not statistically significant at α=0.05 given the small sample — more evidence is needed before generalizing.
- D) p≈0.132 means there's a 13.2% chance the checklist doesn't work.

---

**Q8.** Why is (D) in Q7 a common but incorrect way to describe a p-value?

- A) p-values apply to the sample mean, not the treatment effect.
- B) A p-value is the probability of seeing a result at least this extreme *if there were truly no effect* — it is not the probability that the treatment "doesn't work" or that the null hypothesis is true.
- C) p-values only apply to continuous data, not proportions.
- D) There's no meaningful difference between the two interpretations.

---

**Q9.** Three forecast methods project Crunch Boards' September 2025 MRR at $2,563.86 (linear), $2,930.00 (moving-average $), and $4,342.89 (compounding %) — a roughly 70% spread. What causes the disagreement?

- A) One of the three methods must contain a calculation bug.
- B) Each method makes a different assumption about how growth continues (constant dollar addition, recent trailing dollar pace, or recent trailing percentage pace compounding) — the same six months of input data, run through different assumptions, mechanically produces different projections.
- C) The dataset is too small for any forecast to be valid, so all three numbers are meaningless.
- D) Only the compounding-percentage method is ever correct for SaaS companies.

---

**Q10.** Why is the compounding-percentage forecast method (Method C) the most exposed to June 2025's unusually large MRR jump?

- A) It isn't more exposed than the other two methods.
- B) It applies the recent growth *rate* to an ever-larger base every month, so if June's jump was a one-off rather than a new sustained rate, that anomaly gets compounded forward instead of averaged out.
- C) Method C ignores June's data entirely.
- D) Method C only uses January's data.

---

**Q11.** What are the four parts of a decision narrative, in order, as taught in Lecture 3?

- A) Data, methodology, results, conclusion.
- B) The ask, the evidence (traced to tables), the uncertainty, the checkpoint.
- C) Executive summary, appendix, charts, next steps.
- D) Problem, hypothesis, experiment, p-value.

---

**Q12.** Why does a decision memo that hides `referral`'s small sample size or the experiment's non-significant p-value get graded *down*, not up, even if it makes the recommendation sound more confident?

- A) Graders prefer longer memos.
- B) A memo that only survives because no one asks a follow-up question isn't defensible — the whole point of building the numbers in SQL is being able to answer the follow-up question honestly, not to avoid it.
- C) Non-significant results should never be mentioned in any business document.
- D) It doesn't matter — confidence is what stakeholders actually want.

---

**Q13.** `dim_user.in_experiment` flags the 24 users in `onboarding_checklist_v2` (the May and June cohorts). Why does `fct_acquisition`'s channel-level activation rate calculation need to be aware of this flag conceptually, even if the column itself isn't filtered out?

- A) It doesn't matter at all — experiment membership is unrelated to channel-level funnel numbers.
- B) An experiment cell (control/treatment) is a different kind of grouping than a channel segment, and conflating "this user got a UI treatment" with "this user came from this channel" risks misattributing an experiment effect to a channel effect, or vice versa.
- C) Users in the experiment should be deleted from all channel-level marts.
- D) `in_experiment` should always be `TRUE` for every user.

---

**Q14.** A capstone stress test drops the gross-margin assumption from 75% to 60% and finds `referral`'s LTV:CAC falls from 2.47 to 1.98 — still above 1:1, but a real, non-trivial move. What's the correct way to describe this result?

- A) The margin assumption doesn't matter since the conclusion ("referral clears breakeven") didn't change.
- B) The recommendation to favor `referral` is robust to this specific margin range, but the exact confidence level (comfortably above 3:1 vs. hovering closer to marginal) does shift meaningfully — both facts are worth stating, not just the first one.
- C) A 60% margin assumption is always more realistic than 75%, so the higher number should be discarded.
- D) Stress tests are only useful when they flip the recommendation entirely.

---

**Q15.** Why does this capstone use a smaller, "messier" seed dataset (72 signups, some channels with fewer than 15 paying customers) instead of a larger, cleaner one like earlier weeks' seeds?

- A) Smaller datasets are always easier to grade.
- B) A smaller, more realistic sample forces you to apply the small-sample caution (thin-`n` flags, undefined formulas, wide forecast disagreement) that earlier weeks taught in the abstract to genuinely ambiguous real data — which is closer to what an early-stage company's actual warehouse looks like.
- C) Larger datasets would be too expensive to seed.
- D) The dataset size has no pedagogical purpose — it's arbitrary.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — five subscriptions (users 64, 66, 70, 71, 72) were created in early July, after the 2025-06-30 cutoff, so `stg_subscriptions` excludes them by design — a deliberate right-censoring example, not a data quality bug.
2. **C** — a filter defined once in staging propagates to every downstream mart automatically; if the cutoff ever moves, one line changes instead of a hunt through every mart query.
3. **B** — `paid_search`'s volume and `referral`'s conversion quality point in opposite directions; a real channel decision (Lecture 2/3) needs both signup volume and conversion/unit-economics quality, not just one.
4. **B** — `referral`'s cost is a $50-per-referral bonus plus a small fixed admin fee (scales with volume), while `paid_search`/`organic` are flat monthly budgets regardless of volume — a structurally different cost model, not just a smaller number.
5. **C** — the cohort-sum LTV is the defensible, evidence-based number; the reciprocal formula's assumption (a nonzero observed churn rate) is violated by a small, young sample with zero cancellations so far, which is a sample-size fact, not a claim about permanent retention.
6. **B** — 3 customers observed at organic's age 4 means one cancellation swings the rate by ~33 points; `paid_search`'s drop is backed by 9 customers and persists at every later age, which is a real signal, not noise.
7. **C** — promising, directionally consistent, but not significant at conventional thresholds given the small sample (n=12/arm) — more evidence is needed before generalizing to a full rollout.
8. **B** — a p-value is P(data this extreme | null hypothesis true), not the probability the null (or the treatment's ineffectiveness) is true — a very common and easy-to-make misinterpretation.
9. **B** — same six months of MRR data, three different structural assumptions about how growth continues (constant $, trailing $ pace, compounding % pace) mechanically produce three different projections; none of the three needs a bug to explain the spread.
10. **B** — Method C compounds the recent percentage growth rate onto an ever-larger base every month, so an anomalous month (like June) gets amplified forward instead of averaged into a flatter trend the way Method A's full-series linear fit does.
11. **B** — the ask, the evidence (each claim traced to a table), the uncertainty (thin samples, broken formula assumptions, forecast disagreement), and the checkpoint (what would confirm or kill the recommendation).
12. **B** — the entire premise of this course is defensible numbers, meaning numbers that survive a follow-up question; hiding uncertainty to sound more confident produces a memo that only works until someone asks the obvious next question.
13. **B** — an experiment cell (control/treatment) and a channel (paid_search/organic/referral) are two independent groupings of the same users; treating one as if it were the other risks misattributing an effect that actually came from the UI treatment to the acquisition channel, or vice versa.
14. **B** — both facts matter: the directional recommendation survives the stress test, but the margin of confidence (2.47:1 "comfortably healthy" vs. 1.98:1 "still healthy but closer to the 1:1–3:1 marginal band") shifted meaningfully and should be reported, not smoothed over.
15. **B** — a smaller, messier dataset forces genuine application of small-sample caution to real ambiguity, which is closer to what an early-stage company's actual data looks like than a large, clean seed table would be.

</details>

**Scoring:** 12+ → you've completed C38 Crunch Scale. 9–11 → re-read the lecture sections behind your misses before shipping the mini-project. <9 → re-read all three lectures from the top; this capstone's numbers only make sense once the small-sample and traceability discipline from earlier weeks is fully automatic.
