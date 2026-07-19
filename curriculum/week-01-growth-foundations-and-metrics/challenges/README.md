# Week 1 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is judgment, defended with SQL, not recall. Do them after the three exercises.

1. **[Challenge 1 — Design a full metric tree](challenge-01-design-a-metric-tree.md)** — build StreakLab's complete north-star + input-metric tree, with a runnable SQL query behind every single node. *(~90 min.)*
2. **[Challenge 2 — Defend a north star against gaming](challenge-02-defend-a-north-star.md)** — a skeptical exec tries to poke holes in Weekly Engaged Users. You defend it, fix the real gaps, and know when to concede. *(~90 min.)*

## How these are judged

There's no answer key with a single correct row count. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of the SQL** — every query you write actually runs and returns what you claim it returns.
- **Soundness of the tree structure** — do the input metrics genuinely combine (by multiplication or funnel narrowing) to explain the north star, or are they just a list of numbers that sound related?
- **Honest defense** — when a challenge pokes a real hole in your metric, do you patch the definition, or do you hand-wave? Strong submissions concede real gaps and propose a specific fix.

Keep your work in `challenge-01.md` / `challenge-02.md` with your queries **and** your written reasoning. The reasoning is the point — the SQL alone, without an explanation of *why* it's structured that way, is an incomplete answer.
