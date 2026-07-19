# Week 7 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is judgment, defended with SQL and numbers, not recall. Do them after the three exercises.

1. **[Challenge 1 — Profile segments by value](challenge-01-profile-segments-by-value.md)** — take every segmentation you've built this week and rank the resulting groups by what they're actually worth, not by how large they are. *(~60 min.)*
2. **[Challenge 2 — Turn segments into targeting actions](challenge-02-turn-segments-into-actions.md)** — write the actual playbook a CS/growth team would run against each segment, with a query behind every claim. *(~60 min.)*

## How these are judged

There's no answer key with a single correct ranking. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of the SQL/pandas** — every query or script you write actually runs and returns what you claim it returns.
- **Grounded ranking, not gut feeling** — when you claim one segment is worth more attention than another, the claim traces to a specific number (total MRR, total add-on spend, retention risk signal), not an adjective.
- **Honest handling of disagreement** — RFM, behavioral tiers, and k-means clusters won't always agree on where a given customer belongs. Strong submissions notice the disagreements and reason about *why*, instead of picking whichever segmentation gives the cleaner story.

Keep your work in `challenge-01.md` / `challenge-02.md` with your queries **and** your written reasoning. The reasoning is the point — a ranked table with no explanation of *why* the ranking landed that way is an incomplete answer.
