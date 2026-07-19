# Challenge 2 — Turn Segments into Targeting Actions

**Time:** ~60 minutes. **Difficulty:** Medium. **No single right answer.**

## The scenario

A segmentation that doesn't change what anyone *does* is a slide deck, not an analysis. Crunch Flow's VP of Growth has read Challenge 1's ranking and agrees with your priority call — now she wants an actual playbook: for each of the top three segments you identified, what does the team send, build, or say, and how will you know if it worked? She's explicit: **"No generic 'send them an email.' I want the actual message, the actual trigger condition as a SQL `WHERE` clause, and the actual metric that tells us in 60 days whether it worked."**

## Your task

Write `challenge-02.md`. For the **top three segments** from Challenge 1's ranking, produce a targeting card for each with all five of the following:

1. **Trigger condition** — the exact SQL `WHERE` clause (or full query) that would select this segment's members today, runnable against the seed schema. Not a description of the segment — the actual query.
2. **The action** — specific enough that a CS or growth team member could execute it without asking a follow-up question. ("Reach out" is not specific. "Send a 1:1 email from the assigned CSM offering a 20-minute workflow-automation walkthrough, referencing the specific power-feature gap this segment has" is specific.)
3. **The message angle** — one sentence of what the outreach actually says, grounded in what you know about this segment's real behavior (not a generic template).
4. **The success metric** — the specific number you'd check in 60 days, and what result would count as the action having worked. Must be a metric you could compute with a SQL query against an updated version of this week's schema.
5. **The failure signal** — what result, 60 days out, would tell you the action *didn't* work and something different is needed.

Then, in a closing section (**"What I'd deliberately skip"**), name **one segment** from the full five you're choosing *not* to target this quarter, and defend that choice — every segmentation exercise implicitly asks a team to say no to something, and pretending you have bandwidth for everyone is a cop-out.

## Constraints

- Every trigger-condition query must actually run against the seed data and return a real customer list — paste the query and the row count it returns.
- At least one of your three targeting cards must be aimed at **retention/save** (a segment showing risk) and at least one must be aimed at **expansion** (a segment showing upside) — don't pick three cards that are all the same flavor of action.
- The success metric for each card must be something this week's schema can actually measure (e.g., a change in `orders` frequency, a change in `power_events` count, a change in `days_since_last_event`) — not an abstract KPI you can't compute from the tables you have.

## How success is judged

| Signal | Weak submission | Strong submission |
|--------|------------------|--------------------|
| Specificity | "Send them a re-engagement email" | Exact trigger query + a message angle grounded in that segment's real numbers |
| Balance | Three retention-flavored cards, nothing on expansion | At least one retention card and one expansion card, clearly distinguished |
| Measurability | Success metric is a vague feeling ("they seem happier") | Success metric is a specific query against `orders`/`product_events` you could re-run in 60 days |
| Honest tradeoff | Claims all five segments deserve equal attention | Names one segment deliberately skipped, with a real reason |

## Submission

Commit `challenge-02.md` to your portfolio under `c38-week-07/challenge-02/`.
