# Challenge 2 — Debug a Misleading Funnel

**Time:** ~90 minutes. **Difficulty:** Medium. **One root cause, several acceptable fixes.**

## The scenario

A teammate posts this in the growth Slack channel: *"Pulled revenue-by-channel straight from `touches` — no fancy attribution model needed, just joined it to `conversions`. Google Search - Nonbrand checks out at $347, matches what we got with first-touch last week. Numbers below, shipping this in the board deck tomorrow."*

```sql
-- "Revenue by channel — straight from the touch log"
SELECT
    c.channel_name,
    SUM(cv.revenue) AS revenue_by_channel
FROM touches t
JOIN channels c     ON c.channel_id = t.channel_id
JOIN conversions cv ON cv.user_id = t.user_id
GROUP BY c.channel_name
ORDER BY revenue_by_channel DESC;
```

It runs without error. It returns six clean rows. `Google Search - Nonbrand` really does show $347 — the same number Exercise 2 got from a proper first-touch model. Your teammate takes that as confirmation the query is fine. **It isn't**, and the fact that one channel happens to match is what makes this bug dangerous: it passed a spot-check.

Your job: run it, find every way it's wrong, quantify each one, and ship a corrected version — or an honestly-labeled version, if what your teammate actually wants is a different (valid) kind of report.

## Part 1 — Prove it doesn't reconcile

Run the query above. Sum the `revenue_by_channel` column across all six rows. Compare that sum to `SELECT SUM(revenue) FROM conversions;`. State both numbers. *(One of them is $1,590. The other is not close.)* In one sentence, explain **why** a query that joins a one-row-per-conversion table to a many-rows-per-user table, then sums a conversion column with no de-duplication, is guaranteed not to reconcile whenever any user has more than one touch.

## Part 2 — Find the coincidence that hid the bug

`Google Search - Nonbrand` shows $347 in both the buggy query and Exercise 2's correct first-touch model — same number, different reasons. Explain, using the actual touch data for the three converting users who touched `Google Search - Nonbrand` (1, 18, 20), why that channel's number came out right **by accident** — what property of those three users' paths made the bug not trigger for this one channel specifically.

## Part 3 — Find the channel where the bug bites hardest

Compare the buggy query's number for `Meta Ads` to the correct first-touch and last-touch numbers from Exercise 2. Identify the **specific user** whose data caused Meta Ads' buggy number to be inflated, explain exactly how (which of their touches got double-counted and why), and state the dollar amount of that one user's over-count.

## Part 4 — Fix it two different ways

Your teammate actually wants one of two different, both-valid things — figure out which, or deliver both:

**Fix A — "Which channels touched a converting user" (an assist report).** If the real goal is "show me every channel that played a role in a sale, full revenue value, for reach purposes" — that's a legitimate report type, but it must be **labeled** as such and must **never** be summed across channels and called "total attributed revenue," because by design the same revenue appears against multiple channels. Write this version with `SELECT DISTINCT` (or an equivalent dedup) so at least a user touching the same channel twice isn't double-counted *within* that channel, and add a comment explaining why the cross-channel total still won't equal $1,590 and that's expected.

**Fix B — "Attributed revenue by channel" (an actual attribution model).** If the real goal is the board-deck number that must reconcile, replace the query with either your Exercise 2 first-touch or last-touch model (or Challenge 1's linear/position-based model), and prove it sums to $1,590.

Ship both, labeled, so nobody mixes them up again.

## Constraints

- Every dollar figure you report must come from a query you actually ran — no estimating from the earlier lecture tables.
- Part 3 must name the specific user and touch, not just "some users get double-counted."
- Fix A and Fix B must both run and both be clearly labeled with what question each one answers.

## How success is judged

| Signal | Weak | Strong |
|--------|------|--------|
| Diagnosis | "The numbers seem off" | Names the exact join/fan-out mechanism and quantifies the gap ($3,182 vs $1,590, or similar depending on your query) |
| Part 2 (the coincidence) | Not attempted or hand-waved | Correctly identifies the path property that made Nonbrand match by luck |
| Part 3 (the specific bite) | Generic "Meta Ads is affected" | Names user 9, the duplicate Meta touch, and the exact dollar over-count |
| Fixes | One query, unlabeled | Two clearly labeled queries answering two different real questions |

## Stretch

- Compute the *total* over-count precisely: `(buggy query's grand total) - (SUM(revenue) FROM conversions)`. What does that number represent in plain English?
- Your teammate's original query used a plain `JOIN` from `touches` to `conversions`. What would change if it had been a `LEFT JOIN` from `touches`? Would the bug get better or worse, and why?
- Find one more table in this week's schema that would produce the *same* fan-out bug if joined to `conversions` without care. Name it and explain the mechanism in one sentence.

## Submission

Commit `challenge-02.md` (your written diagnosis) and `challenge-02.sql` (the buggy query, your investigation queries, and both fixes) to your portfolio under `c38-week-02/challenge-02/`.
