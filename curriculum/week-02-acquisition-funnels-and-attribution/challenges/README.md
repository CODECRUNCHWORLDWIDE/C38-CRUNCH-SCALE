# Week 2 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is judgment and debugging, not recall. Do them after the three exercises.

1. **[Challenge 1 — Build a multi-touch attribution model](challenge-01-multi-touch-attribution-model.md)** — implement linear and position-based (40/20/40) attribution from scratch, using only what Lecture 2 taught you as a starting point. *(~90 min.)*
2. **[Challenge 2 — Debug a misleading funnel](challenge-02-debug-a-misleading-funnel.md)** — a teammate's report runs without error and is confidently wrong; find the bug, explain it, fix it. *(~90 min.)*

## How these are judged

There's no answer key with a single row count for Challenge 1's weighting choices, and Challenge 2 has one root cause but several acceptable fixes. You're being graded on:

- **Correctness of the SQL** — does the query actually compute what you claim it computes?
- **Explicit assumptions** — when a rule is ambiguous (edge cases in weighting, tie-breaking), did you write down the interpretation you chose?
- **Reconciliation discipline** — does your attribution total match `SUM(revenue) FROM conversions`? If not, did you notice, and fix it?

Keep your work in `challenge-01.md` / `challenge-02.md` with your queries **and** your written reasoning. The reasoning is the point.
