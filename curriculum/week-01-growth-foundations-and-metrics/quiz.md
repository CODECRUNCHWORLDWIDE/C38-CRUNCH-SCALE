# Week 1 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 2. A mix of multiple-choice and short "what would this query return?" — the answer key explains the *why*, not just the letter.

---

**Q1.** Which of these is the best one-sentence definition of a "metric"?

- A) Any number that appears on a dashboard
- B) A number, computed the same way every time, that summarizes something real about the business
- C) Whatever the CEO decides matters this quarter
- D) A percentage change from last month

---

**Q2.** A candidate north-star metric should pass all **five** questions from Lecture 1. Which of these is **not** one of them?

- A) Does it reflect real value delivered to the customer?
- B) Is it a leading indicator, not a lagging one?
- C) Is it always a currency amount, like dollars?
- D) Is it hard to game in isolation?

---

**Q3.** "Total signups, all time" fails the five-question test hardest on which count?

- A) It's impossible to compute in SQL
- B) It's a cumulative count that can be inflated by ad spend alone and never reflects delivered value
- C) It changes definition every week
- D) It can't be broken down by channel

---

**Q4.** Put the AARRR stages in the correct order:

- A) Referral, Acquisition, Retention, Activation, Revenue
- B) Acquisition, Activation, Retention, Revenue, Referral
- C) Activation, Acquisition, Revenue, Retention, Referral
- D) Acquisition, Retention, Activation, Referral, Revenue

---

**Q5.** In StreakLab's schema, which event marks **Activation**, and why not `session_start`?

- A) `session_start` — any app open counts as activation
- B) `subscription_started` — activation means paying
- C) `habit_created` — it's the first moment a user experiences the product's core value; opening the app proves nothing on its own
- D) `checkin_logged` — activation requires a full week of use

---

**Q6.** StreakLab's north star, Weekly Engaged Users, is defined as:

- A) Any user with at least 1 `session_start` in the trailing 7 days
- B) Any user with 3 or more `checkin_logged` events in the trailing 7 days
- C) Any user who has ever created a habit
- D) Any user currently paying for Pro

---

**Q7.** Given the seed data, how many of StreakLab's 15 users are Weekly Engaged as of 2026-06-21?

- A) 15
- B) 12
- C) 10
- D) 6

---

**Q8.** A metric is "vanity" primarily when:

- A) It's measured in percentages instead of raw counts
- B) It can go up while the business gets no healthier — often because it's a raw cumulative count easily inflated by spending
- C) It was invented by the marketing team
- D) It's hard to compute in SQL

---

**Q9.** Why does this course insist that growth numbers live in PostgreSQL, never in a spreadsheet?

- A) Spreadsheets can't do math correctly
- B) A spreadsheet of hand-typed numbers throws away the raw events, so the metric's definition can never be audited or recomputed later
- C) PostgreSQL is free and spreadsheets cost money
- D) Spreadsheets can't be shared with a team

---

**Q10.** In the `users`/`events` schema, `users` is the ______ table and `events` is the ______ table.

- A) fact / dimension
- B) staging / marts
- C) dimension / fact
- D) raw / staging

---

**Q11.** Why must rows in `events` be immutable (append-only, never `UPDATE`d or `DELETE`d)?

- A) PostgreSQL doesn't support `UPDATE` on tables with a primary key
- B) It keeps the table small
- C) It makes the log auditable — a permanent record, like a ledger that posts reversing entries instead of erasing history
- D) It's required by SQL syntax

---

**Q12.** What does `COUNT(*) FILTER (WHERE event_name = 'habit_created')` compute, inside a `GROUP BY user_id` query?

- A) A syntax error — `FILTER` isn't valid on `COUNT`
- B) The count of `habit_created` rows for each user, without needing a `CASE` expression or a separate subquery
- C) The count of every event, regardless of `event_name`
- D) Whether the user has ever logged in

---

**Q13.** A correct referral **rate** metric requires which two events, in relation to each other?

- A) `session_start` and `checkin_logged`, unrelated counts
- B) `invite_sent` and `invite_accepted`, as a conversion — counting `invite_sent` alone is gameable by nagging users to send invites nobody accepts
- C) `signup` and `subscription_started`
- D) Just `invite_sent` — accepted invites don't matter for referral health

---

**Q14.** MRR grew sharply this month while StreakLab's Weekly Engaged Users barely moved. What's the most defensible interpretation, per Lecture 1's leading-vs-lagging argument?

- A) The north star is broken and should be replaced with MRR
- B) MRR is a lagging signal that can move independently short-term (e.g., one big renewal); WEU is the leading signal of whether the underlying product value is actually growing
- C) This is impossible and indicates a data bug
- D) Revenue and engagement always move together, so one of the two numbers must be miscomputed

---

**Q15.** A product ships a feature letting users log a check-in "for yesterday," retroactively. What's the risk to the Weekly Engaged Users metric if the query definition doesn't change?

- A) None — `event_time` will always reflect exactly when the behavior happened, so nothing changes
- B) Backfilled check-ins could make a past week's WEU appear to change after the fact, or let users pad an old week's count after the fact — the query needs to decide whether it trusts `event_time` (when logged) or a separate "logged for" date
- C) The `events` table would need a new primary key
- D) `checkin_logged` would need to be renamed

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — a metric is a reproducible, well-defined number. "Whatever the CEO decides" or "appears on a dashboard" both fail the "computed the same way every time" test.
2. **C** — dollars-only is not one of the five questions. The five are: real customer value, leading indicator, team-movable, hard to game, one-sentence understandable.
3. **B** — total signups is a pure top-of-funnel cumulative count, trivially inflated by ad spend, and reflects zero delivered value on its own (question 1 and question 4 both fail).
4. **B** — Acquisition, Activation, Retention, Revenue, Referral, in that order — each stage narrows the funnel from the one before it.
5. **C** — `habit_created` is StreakLab's activation event because it's the first moment the core value (starting a habit) is actually experienced; `session_start` only proves the app opened, not that anything of value happened.
6. **B** — 3 or more `checkin_logged` events in the trailing 7 days. Not `session_start` (too weak a signal) and not "ever created a habit" (that's activation, a one-time event, not a repeat-behavior retention signal).
7. **C** — 10. (12 are activated; 2 of those 12 don't clear the 3-checkins-in-7-days bar this particular week, most visibly the "churner" archetype whose check-ins stopped a week or more ago.)
8. **B** — vanity means inflatable without reflecting real business health, most commonly via raw cumulative counts driven up by spending.
9. **B** — the structural problem: a spreadsheet of typed numbers has already discarded the raw events, so there's no way to recompute, audit, or correct the metric's definition later. It's not about spreadsheet math being wrong or cost.
10. **C** — `users` is the dimension table (who); `events` is the fact table (what happened, when).
11. **C** — immutability is what makes the event log auditable and trustworthy over time, the same principle as a financial ledger that never erases a transaction.
12. **B** — `FILTER (WHERE ...)` aggregates over a subset of the grouped rows in one clean clause, replacing a `CASE WHEN ... THEN 1 ELSE 0 END` pattern.
13. **B** — referral health needs the sent→accepted **pair** as a conversion; raw `invite_sent` alone rewards nagging over genuine advocacy.
14. **B** — MRR is a lagging, sometimes lumpy signal (one renewal can move it); WEU is the leading signal Lecture 1 chose specifically because it can't move without real, repeated product use.
15. **B** — without a defined "logged for" date separate from "when it was actually entered," backfilled check-ins can silently distort a historical week's engaged-user count after the fact — a live instance of the "the definition must be precise and stable" argument from Lecture 1.

</details>

**Scoring:** 13+ → start Week 2. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; Week 2 (acquisition/attribution) assumes every one of these concepts is automatic.
