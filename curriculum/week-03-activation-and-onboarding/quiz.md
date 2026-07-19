# Week 3 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 4. A mix of multiple-choice and short "what does this return?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Why is activation described as the highest-leverage stage in the AARRR funnel?

- A) It's the easiest stage to measure.
- B) Acquisition spend is wasted if activation is broken, and retention is impossible without it — and unlike acquisition or retention, it's fixable on a short feedback loop.
- C) It generates the most revenue directly.
- D) It's the first stage a user encounters.

---

**Q2.** Which of these is the strongest reason **not** to use "became a paying customer" as an activation event?

- A) Payment data is hard to query in SQL.
- B) Payment is downstream of activation for most products — users pay because they already got value, so it arrives too late to act on.
- C) Not all products have a paid tier.
- D) Payment events are always duplicated in the events table.

---

**Q3.** A candidate activation event has 9.2% reach and a strong retention lift. What's the correct conclusion?

- A) It's automatically the best choice — highest lift wins.
- B) It should be discarded entirely — low reach means it's meaningless.
- C) It's a real signal but too rare to serve as the org's single, actionable activation KPI.
- D) Reach doesn't matter once you've confirmed a lift exists.

---

**Q4.** In this week's seed data, which candidate event has the highest lift **and** a workable (not tiny) reach?

- A) `verified_email`
- B) `connected_integration`
- C) `invited_teammate`
- D) `signed_up`

---

**Q5.** `completed_first_task` has a lift almost as high as `invited_teammate`'s. Why is it still the weaker choice as *the* activation event?

- A) It has lower reach than `invited_teammate`.
- B) It happens later, and in this data nearly always occurs after `invited_teammate` — its correlation likely reflects users who were already engaged, not an independent early lever.
- C) It's not a real event in the seed data.
- D) `LAG()` cannot be computed on it.

---

**Q6.** What does "window leakage" refer to, in the context of measuring activation-event reach?

- A) Counting an event as "reached" without capping it to an early window, so users who did it very late (with no early information available) still count as activated.
- B) A SQL syntax error in a windowed aggregate.
- C) Data loss from an uncommitted transaction.
- D) The percentile function producing `NULL`.

---

**Q7.** Why do you need a retention definition (like `week4_active`) **before** you can build a lift table?

- A) You don't — lift tables only need the candidate events.
- B) The lift table measures how well each candidate predicts the retention outcome, so the outcome has to be defined and computed first.
- C) SQL requires all `CREATE TABLE` statements to run before any `SELECT`.
- D) Retention and activation are the same metric.

---

**Q8.** In the `flags` CTE pattern from Lecture 2, what does each row represent?

- A) One event.
- B) One user, with boolean columns for each candidate action plus the retention outcome.
- C) One day of the observation window.
- D) One funnel step.

---

**Q9.** A step funnel shows `created_first_card → invited_teammate` converting at 55.5%. What does that number mean?

- A) 55.5% of all 400 signups invited a teammate.
- B) Of the users who reached `created_first_card`, 55.5% went on to reach `invited_teammate`.
- C) 55.5% of invited teammates accepted the invite.
- D) The step took 55.5 hours on average.

---

**Q10.** Which window function does the Lecture 3 step-conversion query rely on to compare each step's count to the previous step's?

- A) `RANK()`
- B) `ROW_NUMBER()`
- C) `LAG()`
- D) `NTILE()`

---

**Q11.** `created_first_card`'s cumulative reach is 52.2% of signups, and its step conversion to `invited_teammate` is 55.5%. Roughly what should `invited_teammate`'s cumulative reach be?

- A) 107.7% (add them)
- B) About 29% (multiply them)
- C) 3.3% (subtract them)
- D) There's no relationship between the two numbers.

---

**Q12.** Why is reporting only the **mean** time-to-value potentially misleading?

- A) The mean is always mathematically wrong.
- B) Time distributions are typically right-skewed, so a handful of slow stragglers can pull the mean well past where most users actually land — the median and spread tell the real story.
- C) SQL cannot compute a mean over time intervals.
- D) It isn't misleading; the mean is always the best single summary.

---

**Q13.** SQLite has no `PERCENTILE_CONT`. What's the portable fallback used in this week's materials?

- A) Convert the SQLite database to PostgreSQL first.
- B) Sort the values and select by row position with `ORDER BY` + `LIMIT`/`OFFSET`.
- C) There is no fallback; percentiles are impossible in SQLite.
- D) Use `AVG()` instead, since it's equivalent.

---

**Q14.** Challenge 2's projection assumes newly-converted inviters would retain at the same rate as today's existing inviters. What is this assumption, and what does the challenge require you to do about it?

- A) It's a proven fact from the data; no further action is needed.
- B) It's a simplifying assumption that must be stated explicitly, alongside naming the experiment (an A/B test, Week 8) that would actually validate the projected lift.
- C) It's irrelevant to the final number.
- D) It replaces the need for a sensitivity range.

---

**Q15.** Which statement best describes what a retention-lift table (Lecture 2) proves versus what it doesn't?

- A) It proves the candidate event *causes* retention; no further testing is needed.
- B) It's a hypothesis-generating correlation; only a controlled experiment can establish that the event *causes* the retention difference.
- C) It proves nothing at all and should be ignored.
- D) It proves causation only if the lift exceeds 50 percentage points.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — acquisition spend is wasted without activation, retention can't happen without it, and it's the fastest stage to iterate on and measure — the combination that makes it the highest-leverage lever.
2. **B** — payment is a monetization decision users make *after* they've already gotten value; using it as the activation event means you find out about a broken activation experience only after it's too late to intervene.
3. **C** — this is the reach trap from Lecture 1/2: a real signal, but too rare (9.2%, `connected_integration` in the seed) to be the org's headline, actionable metric.
4. **C** — `invited_teammate`: +50.4pp lift, 29.0% reach — the strongest combination of lift and workable reach among all six candidates.
5. **B** — the reverse-causation/endogeneity trap: in the seed, `completed_first_task` essentially always follows `invited_teammate`, so its lift likely reflects users who were already retained-track, not an independent early lever.
6. **A** — measuring reach (or any candidate flag) without an early time cap lets very-late occurrences count, even though on the day you'd want to act, that information didn't exist yet.
7. **B** — a lift table's whole purpose is comparing each candidate against a retention outcome; without that outcome defined and computed first, there's nothing to correlate against.
8. **B** — one row per user, boolean flag columns per candidate event plus the retention outcome — this shape is what makes every subsequent aggregate a simple `GROUP BY`/`FILTER`.
9. **B** — step conversion is relative to the *previous* step's population, not the original 400 signups — that's what distinguishes it from cumulative reach.
10. **C** — `LAG()` pulls the previous row's value into the current row within an `ORDER BY step_order` window, avoiding a manual self-join.
11. **B** — cumulative reach at one step times step-over-step conversion to the next gives (approximately) the next step's cumulative reach: 52.2% × 55.5% ≈ 29.0%.
12. **B** — right-skewed time distributions (a few very slow adopters) drag the mean above where the bulk of users actually land; the median and percentile spread describe the real shape.
13. **B** — sort the values, then pick by row position (`LIMIT`/`OFFSET`) — an approximate, non-interpolated percentile, but portable and precise enough for course purposes.
14. **B** — it's explicitly a simplifying assumption; the challenge requires stating it plainly and naming the A/B test (Week 8) that would actually confirm whether the projected lift holds.
15. **B** — a lift table generates a well-evidenced hypothesis about what predicts retention; it does not, by itself, prove the candidate event *causes* the retention difference — only a controlled experiment does that.

</details>

**Scoring:** 13+ → start Week 4. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; this week's reasoning (reach vs. lift, causation vs. correlation) compounds directly into Week 4's cohort work and Week 8's experiments.
