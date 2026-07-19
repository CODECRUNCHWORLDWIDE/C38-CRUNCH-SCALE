# Challenge 1 — Design a Full Metric Tree

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

StreakLab's founder has read Lecture 1 and now wants more than a north star on a slide — she wants a **metric tree document** she can hand to three team leads (Acquisition, Product, Lifecycle) so each of them knows exactly which number is theirs to move, and exactly which SQL query proves whether they moved it. Lecture 3, section 4 sketched this; your job is to build the complete version, one level deeper than the lecture went.

## Your task

Produce `challenge-01.md` containing a **tree diagram** (plain text/ASCII is fine — see the example below) plus a **runnable SQL query for every single node** in the tree.

### Required tree shape

```
Weekly Engaged Users (NORTH STAR)
├── New Activated Users (this week)
│   ├── New Signups (this week)
│   └── Activation Rate (%)
├── Returning Engaged Users (activated earlier, still 3+ check-ins/week)
│   └── Week-over-week Retention Rate (%)
└── [your addition — see below]
```

1. **Reproduce the three branches above with a query for each node**, using the seed data's "this week" = the 7 days ending 2026-06-21.
2. **Add a fourth branch of your own** connecting **Revenue** or **Referral** into the tree — you decide which, and how it plausibly feeds back into future Weekly Engaged Users (e.g., does a referred signup activate faster or slower than an organic one? Does subscribing change check-in behavior?). Write the query that tests your hypothesis against the seed data, and report what it actually shows — even if the answer is "no detectable difference," which is a valid, honest finding with only 15 users.
3. For every node, write **one sentence** stating which team/role owns moving that number.

## Constraints

- Every node needs a **working SQL query** — a tree with unqueried boxes is a diagram, not a metric tree, and this course doesn't accept diagrams as a deliverable.
- The three required branches must **mechanically reconcile**: New Activated Users + Returning Engaged Users should be a sensible (not necessarily exact, but explainable) decomposition of the total Weekly Engaged Users number from Exercise 3, Task 3. If your two branches don't roughly add up, say why in prose (e.g., "some returning engaged users activated more than a week ago and don't appear in either 'new' bucket, which is expected").
- Your 4th branch must connect to a **different** AARRR stage than the three given branches (which already cover Acquisition and Activation) — pick Revenue or Referral, not another Activation angle.

## Hints

<details>
<summary>On "New Activated Users (this week)"</summary>

This is: `habit_created` events whose `event_time` falls in the last 7 days, counted by distinct `user_id`. Compare this to the *cumulative* activation rate from Exercise 3 (which asks about all-time activation) — this challenge wants the **weekly flow**, not the all-time stock. The distinction between a stock (total activated, ever) and a flow (newly activated, this week) is one of the most common confusions in growth reporting — get it right here and it won't trip you up again.

</details>

<details>
<summary>On reconciling the branches</summary>

With only 15 users and 3 signup cohorts, don't expect the arithmetic to be clean — that's realistic. A tree with 5,000 users would reconcile more smoothly by the law of large numbers; a tree with 15 will show its seams. Naming the seam honestly (in one sentence) is worth more credit than forcing a fake-clean number.

</details>

## How success is judged

| Signal | Weak submission | Strong submission |
|--------|------------------|--------------------|
| Query coverage | Some nodes have no SQL, or SQL that doesn't run | Every node has a runnable, correct query |
| Tree logic | Branches are just "other metrics I found interesting" | Branches genuinely decompose or explain the north star |
| 4th branch | Restates Activation in different words | Connects a genuinely different AARRR stage (Revenue/Referral) with a testable hypothesis |
| Ownership | Missing or vague ("the team") | Specific role per node, matching Lecture 1's table |
| Honesty about seams | Pretends the small dataset reconciles perfectly | Names where/why the numbers don't add up cleanly |

## Submission

Commit `challenge-01.md` (queries + tree + reasoning) to your portfolio under `c38-week-01/challenge-01/`.
