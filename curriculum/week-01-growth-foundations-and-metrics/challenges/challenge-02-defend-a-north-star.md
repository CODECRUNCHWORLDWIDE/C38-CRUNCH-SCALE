# Challenge 2 — Defend a North Star Against Gaming

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

You proposed **Weekly Engaged Users (WEU)** — 3+ `checkin_logged` events in the trailing 7 days — as StreakLab's north star in Lecture 1. StreakLab's Head of Growth is not convinced. She's smart, skeptical by nature, and her job is to find the metric's weak points before an intern with a KPI target does. She sends you five pointed questions. You must answer every one **with a query, a number, or a specific proposed fix** — "I think it's fine" is not an acceptable answer to any of them.

## Your task

Write `challenge-02.md`. For each question below: quote it, answer it directly, back the answer with SQL and/or a specific number from the seed data where possible, and state whether you're **defending** the metric as-is or **conceding** a real gap and proposing a fix.

### The five questions

1. **"Couldn't a user just tap 'log check-in' three times in one minute, count as 'engaged,' and get nothing out of it?"**
   *(Look at the actual event timestamps in the seed data — is this plausible given how `checkin_logged` events are currently spaced for real users? What would you add to the schema or the query to make rapid-fire fake check-ins detectable or impossible to count?)*

2. **"Why 3 check-ins and not 2, or 5? Did you just pick a number that felt right?"**
   *(Defend the threshold using something more rigorous than a gut feeling — propose how you would actually validate "3" empirically with more data than 15 users, referencing how Facebook validated "7 friends in 10 days." What analysis would you run, even if you can't run it on this tiny seed set?)*

3. **"Two of your five signup channels contribute zero long-term engaged users once I look past week 1 — why is your north star blind to that?"**
   *(Query the seed data: break weekly-engaged-user counts down by original `signup_channel`. Is this claim actually true in the seed data? If yes, is that a flaw in the *north star itself*, or a job for a different, channel-level metric sitting next to it? Argue it either way, with numbers.)*

4. **"MRR grew 50% this month and your precious WEU barely moved. Doesn't that prove revenue is the real story?"**
   *(Compute both numbers from the seed data as best you can. Then answer: does a north star need to move in lockstep with revenue every single month? Revisit Lecture 1's argument about leading vs. lagging indicators — restate it specifically against this objection, not generically.)*

5. **"What happens to your metric the day we ship a feature that lets people log a check-in for yesterday, retroactively?"**
   *(This is a real, common product change — "backfill" logging. Walk through exactly how it would distort WEU if the metric definition doesn't change, and propose the specific schema or query change that keeps the metric honest once backfilled check-ins exist. Hint: think about what timestamp the query should filter on — `event_time`, or something else entirely.)*

## Constraints

- You must genuinely **concede at least one** of the five points — find the one where the Head of Growth has a real, unpatched hole in your metric, and say so plainly, with a proposed fix. A submission that defends all five perfectly reads as defensive, not rigorous — real north stars have real weak points.
- Every quantitative claim ("channel X contributes zero long-term users") must be backed by an actual query against the seed data, not an assertion.
- Question 5's answer must include the literal schema or query change, not just "we'd think about it."

## How success is judged

| Signal | Weak submission | Strong submission |
|--------|------------------|--------------------|
| Evidence | Opinions without queries | Every claim backed by a query and its actual result |
| Honesty | Defends everything | Concedes at least one real gap with a specific fix |
| Depth on Q2 | "3 felt right" restated | A concrete plan for validating the threshold with more data |
| Depth on Q5 | Vague "we'd handle it" | Names the exact timestamp/schema fix (e.g., distinguishing `event_time` from a `logged_for_date` the event refers to) |
| Tone | Combative or purely defensive | Reads like a real analyst who respects the challenge |

## Submission

Commit `challenge-02.md` to your portfolio under `c38-week-01/challenge-02/`.
