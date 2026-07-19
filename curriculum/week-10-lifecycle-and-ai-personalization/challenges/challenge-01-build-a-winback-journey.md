# Challenge 1 — Build a Win-Back Lifecycle Journey

**Goal:** Design and implement, in SQL and Python, the complete targeting and randomization logic for a win-back lifecycle journey aimed at the `winback_email_discount` segment — the piece of infrastructure a real marketing-automation platform would need to actually run this campaign.

**Estimated time:** 90 minutes.

## Context

Exercise 3 produced 58 customers in the `winback_email_discount` action — high churn risk, low MRR, exactly the population an automated (not human) intervention is built for. A decision table is not a campaign until it has: a precise trigger, a defined sequence of touches, exclusion rules, and — because you're a Crunch Scale graduate, not a marketer who "just sends the email" — a randomized holdout baked in from day one, not bolted on after the fact.

## Part A — Segment definition (SQL)

1. Write the `winback_segment` query from Lecture 3, Section 3, against your persisted `scored_customers` table (or recreate it from `churn_scored.csv` loaded into a table). *(Expected: **58** rows.)*
2. Compute the segment's baseline (no-intervention) churn rate using `AVG(churned_in_window)`. *(Expected: **62.1%**.)* Write one paragraph explaining, precisely, what this number does and does not tell you — is it a promise about what will happen, or something else?
3. **Exclusion rules.** Propose and implement in SQL at least two exclusion criteria a real company would apply before launching this campaign — for example, customers already contacted by a human in the last 14 days, customers with an open support ticket (who should get a human reply, not an automated discount pitch), or customers on a legal/compliance hold. You don't have the underlying tables for these in this week's seed — write the SQL as if the tables existed (`support_tickets` with an `is_open` flag, `crm_contacts` with a `last_human_contact_at`), and state your assumptions explicitly in a comment.

## Part B — Journey design (write-up)

Write `challenge-01.md` covering:

1. **The trigger.** What exact event or state change fires this journey — the customer entering the `winback_email_discount` action for the first time, or something more specific (e.g., only fire once per customer per quarter, to avoid re-triggering someone who was already win-back'd last month)?
2. **The sequence.** Design a concrete day-by-day (or event-by-event) sequence of touches, extending Lecture 3, Section 2's skeleton. For each step, specify: the channel (email, in-app, SMS), the content angle, and the condition that determines whether it fires (e.g., "day 3 reminder fires only if no email open since day 0").
3. **The discount.** Justify a specific discount size and structure (percentage off, fixed dollar credit, extra month free) using the segment's economics — you have `mrr_usd` for every customer in the segment; what's the largest discount you could offer and still come out ahead if it saves the customer, and how does that compare to what you'd propose?
4. **Frequency guardrails.** State, explicitly, the rule that prevents this journey from colliding with any other lifecycle touch (the `inapp_checkin_nudge` or `lifecycle_nurture_email` journeys from the other action cells) for a customer who happens to qualify for more than one over time.

## Part C — Randomized holdout (SQL + Python)

5. Implement the hash-based split from Lecture 3, Section 4, using the exact salt string `"crunch-flow-winback-2025-09"`. *(Expected: **30** in `treatment`, **28** in `holdout`.)*
6. Explain, in your write-up, why the split must happen **before** the journey fires (i.e., at segment-entry time) rather than after observing whether a customer engages with the first email.

## Done when…

- [ ] Part A's segment query returns exactly 58 rows and the 62.1% baseline is reproduced.
- [ ] At least two exclusion rules are implemented in SQL with stated assumptions.
- [ ] `challenge-01.md` answers all four Part B questions with specific, numbered reasoning (not generic marketing-copy platitudes).
- [ ] Part C's split reproduces exactly 30/28.

## Submission

Commit `challenge-01.md` plus your SQL/Python to your portfolio under `c38-week-10/challenge-01/`. Challenge 2 scores the outcome of the journey you designed here.
