# Week 2 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 3. A mix of multiple-choice and short "what does this return?" — the answer key explains the *why*, not just the letter.

---

**Q1.** In a funnel built from an events table like `funnel_events`, why should you generally use `COUNT(DISTINCT user_id)` rather than `COUNT(*)` per step?

- A) `COUNT(*)` is invalid in a `GROUP BY` query
- B) A user can log more than one event at the same step, which would inflate `COUNT(*)`
- C) `COUNT(DISTINCT user_id)` runs faster
- D) They always return the same number, so it doesn't matter

---

**Q2.** `FIRST_VALUE(users_at_step) OVER (ORDER BY step_order)` in a per-step summary query is used to:

- A) Sort the steps alphabetically
- B) Compute the step-over-previous-step conversion rate
- C) Repeat the top-of-funnel count on every row, as a fixed denominator for step-over-top rates
- D) Remove duplicate steps

---

**Q3.** In this week's seed data, the funnel is 20 → 16 → 13 → 10 across site_visit, signup, activated, and paid_conversion. Which single step-to-step transition loses the highest **percentage** of the entering users?

- A) site_visit → signup (80.0%)
- B) signup → activated (81.3%)
- C) activated → paid_conversion (76.9%)
- D) They all lose the same percentage

---

**Q4.** Why must `LAG() OVER (ORDER BY event_ts)` be paired with `PARTITION BY user_id` in a per-user funnel timing query?

- A) It isn't required; `PARTITION BY` is only cosmetic
- B) Without it, `LAG` looks back across *all* users' events in timestamp order, not just the current user's previous step
- C) `PARTITION BY` is required syntax with no functional effect
- D) `LAG` only works with `PARTITION BY`

---

**Q5.** A conversion window (e.g., "converted within 14 days of first visit") exists to:

- A) Make queries run faster
- B) Prevent a stale, much-later conversion from being credited to a visit that likely didn't cause it, and keep funnels comparable across cohorts
- C) Enforce a database constraint
- D) Convert `NULL` timestamps to real dates

---

**Q6.** `ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY touch_ts ASC)`, filtered to rank 1, computes:

- A) Last-touch attribution
- B) First-touch attribution
- C) Linear attribution
- D) Position-based attribution

---

**Q7.** For a 3-touch conversion path under **linear** attribution, each touch receives:

- A) 100% each
- B) 50%, with the third ignored
- C) ⅓ of the revenue each
- D) 40%, 20%, 40%

---

**Q8.** Under **position-based (40/20/40)** attribution, a conversion with exactly 2 touches is typically handled as:

- A) 100% to the first touch only
- B) 50% to each of the two touches
- C) 40% to each, 20% discarded
- D) Position-based cannot handle 2-touch paths

---

**Q9.** In this week's data, `Google Search - Brand` earns $49 under first-touch but $198 under last-touch for the same two conversions. The most accurate interpretation is:

- A) One of the two numbers must be a bug
- B) Attribution is a modeling choice; the two models reward different things (discovery vs. closing), and the swing itself is a signal worth investigating, not an error
- C) `Google Search - Brand` is a fake channel
- D) Last-touch is always the correct model

---

**Q10.** `SUM(spend_amount) / NULLIF(COUNT(DISTINCT user_id), 0)` in a CAC query, for a channel with zero attributed conversions, returns:

- A) `0`
- B) An error
- C) `NULL` — correctly signaling an undefined ratio rather than a false zero
- D) The total spend, unchanged

---

**Q11.** In Lecture 3, Meta Ads' daily spend rises from $80 to $260 across 10 days while all of its touches land in the first 5 days on spend of $80–$160/day. This pattern is evidence of:

- A) A data entry error
- B) Saturation — spend past a certain point stopped buying additional results in this window
- C) Meta Ads becoming more efficient over time
- D) Nothing meaningful; 10 days is too short to say anything

---

**Q12.** User 18's path is `Google Search - Nonbrand → Meta Ads → Google Search - Brand`, and their last touch (`Google Search - Brand`) gets 100% of the credit under last-touch attribution. Why does Lecture 3 call this a potential case of a channel "reshuffling organic demand"?

- A) `Google Search - Brand` is technically not a real channel
- B) Two other channels did the discovery work earlier in the path; last-touch credits the channel nearest the finish line regardless of what happened upstream
- C) The user should not have been counted as a conversion at all
- D) Brand search always indicates fraud

---

**Q13.** A teammate's query joins `touches` to `conversions` with a plain `JOIN` and sums `revenue` grouped by channel, with no `ROW_NUMBER()` filter or split. What happens to a user with 3 touches before converting?

- A) Their revenue is correctly split three ways
- B) Their full revenue is counted once for each of their 3 touches — inflating the grand total to more than actual revenue
- C) The query fails with a duplicate-row error
- D) Only their first touch is counted

---

**Q14.** Which set of numbers, taken together, is *closest* to what Lecture 3 argues a channel report needs at minimum before a budget decision is made from it?

- A) Total spend alone
- B) Last-touch CAC alone
- C) Spend, conversions (and CAC) under at least two attribution models, plus a saturation/upstream-path check
- D) Total number of touches alone

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — a user logging a step more than once (common with `site_visit` in real event logs) inflates `COUNT(*)` but not `COUNT(DISTINCT user_id)`. In this seed the two happen to agree, but default to `DISTINCT user_id` because real warehouses rarely stay that clean.
2. **C** — `FIRST_VALUE` repeats the first row's value (in `ORDER BY` order) across every row in the window, giving every step a fixed top-of-funnel denominator without a self-join.
3. **C** — activated → paid_conversion loses 23.1% of entrants (100 − 76.9), the largest percentage loss, even though site_visit → signup loses more people in absolute count (4 vs. 3). Percentage and absolute-count "biggest drop" can point to different steps.
4. **B** — without `PARTITION BY user_id`, `LAG` slides across the whole ordered result set, treating some other user's event as "the previous step," which is meaningless.
5. **B** — a conversion window keeps "did this visit lead to this conversion" comparable across cohorts; without it, older cohorts (which have had more time to convert) look artificially better.
6. **B** — earliest touch, rank 1, is first-touch. Flipping to `ORDER BY touch_ts DESC` gives last-touch instead.
7. **C** — linear splits credit evenly: a 3-touch path is ⅓ each. (D is the position-based 40/20/40 split, a common mix-up.)
8. **B** — with no room for a distinct "middle" weight, a 2-touch path under position-based is typically split 50/50 between the only two touches (some teams choose differently — the point is stating the rule, not memorizing one true answer).
9. **B** — the swing is the finding: first-touch rewards discovery, last-touch rewards closing. Neither is "wrong"; disagreement between them is exactly what should trigger the upstream-path check from Lecture 3.
10. **C** — `NULLIF(count, 0)` converts a zero denominator to `NULL` before the division runs, so the result is `NULL` (undefined) instead of a divide-by-zero error or a misleading `0`.
11. **B** — spend more than tripled with zero incremental touches after day 5; that's the textbook shape of a saturation ceiling within this window.
12. **B** — last-touch hands 100% of the credit to whichever touch is nearest the conversion, even when — as with user 18 — two other channels demonstrably did the discovery work first. That doesn't make the channel worthless, but it means its last-touch number alone overstates its incremental contribution.
13. **B** — a plain join with no filter or split multiplies a user's revenue by their touch count; a 3-touch user's revenue gets counted three times, once per joined row, inflating the grand total well past actual total revenue.
14. **C** — no single number is enough; Lecture 3's argument is that spend, multi-model conversion counts/CAC, and a saturation or reshuffling check together are what keep one cheap ratio from driving a bad budget call alone.

</details>

**Scoring:** 12+ → start Week 3. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; funnels and attribution compound directly into every later week's metrics.
