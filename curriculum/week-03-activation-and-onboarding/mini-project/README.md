# Mini-Project — Diagnose a Leaky Onboarding Funnel

> Bring the whole week together into one deliverable: define Crunch Boards' activation event, find the aha moment empirically, diagnose the funnel's worst step, measure time-to-value, and quantify the retention lift a proposed onboarding fix would deliver.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it mirrors exactly what a growth analyst delivers in their first real diagnostic project: a stakeholder doesn't ask for "a query" — they ask "why are we losing people, and what should we do about it?" Your job is to answer that in one coherent report, with every claim backed by a query you can point to.

---

## Deliverable

A directory in your portfolio `c38-week-03/mini-project/` containing:

1. `schema.sql` — the `CREATE TABLE` statements you used (from the week README), for the record.
2. `analysis.sql` — every query behind every number in your report, grouped under `-- Section N` comments matching the sections below.
3. `report.md` — the full write-up, following the five sections below.
4. `notes.md` — a short reflection (see the end).

Everything runs against this week's seed (`generate_seed.py`, seed `38`). Works on PostgreSQL or SQLite; note which you used.

---

## The five sections

### Section 1 — Define the activation event

State Crunch Boards' activation event in the exact shape from Exercise 1: *"A user is activated when they `<event>` within `<window>` of signup."* Support it with the reach table (unbounded and windowed) from Exercise 1 and the criteria table from Lecture 1.

### Section 2 — Find the aha moment empirically

Report the full six-candidate lift table from Challenge 1 (`verified_email` through `connected_integration`): reach, retained-if-did, retained-if-not, lift. State which candidate you selected and why, explicitly addressing the reach tradeoff (§1 of Challenge 1) and the reverse-causation risk (§2 of Challenge 1) for the two next-best candidates.

### Section 3 — Diagnose the funnel

Report the six-step funnel with step-over-step conversion (Exercise 3), and name the single worst transition with its exact drop percentage. Include the sanity-check arithmetic (cumulative reach = product of step conversions) from Exercise 3 Task 4.

### Section 4 — Measure time-to-value

Report the TTV distribution (min/p25/median/p75/max/mean, and the four buckets) for your chosen activation event, from Exercise 2. State in one sentence what the shape implies for *when* a product team should intervene.

### Section 5 — Propose a fix and quantify the projected lift

Using Challenge 2's method: propose your redesign, state your target conversion-rate assumption, project the new inviter count and the new overall week-4 retention rate, and report the 3-point sensitivity range. Close with the honesty caveat — this is a projection, not a proof, and name the Week 8 experiment that would validate it.

---

## Milestones

Pace yourself; don't try to do all five sections in one sitting.

- **Milestone 1 (30 min):** Section 1 — pull the reach numbers, fill in the criteria table, write the definition.
- **Milestone 2 (45 min):** Section 2 — run the full six-candidate lift query, write the comparison.
- **Milestone 3 (30 min):** Section 3 — funnel + worst step + sanity check.
- **Milestone 4 (30 min):** Section 4 — TTV distribution + shape interpretation.
- **Milestone 5 (45 min):** Section 5 — proposal, target rate, projection arithmetic, sensitivity range, caveat.

---

## Rules

- **Every number in `report.md` must trace to a query in `analysis.sql`.** No hand-typed statistics, even ones you remember correctly from the lectures — run them fresh against your own loaded seed and confirm they match.
- **Section 2 must include all six candidates**, not just the winner — the comparison *is* the evidence.
- **Section 5's projection must show the arithmetic**, not just a final percentage. Grading looks for the work, not just the answer.
- **The caveat in Section 5 is mandatory**, not optional flavor text — a report that presents a projection as a guarantee is graded down regardless of how good the rest of the analysis is.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|---|---:|---|
| Correctness | 30% | All numbers match what the seed actually produces; SQL runs clean |
| Activation-event reasoning | 20% | Section 1–2 weigh reach, lift, and causality, not just the biggest lift number |
| Funnel diagnosis | 15% | Worst step correctly identified with the sanity-check arithmetic shown |
| TTV interpretation | 10% | States the distribution's shape, not just its median |
| Projection quality | 15% | Section 5's arithmetic is correct, the target rate is justified, and a sensitivity range is given |
| Honesty | 10% | The projection is explicitly labeled as unproven, with the validating experiment named |

---

## Reflection (`notes.md`, ~200 words)

1. Which section forced you to think hardest, and why?
2. Where did you almost fall into the reach trap, the reverse-causation trap, or the window-leakage trap — even briefly?
3. If you had one more week of data (a longer observation window), what would you want to re-check?
4. What would it take to turn Section 5's projection into an actual, defensible number? (Foreshadows Week 8 — Experimentation & A/B Testing.)

---

## Why this matters

This is the exact shape of a real activation diagnosis: pick the metric, find what predicts it, locate where it breaks, measure how long value actually takes to arrive, and put a number — an honest, caveated number — on the fix. Keep `analysis.sql`; Week 4 (retention & cohort analysis) extends this same seed data with the full cohort-retention machinery this week's `week4_active` flag only sketched.

When done: push, then take the [quiz](../quiz.md) and start [Week 4 — Retention & cohort analysis](../../week-04-retention-and-cohort-analysis/).
