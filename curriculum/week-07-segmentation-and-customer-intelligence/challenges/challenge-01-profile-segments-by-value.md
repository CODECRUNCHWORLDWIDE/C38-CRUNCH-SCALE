# Challenge 1 — Profile Segments by Value

**Time:** ~60 minutes. **Difficulty:** Medium. **No single right answer.**

## The scenario

Crunch Flow's Head of Customer Success has seen your RFM tiers, your behavioral tiers, and your k-means clusters from this week's exercises. She has one blunt question: **"Great, three segmentations. Which segment, out of everything you've built, deserves my team's time first?"** She doesn't want three separate answers from three separate techniques — she wants *one* ranked list, with the reasoning made explicit, because her team has bandwidth for maybe two segments this quarter, not eight.

"Value" in her question doesn't just mean current MRR — a segment that's currently small revenue but growing fast, or currently large revenue but quietly rotting, both deserve a hard look. Your job is to build a value profile that captures more than one dimension, then rank honestly.

## Your task

Write `challenge-01.md`. Do the following, in order:

### Part 1 — Build a unified segment-value table (SQL)

For every one of Exercise 1's five `rfm_segment` groups, compute in a single query:

- `n` — customer count
- `total_mrr` — sum of subscription MRR (`customers.mrr`) for the segment
- `total_addon_spend` — sum of `orders.amount` for the segment (this is what Exercise 1 called `monetary`, rolled up)
- `avg_power_events` — average behavioral power-event count (from Exercise 2) for the segment
- `pct_dormant` — what percentage of the segment is behaviorally `Dormant` (from Exercise 2)

This requires joining your RFM CTE chain to your behavioral CTE chain — reuse both from Exercises 1 and 2 rather than re-deriving them from scratch.

### Part 2 — Rank the five segments (written reasoning)

Using Part 1's table, rank the five `rfm_segment` groups from **highest** to **lowest** priority for the CS team's next quarter. For each segment, in 2–3 sentences:

- State its rank and the primary reason (one number from Part 1 that drives the call).
- State the **secondary** consideration that either reinforces or complicates the primary reason (e.g., a segment might have high current MRR but also a high `pct_dormant`, which should worry you even though the top-line number looks good).
- Recommend a **time horizon**: is this a "act this week" segment, a "monitor this quarter" segment, or a "revisit next quarter" segment?

### Part 3 — Where the three techniques disagree

Pick **two specific customers** where the RFM tier (Exercise 1), the behavioral tier (Exercise 2), and the k-means cluster (Exercise 3) don't all point to the same conclusion about how healthy that customer's relationship with Crunch Flow is. For each:

- State the customer, and the three techniques' verdicts, side by side.
- Explain, in plain language, what's actually going on with that customer that causes the disagreement.
- Say which of the three techniques you'd trust *most* for this specific customer, and why.

## Constraints

- Every number in Part 1 must come from a query you actually ran — paste the query and its real output, not a hand-estimated table.
- Part 2's ranking must use **at least two** of the five Part 1 columns per segment in your justification — a ranking based on `total_mrr` alone is exactly the "sort by lifetime spend and stop thinking" mistake Lecture 1 warns against.
- Part 3's two customers must be **real** disagreements you found in your own data, not hypothetical ones.

## How success is judged

| Signal | Weak submission | Strong submission |
|--------|------------------|--------------------|
| Evidence | A ranking with no query behind it | Every number in Part 1 traces to a runnable query |
| Ranking logic | "Champions are #1 because they spend the most" | Uses ≥2 columns per segment, and at least one ranking call goes against the naive "sort by revenue" answer |
| Disagreement analysis | Picks two customers who actually agree across all three techniques | Genuinely finds and explains real disagreement, with a stated trust call |
| Business framing | Generic "focus on top segment" | Time-horizon recommendations a CS team could put on a calendar tomorrow |

## Submission

Commit `challenge-01.md` (plus your Part 1 SQL) to your portfolio under `c38-week-07/challenge-01/`.
