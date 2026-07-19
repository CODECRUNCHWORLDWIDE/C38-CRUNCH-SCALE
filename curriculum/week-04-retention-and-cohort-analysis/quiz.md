# Week 4 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 5. A mix of multiple-choice and short "what does this return?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In a cohort retention triangle, why is `month_number = 0` always 100% retention?

- A) Because the first month always has the most users
- B) Because by construction, everyone in a cohort was active in their own signup month
- C) Because month 0 excludes churned users
- D) It isn't always 100% — that's a bug if you see it

---

**Q2.** The May 2025 cohort's row in the triangle has no `month_number = 2` cell. What does that empty cell mean?

- A) 0% of the cohort retained to month 2
- B) The cohort hasn't existed long enough to have a month-2 data point yet
- C) A data-loading error
- D) Every user in that cohort churned before month 2

---

**Q3.** A cohort's retention curve reads: 100% → 45% → 22% → 19% → 20% → 19%. What shape is this?

- A) Terminal (heading to zero)
- B) Flattening, with a floor around 19–20%
- C) Smiling
- D) Cannot be determined from percentages alone

---

**Q4.** Why is comparing the *same column* (same `month_number`) across different cohort rows more useful for measuring product improvement than comparing different columns within one cohort's row?

- A) It isn't — within-row comparison is always better
- B) Same-column comparison isolates "did retention change" from "how much time has this cohort had"
- C) Different columns within a row can't be compared at all
- D) Same-column comparison only works with exactly two cohorts

---

**Q5.** A user's day-since-signup values with events are 3, 9, and 41. Their **N-day retention** at N=7 is:

- A) True — they had an event on day 9, which is close to 7
- B) False — no event occurred on exactly day 7
- C) True — they had at least one event within 7 days
- D) Undefined — N-day retention requires at least 3 data points

---

**Q6.** Which retention definition is the right choice for a live dashboard that needs to answer "is our active user base healthy *today*"?

- A) N-day retention
- B) Unbounded retention
- C) Rolling (trailing-N-day) retention
- D) The cohort triangle

---

**Q7.** A user is active in March, inactive in April, and active again in May. How is May classified in the growth-accounting split?

- A) Retained
- B) New
- C) Resurrected
- D) Churned

---

**Q8.** Which equation is the growth-accounting identity that active users in a period must satisfy?

- A) `active = new − retained − resurrected`
- B) `active = new + retained + resurrected`
- C) `active = new + retained + resurrected + churned`
- D) `active = retained − churned`

---

**Q9.** `churned(month)` is computed against which period's active count?

- A) The current month's active count
- B) The previous month's active count
- C) The cohort's original signup count
- D) The next month's active count

---

**Q10.** Why does true Kaplan-Meier survival analysis not perfectly apply to product retention curves?

- A) Survival analysis requires more than 4 cohorts
- B) Survival analysis assumes the event (death) is absorbing/irreversible, but a churned user can resurrect
- C) Survival analysis can't handle monthly data, only daily
- D) It applies perfectly; the names are unrelated

---

**Q11.** In the eligible-adjusted survival table, `eligible_users` shrinks from 48 (month 0) down to 12 (month 4). Why?

- A) 36 users were deleted from the dataset
- B) Fewer and fewer cohorts have lived long enough to reach later month numbers
- C) It's a data-quality bug that should be fixed
- D) Eligible users always equal active users

---

**Q12.** A product has MAU = 20 and an average of 3 daily active users across the month. Its stickiness ratio is:

- A) 3%
- B) 15%
- C) 20%
- D) 60%

---

**Q13.** Two segments have identical unbounded month-1 retention (both 40%), but Segment A has 8% stickiness and Segment B has 30% stickiness. What does this tell you that retention alone didn't?

- A) Nothing — if retention is equal, the segments are equal
- B) Segment B's users engage far more frequently while they're retained, even though the same fraction of each segment sticks around
- C) Segment A has more total users
- D) Stickiness and retention always move together, so this scenario is impossible

---

**Q14.** Why should `NOT IN`-style spreadsheet pivot tables never be used as the system of record for a cohort-retention analysis in this course?

- A) Spreadsheets can't sort data
- B) A spreadsheet can hold a report, but not the raw event log it's derived from, in a way that's auditable and reproducible when the definition changes
- C) Spreadsheets don't support percentages
- D) There is no reason; spreadsheets are equally valid here

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — everyone in a cohort was, by definition, active in the month they signed up. Month 0 tells you nothing about *retention*; the story starts at month 1.
2. **B** — an empty cell means "not observed yet" (the cohort is too young), which is different from a 0% cell ("observed, and nobody came back"). Confusing the two is the classic censoring mistake.
3. **B** — flattening: a sharp initial drop settling into a stable floor around 19–20%. The small wiggle (19→20→19) is normal noise, not a new trend.
4. **B** — same-`month_number` comparison across cohorts holds "time since signup" constant, so any difference reflects a real change in the product/onboarding rather than one cohort simply having existed longer.
5. **B** — N-day retention at N=7 requires activity on *exactly* day 7. Day 9 doesn't count, no matter how close.
6. **C** — rolling retention is a present-tense, whole-base snapshot that updates continuously, which is exactly what a live dashboard needs. The triangle and unbounded retention are cohort-anchored and only gain a new data point once a full period passes.
7. **C** — resurrected: active this period, not active the immediately preceding period, but active at some earlier point (March). If they'd been active in April too, May would be "retained."
8. **B** — `active = new + retained + resurrected`. Churned is a separate count, checked against the *previous* period's active total, not summed into the current period.
9. **B** — the previous month's active count. `churned(month) = active(month−1) − retained(month)`.
10. **B** — Kaplan-Meier assumes an absorbing event (death doesn't reverse). Product churn is reversible — a user can resurrect — so the "survival curve" used in growth analytics borrows the name and the censoring discipline, not the irreversibility assumption.
11. **B** — by month 4, only the oldest cohort (Feb, 12 users) has actually lived 4 months; the other three cohorts haven't reached month 4 yet and are correctly excluded from that month's denominator.
12. **B** — 3 / 20 = 0.15 = 15%.
13. **B** — retention (presence over time) and stickiness (frequency while present) measure different things. Equal retention with very different stickiness means Segment B's retained users are engaging much more often — a materially different product experience even though the "did they stay" number matches.
14. **B** — a spreadsheet can present a finished report, but it can't serve as the raw, queryable, version-controlled event log a retention definition is built from — and retention definitions change (as this week showed with N-day vs. unbounded vs. rolling). SQL keeps the raw facts auditable and every derived metric reproducible.

</details>

**Scoring:** 12+ → start Week 5. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; unit economics in Week 5 is built directly on this week's retention numbers.
