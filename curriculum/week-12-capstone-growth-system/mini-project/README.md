# Mini-Project — Ship the Growth System

> Assemble everything this week (and, honestly, this entire course) built — warehouse, funnel, retention, unit economics, the checklist experiment, and a defended revenue forecast — into one submission: a running pipeline plus a decision memo a founder could act on tomorrow morning.

**Estimated time:** 5–6 hours, best spread across Saturday and a Sunday-morning polish pass.

This is not a bigger version of a weekly mini-project. It's the deliverable the other eleven weeks of **C38 · Crunch Scale** were preparing you to produce: a real growth system, on real (if fictional) data, defended end to end. Treat it accordingly — this is the piece of work from this course you're most likely to show an interviewer.

---

## Deliverable

A directory in your portfolio `c38-week-12/mini-project/` containing:

1. **`build.sql`** — the complete, idempotent warehouse build from Challenge 1: all `stg_*` views, `dim_date`, `dim_user`, `fct_acquisition`, `fct_mrr_monthly`, `fct_unit_economics`, `fct_experiment_results`, and `vw_retention_cohorts`. Run twice, both row-count outputs pasted as a comment at the top of the file — your idempotency proof.
2. **`forecast.py`** — all four forecast methods from Challenge 2 (the three from Lecture 3 plus your own), computing July–September 2025 MRR, printed as a comparison table.
3. **`metrics.md`** — the completed metric-definitions glossary you started in Lecture 1 and grew across Lectures 2–3. At minimum: analysis cutoff, activated user, paying customer, MRR, fully-loaded CAC (noting the flat-vs-variable spend distinction), LTV (cohort-sum, with the reciprocal-formula caveat for `referral`), and the experiment's success metric with its significance result.
4. **`experiment-writeup.md`** — from Exercise 3: the lift, the significance test, the minimum-detectable-sample-size estimate, and your ship/hold/extend recommendation.
5. **`decision-memo.md`** — the actual capstone deliverable. Follows the four-part shape from Lecture 3, Section 4: the ask, the evidence (funnel + unit economics + experiment + forecast, each traced to a table), the uncertainty (name every thin sample, every broken formula assumption, every forecast disagreement), and the checkpoint (what number, checked again in a month, confirms or kills the recommendation). **400–600 words.** This is the file a non-technical reader opens first — it should stand alone without requiring them to read your SQL.
6. **`notes.md`** — a short reflection (see the end).

Everything runs against the `crunch_boards` seed database from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used and call out every place your SQL diverges between the two engines.

---

## What you're building, end to end

```
raw_users, raw_events,                stg_users, stg_events,             dim_date, dim_user,
raw_marketing_spend,           ──▶     stg_subscriptions,           ──▶  fct_acquisition,
raw_subscriptions,                    stg_marketing_spend,               fct_mrr_monthly,
raw_experiment_assignments            stg_experiment_assignments         fct_unit_economics,
                                                                          fct_experiment_results,
                                                                          vw_retention_cohorts
                                                                                │
                                                                                ▼
                                                            forecast.py ──▶ decision-memo.md
```

If a future teammate — or you, in six months — runs `build.sql` twice, `fct_mrr_monthly`'s June row must contain **exactly $1,878.00 across 22 active customers**, both times. If they run your significance test, it must reproduce `z≈1.508, p≈0.132`. If they read `decision-memo.md` cold, with no other context, they should come away knowing exactly what you're recommending and exactly which query proves each claim.

---

## Milestones

Pace yourself; don't try to do this in one sitting.

- **Milestone 1 (60 min):** Finalize `build.sql` from Challenge 1. Run it twice, confirm identical row counts, paste the proof.
- **Milestone 2 (45 min):** Finalize `forecast.py` from Challenge 2 — all four methods, one comparison table, July–September 2025.
- **Milestone 3 (45 min):** Finalize `metrics.md` — read it cold as if you'd never seen this dataset; if a definition requires outside context to understand, rewrite it.
- **Milestone 4 (45 min):** Finalize `experiment-writeup.md` from Exercise 3.
- **Milestone 5 (90 min):** Write `decision-memo.md`. Draft it once, then re-read every sentence and ask "what table proves this?" — if you can't answer immediately, either add the query reference or cut the sentence.
- **Milestone 6 (30 min):** `notes.md`, then a final pass: does everything in the deliverable list actually exist, in the right directory, and does `build.sql` still run clean from a fresh database?

---

## Rules

- **Everything is SQL or pandas.** No spreadsheet, at any point, for any intermediate calculation, cross-check, or "scratch work" — this course's hard rule, unchanged for the capstone.
- **The pipeline must be idempotent.** Prove it in `build.sql`'s header comment with the twice-run row counts.
- **Every raw table stays untouched.** No `UPDATE`/`DELETE` against `raw_*` anywhere in your submission.
- **Every number in `decision-memo.md` traces to a table or a script in this submission.** If a reader can't find where a number came from, it doesn't belong in the memo.
- **State uncertainty as plainly as you state the recommendation.** A memo that hides `referral`'s small sample or the experiment's non-significant p-value to make the story cleaner is graded down, not up — the whole point of this course is numbers you can defend under a follow-up question, not numbers that only survive if nobody asks one.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Pipeline correctness & idempotency | 25% | All figures match exactly (June MRR $1,878.00/22 active; experiment z≈1.508); two-run proof genuinely identical |
| Metric documentation | 15% | `metrics.md` reads standalone; every metric names its source table and at least one real limitation |
| Experiment rigor | 15% | Significance correctly computed and correctly interpreted (not overclaimed); sample-size estimate included |
| Forecast reconciliation | 15% | 4 methods shown, disagreement quantified, one number chosen with stated (not vibes-based) reasoning |
| Decision memo | 25% | Ask/evidence/uncertainty/checkpoint all present; every claim traceable; a non-technical reader could act on it |
| Readability & craft | 5% | Consistent formatting, meaningful object names, comments on non-obvious `WHERE`/`JOIN` conditions |

---

## Reflection (`notes.md`, ~250 words)

1. Which single number in your `decision-memo.md` were you most tempted to overstate — round a "promising but not significant" result up to "it works," or round a thin-sample LTV up to a confident one — and what stopped you (or didn't)?
2. If Crunch Boards handed you three more months of real data tomorrow, which of this capstone's numbers would you expect to change the most, and which would you expect to barely move? Why?
3. Across all twelve weeks of this course, which single skill from an earlier week did this capstone lean on hardest — and which one did you have to actively go back and re-read to use correctly here?
4. If you were handed a *different* company's data next month and asked to run this same capstone process on it, what would you build in the same order, and what would you do differently from the start, given what this week taught you?

---

## Why this matters

Every earlier week of this course taught you one instrument in a growth team's toolkit. This week you played the whole set, on one dataset, in front of an audience that (in a real job) will ask you follow-up questions you can't dodge. The habit this capstone is actually building isn't "I can compute LTV" or "I can run a significance test" — those were done by Week 8. It's **"I can turn a warehouse full of numbers into a recommendation someone will act on, and defend every part of it live."** That is the job title this course has been training you for the entire time, whatever it's called at the company that hires you.

When done: push, then take the [quiz](../quiz.md). You've finished **C38 · Crunch Scale**.
