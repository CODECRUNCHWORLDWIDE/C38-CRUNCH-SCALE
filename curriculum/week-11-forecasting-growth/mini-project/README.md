# Mini-Project — A Next-Four-Quarter MRR Forecast, Built Three Ways

> Build Crunch Flow's Q1–Q4 2026 MRR forecast the way a real RevOps/FP&A team would: bottoms-up bridge, retention-adjusted cohort cross-check, and base/bull/bear scenarios — all in SQL and pandas, never a spreadsheet — and write the assumption memo that defends the number.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone. Finance doesn't want a chart — they want a number, a range around that number, and a paper trail showing exactly which assumptions produced it. You'll build all three forecasting methods this week taught, cross-check them against each other, and write the memo that would actually survive being read out loud in a board meeting.

---

## Deliverable

A directory in your portfolio `c38-week-11/mini-project/` containing:

1. `bridge_forecast.sql` and `bridge_forecast.py` — the bottoms-up base-case forecast (Part A).
2. `cohort_projection.py` — the retention-adjusted cross-check (Part B).
3. `scenarios.py` — the base/bull/bear model (Part C).
4. `forecast_report.md` — the consolidated quarterly table plus your interpretation (Part D).
5. `assumption_memo.md` — the written memo (Part E) — this is the piece a stakeholder actually reads.

Everything runs against the seed tables from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used.

---

## Part A — Bottoms-up bridge forecast (45 min)

Using the method from Lecture 1:

1. Compute the trailing-3-month (Oct–Dec 2025) average expansion, contraction, and churn rates.
2. Fit a linear trend to `new_mrr` and derive a January 2026 starting value and monthly growth increment.
3. Roll the bridge forward twelve months (Jan–Dec 2026), anchored on December 2025's actual `ending_mrr`.
4. Produce a table with one row per month: `starting_mrr`, `new_mrr`, `expansion_mrr`, `contraction_mrr`, `churned_mrr`, `ending_mrr`.

Do this **once in SQL** (a `WITH RECURSIVE` query) and **once in pandas** (a loop). They should agree to within rounding — if they don't, find the bug before moving on.

## Part B — Retention-adjusted cross-check (40 min)

Using the method from Lecture 2:

1. Build the weighted, curve-segmented (legacy vs. new-onboarding) cohort revenue-retention curves.
2. Determine each curve's flat-continuation value.
3. Project every 2025 cohort (Jan–Dec, using `cohort_revenue_retention` where available and `mrr_bridge_actuals.new_mrr` for the cohorts with no retention rows yet) forward to **December 2026**, and sum their contributions.
4. State, as a percentage, how much of your Part A December 2026 forecast this "already exists today" figure covers — and one paragraph on whether that split (baked-in vs. still-to-be-sold) looks reasonable for a company at Crunch Flow's growth stage.

## Part C — Base/bull/bear scenarios (40 min)

Using the method from Lecture 3:

1. Build base, bull, and bear scenarios. You may reuse Exercise 3's assumption table, **or** propose your own — if you change any assumption from Exercise 3, state the specific reason in `assumption_memo.md` (a changed number with no stated reason will cost you on the rubric below).
2. Produce the quarterly ending-MRR table for all three scenarios (4 quarters × 3 scenarios = 12 numbers).
3. Compute the bull-minus-bear spread for each quarter, in dollars and as a percentage of the base case.
4. Fit the simple linear-trend model from Lecture 3 §1–3 to the 2025 actuals, and compute its own Q4 2026 point forecast and ±1-RMSE interval. State, in one sentence, whether it falls inside or outside your bear/bull range — and if it falls outside, say why that's expected (or isn't).

## Part D — Consolidated forecast report (20 min)

In `forecast_report.md`, present one table with a row per quarter (Q1–Q4 2026) and columns: `bear`, `base`, `bull`, `retention_adjusted_check` (Part B's cumulative baked-in figure, where computable per quarter), `linear_trend_point` (Part C §4). Follow it with 3–4 sentences answering: **do the three methods roughly agree, or does one stick out — and if one sticks out, which, and why do you think it does?**

## Part E — The assumption memo (25–30 min)

Write `assumption_memo.md` as if it's actually going to Finance. Follow the five-part shape from Lecture 3 §5:

1. **The headline number and method**, in one sentence.
2. **Every assumption, named as a number** — a short table is fine (and expected).
3. **The range** (bear→bull) and which one or two assumptions drive most of its width.
4. **The cross-check** — what the retention-adjusted view and the linear-trend view each say, and whether they support or challenge the bridge forecast.
5. **A stated invalidation condition** — a concrete, checkable thing that, if it happens in Q1 2026 actuals, means this forecast should be rebuilt before it's used again.

Keep it under 500 words. A memo that goes on for two pages doesn't get read; a memo this tight does.

---

## Rules

- **No spreadsheets, anywhere, for any part.** All computation lives in SQL and/or pandas. This is a hard rule of the course.
- **Every scenario assumption is a named number**, sourced to either "trailing-3-month average," "Exercise 3's given value," or a stated reason if you changed it. "Felt reasonable" is not a source.
- **Part A's SQL and pandas forecasts must agree.** This is your own internal reconciliation check, same spirit as Week 6's data tests — if they don't match, you have a bug, not two valid answers.
- **Round dollar figures to the nearest whole dollar** in every table; don't report forecast precision your method doesn't actually have.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Bridge forecast correctness (Part A) | 25% | SQL and pandas agree; base case reconciles month over month with no breaks |
| Retention cross-check (Part B) | 20% | Correct handling of right-censoring; curves correctly segmented; flat-continuation applied and justified |
| Scenario construction (Part C) | 20% | Bear/base/bull each internally consistent; any changed assumption is explicitly justified |
| Consolidated report (Part D) | 15% | All four methods land in one readable table; the "do they agree" question is actually answered with reasoning |
| Assumption memo (Part E) | 20% | Every number is named and sourced; the invalidation condition is concrete and checkable, not vague |

---

## Reflection (`notes.md`, ~200 words)

1. Where did the three methods (bridge, cohort, linear-trend) disagree the most, and what do you think that disagreement is actually telling you about the business?
2. Which single assumption, if it turns out wrong, would blow the biggest hole in your Q4 2026 forecast — and how would you find out early that it's wrong?
3. If you had to defend this forecast to a skeptical board member in one minute, what would you say first?
4. What's one thing you'd need — more data, more cohorts, more months of actuals — to make next quarter's version of this forecast meaningfully more confident?

---

## Why this matters

This is the shape of real forecasting work: not one clever model, but several honest, cross-checked ones, reconciled into a number someone can act on and a memo that survives questioning. The habits here — naming every assumption, quantifying uncertainty instead of hiding it, and stating what would prove you wrong — are what separate a forecast from a guess with a dollar sign on it. Keep this mini-project; the [Week 12 capstone](../../week-12-capstone/) builds a growth system that generates this exact forecast automatically, off a live warehouse, instead of by hand.

When done: push, then take the [quiz](../quiz.md) and start [Week 12 — Capstone](../../week-12-capstone/).
