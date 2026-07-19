# C38 · Crunch Scale

> A free, open-source 12-week course to accelerate growth through AI, customer intelligence, and modern revenue systems — funnels, retention, RevOps, experimentation, and pricing on a SQL data warehouse.

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![PostgreSQL · SQLite](https://img.shields.io/badge/stack-PostgreSQL_·_SQLite_·_pandas-2463EB.svg)](#stack)
[![Built in the open](https://img.shields.io/badge/built-in%20the%20open-2463EB.svg)](https://github.com/CODECRUNCHWORLDWIDE)

C38 is the growth-and-revenue course for people who refuse to guess. It takes you from "what is a north-star metric?" to running a company's growth on a warehouse you built — funnels, cohorts, LTV/CAC, experiments, pricing, and AI-driven lifecycle, every number defended in SQL. It assumes the database skills from [C33 Crunch SQL](../C33-CRUNCH-SQL/) and turns them on the hardest question a business asks: *where does durable growth actually come from, and how do we prove it?*

---

## Pathway summary

- **Full-time:** 12 weeks · ~28 hrs/week · ~336 hours
- **Working-operator pace:** 6 months · ~14 hrs/week
- **Evening pace:** 12 months · ~7 hrs/week

See [`SYLLABUS.md`](SYLLABUS.md).

---

## What you will be able to do at the end of 12 weeks

- **Define growth honestly:** pick a north-star metric, map the AARRR funnel, and instrument events into a warehouse instead of a dashboard you can't audit.
- **Measure acquisition:** build funnel-conversion and multi-touch attribution models in SQL, and separate channels that scale from channels that flatter.
- **Fix activation:** find the activation moment and time-to-value, and turn a leaky onboarding funnel into a step-by-step SQL diagnosis.
- **Understand retention:** build cohort-retention tables and survival curves, tell resurrection from real stickiness, and read the shape of a retention curve.
- **Own unit economics:** model LTV, CAC, payback, and contribution margin — and know when a "growing" business is quietly unprofitable.
- **Run RevOps on a warehouse:** model the customer data stack (events → staging → marts) in SQL, never in a spreadsheet, and make one source of truth everyone trusts.
- **Segment with intelligence:** RFM, behavioral, and clustering-based segments in SQL + pandas that drive real targeting, not vanity personas.
- **Experiment like a scientist:** design A/B tests, size them, compute significance, and avoid the peeking, novelty, and Simpson's-paradox traps.
- **Price and package:** model price sensitivity, tiering, and packaging, and forecast the revenue impact of a pricing change before you ship it.
- **Personalize the lifecycle with AI:** score churn and propensity, build recommendation and next-best-action logic, and forecast growth under multiple scenarios.

---

## Curriculum (12 weeks)

| Week | Topic | You leave able to… |
|------|-------|--------------------|
| 1 | [Growth foundations & metrics](curriculum/week-01-growth-foundations-and-metrics/) | Pick a north-star, map AARRR, and land events in a warehouse. |
| 2 | [Acquisition funnels & attribution](curriculum/week-02-acquisition-funnels-and-attribution/) | Build funnel + multi-touch attribution models in SQL. |
| 3 | [Activation & onboarding](curriculum/week-03-activation-and-onboarding/) | Find the activation moment and fix a leaky onboarding funnel. |
| 4 | [Retention & cohort analysis](curriculum/week-04-retention-and-cohort-analysis/) | Build cohort-retention tables and read retention curves. |
| 5 | [LTV, CAC & unit economics](curriculum/week-05-ltv-cac-and-unit-economics/) | Model LTV, CAC, payback, and contribution margin. |
| 6 | [RevOps & the customer data stack](curriculum/week-06-revops-and-the-customer-data-stack/) | Model events → staging → marts as one warehouse truth. |
| 7 | [Segmentation & customer intelligence](curriculum/week-07-segmentation-and-customer-intelligence/) | Build RFM, behavioral, and clustering segments. |
| 8 | [Experimentation & A/B testing](curriculum/week-08-experimentation-and-ab-testing/) | Design, size, and read an A/B test without fooling yourself. |
| 9 | [Pricing & packaging](curriculum/week-09-pricing-and-packaging/) | Model price sensitivity and tiering; forecast a price change. |
| 10 | [Lifecycle & AI personalization](curriculum/week-10-lifecycle-and-ai-personalization/) | Score churn/propensity and drive next-best-action logic. |
| 11 | [Forecasting growth](curriculum/week-11-forecasting-growth/) | Forecast revenue and run scenario models in SQL + pandas. |
| 12 | [Capstone — the growth system](curriculum/week-12-capstone-growth-system/) | Ship an end-to-end growth + revenue system on a warehouse. |

---

## How to navigate a week

Every week folder holds the same structure:

- **`README.md`** — the week overview + how the pieces fit + the week's goal.
- **`lecture-notes/`** — 3 lectures (~2 hrs each), the conceptual core.
- **`exercises/`** — 3 short, guided reps against a real warehouse.
- **`challenges/`** — 2 open-ended problems with no single right answer.
- **`mini-project/`** — one build that ties the week together.
- **`homework.md`**, **`quiz.md`**, **`resources.md`** — practice, self-check, and further reading.

---

## Stack

PostgreSQL (16+) is the primary warehouse engine, with SQLite for zero-setup practice and Python + pandas for modeling, stats, and forecasting. **This course never uses Excel or a spreadsheet as a database.** Growth and finance workflows that a company would traditionally run in a spreadsheet — cohort grids, LTV/CAC models, funnel math, forecasts — are built here in SQL and pandas instead, because a warehouse is auditable, version-controlled, and reproducible where a spreadsheet is none of those. Spreadsheets are taught only in [C41 Crunch Excel](../C41-CRUNCH-EXCEL/). Everything is free and runs on macOS, Linux, and Windows, with a seed events + revenue dataset shipped with the course.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · [Browse all courses](https://codecrunchglobal.vercel.app/courses)*
