# Mini-Project — Channel Funnel & Multi-Touch Attribution Report

> Build the report a growth team actually ships: a full acquisition funnel, four attribution models compared side by side, a CAC report that accounts for saturation and reshuffled demand, and a one-page recommendation for where Crunch's next acquisition dollar should go. One dataset, one deliverable, real evidence.

**Estimated time:** 2.5–3 hours, best done after all three exercises and both challenges.

This is the week's capstone, and it's deliberately shaped like a real growth-analytics request: nobody hands you a SQL assignment, they hand you a decision to inform. You have everything you need in the seed dataset from the [week README](../README.md) — 20 users, 8 channels, 31 touches, 59 funnel events, 10 conversions, $4,500 of spend. Your job is to turn that into a defensible answer to "where should we spend the next dollar," and to show your work.

---

## Deliverable

A directory in your portfolio `c38-week-02/mini-project/` containing:

1. `queries.sql` — every query behind your report, organized under section comments (`-- Funnel`, `-- Attribution`, `-- CAC`).
2. `report.md` — the actual report: the funnel, the attribution comparison table, the CAC table, and your written recommendation (see structure below).
3. `notes.md` — a short reflection (see the end).

Everything runs against the seed tables from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used and flag any engine-specific rewrites (interval arithmetic, `FILTER`).

---

## What `report.md` must contain

### Section 1 — The funnel

The full `site_visit → signup → activated → paid_conversion` funnel: user counts, step-over-top rate, and step-over-previous rate at every step (Exercise 1's query, extended). State in one sentence which single step loses the most people, in absolute terms and in percentage terms, and whether those are the same step.

### Section 2 — Attribution, four ways

A table with one row per channel and four columns: first-touch revenue, last-touch revenue, linear revenue, position-based revenue. Every column must reconcile to $1,590 (or note and explain any deliberate exception). Below the table, name the channel with the single largest swing between its highest and lowest column, and explain in 2–3 sentences what that swing means for how the channel should be evaluated.

### Section 3 — CAC, not in isolation

A table with one row per **paid** channel: total spend, first-touch CAC, last-touch CAC, and a saturation note (does the channel's spend curve show diminishing touches/conversions over the 10-day window, yes/no, with the numbers). Flag any channel whose last-touch conversions have an average upstream path length greater than 1 (Lecture 3 §3's check) — that's your signal a channel may be reshuffling demand rather than creating it.

### Section 4 — Recommendation (the point of the whole project)

400–600 words, written for Crunch's growth lead, who will act on it. Must include:

- A specific recommendation on the **next acquisition dollar**: which channel gets more spend, which gets held flat, which gets cut, and why — citing specific numbers from Sections 1–3, not vibes.
- At least one explicit acknowledgment of a channel that looks better or worse than it "really" is depending on which model you trust, and which model you're choosing to weight more heavily for this decision, and why.
- A one-sentence caveat about what this recommendation would need (more data, a longer window, a controlled test) to become higher-confidence than "best read of ten days of seed data."

---

## Milestones

- **Milestone 1 (45 min):** Section 1. Build and verify the funnel — reuse and extend Exercise 1.
- **Milestone 2 (60 min):** Section 2. Build all four attribution models — reuse Exercise 2 and Challenge 1's linear/position-based queries; reconcile every column.
- **Milestone 3 (45 min):** Section 3. Build the CAC report — reuse Exercise 3; add the saturation and upstream-path checks from Lecture 3.
- **Milestone 4 (30–45 min):** Section 4. Write the recommendation. Draft it, then re-read Sections 1–3 and cut any sentence that isn't backed by a number sitting above it in the same report.

---

## Rules

- **Every number in `report.md` must trace to a query in `queries.sql`.** No numbers pulled from memory of the lecture notes.
- **Every attribution column must reconcile to $1,590**, or you must explain, inline, why it doesn't (only Section 2's "assist"-style variant, if you choose to include one, is allowed to skip reconciliation — and it must say so explicitly).
- **The recommendation must name specific channels and specific numbers.** "Paid channels are performing well" is not a recommendation.
- **Use `NULLIF` for any spend ÷ conversions computation** where a zero-conversion channel is possible.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Funnel correctness | 20% | Counts and both rate types match the verified values; drop-off correctly identified |
| Attribution correctness | 25% | All four models implemented correctly and reconciled to $1,590 |
| CAC + saturation/reshuffling analysis | 25% | CAC computed under two models; saturation and upstream-path checks both present with real numbers |
| Recommendation quality | 20% | Specific, numbers-backed, acknowledges model disagreement, states a caveat |
| Reproducibility | 10% | `queries.sql` runs top to bottom and produces every number in `report.md` |

---

## Reflection (`notes.md`, ~200 words)

1. Which of the four attribution models changed your recommendation the most, and why?
2. Where did you almost report a number that didn't reconcile — and what caught it?
3. If Crunch gave you 90 more days of real data instead of this 10-day seed, which part of your analysis would you trust least as-is, and what would you want to re-check first?
4. One question this week's data *couldn't* answer that you'd want a warehouse to answer next (foreshadows Week 3 — activation and onboarding, where the funnel keeps going past `paid_conversion`).

---

## Why this matters

This is the shape of real growth analytics: funnel, attribution, spend, and a recommendation someone will actually act on — in that order, each step's numbers feeding the next. Get comfortable building this exact four-part report from a raw events schema and you can walk into any growth team's data warehouse and be useful in a week, not a quarter.

When done: push, then take the [quiz](../quiz.md) and start Week 3 — Activation & onboarding.
