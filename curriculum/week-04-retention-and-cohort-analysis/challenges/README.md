# Week 4 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is diagnosis and argument, not recall. Do them after the three exercises.

1. **[Challenge 1 — Diagnose a leaky-bucket product](challenge-01-diagnose-a-leaky-bucket.md)** — one segment of Crunch is bleeding users. Find it, quantify it, and propose what you'd investigate next. *(~90 min.)*
2. **[Challenge 2 — Compare retention across segments](challenge-02-compare-retention-across-segments.md)** — `self_serve` and `sales_assisted` users behave very differently. Build the comparison properly, and know which parts of it you can trust. *(~90 min.)*

## How these are judged

There's no answer key with a single correct number. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of the SQL** — do your queries actually compute what you claim they compute?
- **Correct handling of small samples and censoring** — do you flag where a number is built on too few users, or too little history, to trust at face value?
- **A defensible recommendation** — not just "here are the numbers," but "here's what I'd do next, and why."

Keep your work in `challenge-01.md` / `challenge-02.md` with your queries **and** your written reasoning. The reasoning is the point.
