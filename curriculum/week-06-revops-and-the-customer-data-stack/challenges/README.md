# Week 6 — Challenges

Two open-ended problems, ~90 minutes each. **No single right answer** — these are judged on the reasoning and the documentation, not just a query that runs.

1. **[Challenge 1 — Design a mart schema](challenge-01-design-a-mart-schema.md)** — sketch, in real DDL, the full mart layer this course's later weeks (funnels, retention, LTV, experiments) will build on.
2. **[Challenge 2 — Add data tests](challenge-02-add-data-tests.md)** — write a SQL test suite that fails loudly the moment the warehouse's data goes bad.

## Before you start

- Exercises 1–3 are complete: all four `stg_*` models, `dim_date`, `dim_user`, `fct_mrr_monthly`, and the reconciliation queries from Exercise 3.
- You've read Lecture 3 (RevOps hygiene) in full — both challenges lean directly on it.

## How these differ from the exercises

The exercises had exact expected row counts and dollar figures. These challenges don't — they ask you to make and justify a **design decision** the way a real RevOps or analytics-engineering hire would: propose a shape, defend it in writing, and be honest about the tradeoffs you didn't choose. A grader (a course reviewer, a hiring manager, a future teammate) should be able to read your `challenge-0N.md` and understand not just *what* you built but *why* you built it that way instead of three other reasonable ways.

## Submission

Each challenge asks for a `challenge-0N.md` (design doc) plus the SQL that implements it. Commit both to your portfolio under `c38-week-06/challenge-0N/`.
