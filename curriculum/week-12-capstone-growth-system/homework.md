# Week 12 — Homework

Five problems, ~4.5 hours total, spread across the week. These stress-test the mini-project's assumptions, add a plan-tier cut of the data none of the exercises built, force you to prove the warehouse handles a new channel without a redesign, and let you find out — with real numbers — exactly how much sample size the checklist experiment actually needed.

All queries run against the `crunch_boards` seed database and the `stg_*`/`dim_*`/`fct_*` objects you built in the exercises, unless a problem says to add something new.

---

## Problem 1 — Stress-test the margin assumption (45 min)

Every LTV number this week assumed a **75% gross margin**. Real early-stage SaaS companies often run leaner — recompute at **60%**.

1. Recompute `contribution_margin_per_month` and `ltv_cohort_sum` for all three channels at 60% gross margin, using the same retention curve from Exercise 2.
2. Compare against the 75% figures side by side:

   | Channel | LTV @ 75% | LTV @ 60% | LTV:CAC @ 75% | LTV:CAC @ 60% |
   |---|---:|---:|---:|---:|
   | `paid_search` | $165.93 | ? | 0.08 | ? |
   | `organic` | $206.43 | ? | 0.17 | ? |
   | `referral` | $555.60 | ? | 2.47 | ? |

   *(Expected `referral` LTV @ 60%: **$444.50**, LTV:CAC @ 60%: **1.98**.)*

3. Does the channel-budget recommendation from Lecture 3 / Challenge 2 survive at 60% margin, or does it flip for any channel? Be specific about which channel, if any, crosses a decision-relevant threshold (1:1 or 3:1) between the two margin assumptions.

**Deliver** `margin-stress.py` (or `.sql`) and a one-paragraph `margin-stress.md` answering Task 3.

---

## Problem 2 — Cut unit economics by plan tier, not just channel (75 min)

Every mart this week grained by `signup_channel`. Plan tier (`Starter`/`Growth`/`Scale`) is a completely different, and equally valid, cut of the same paying-customer base — and nothing this week built it.

1. Using `stg_subscriptions` (cutoff already applied), compute, per `plan_name`: count of subscriptions, count active, count canceled, and **logo churn rate** (`canceled / (active + canceled)`, following the definition style from Week 6's homework).
2. You should land on: `Growth` — 16 subs, 10 active, 6 canceled (**37.5%** logo churn); `Starter` — 12 subs, 10 active, 2 canceled (**16.7%** logo churn); `Scale` — 2 subs, 2 active, 0 canceled (**0%**, but flag the sample size).
3. This is counterintuitive — the *most expensive* plan (`Growth`, $99/mo) churns more than twice as often as the *cheapest* paying plan (`Starter`, $29/mo). In 3–4 sentences, propose two plausible business explanations for why a mid-tier plan might churn harder than an entry-level one (this dataset doesn't tell you *why* — reasoning about plausible causes from a real pattern is the actual skill), and name the one additional raw data source (not in this week's seed) that would let you actually test one of your explanations.

**Deliver** `plan-tier-churn.sql` and `plan-tier-churn.md` with your Task 3 write-up.

---

## Problem 3 — Add a fourth channel and prove the warehouse doesn't need a redesign (60 min)

A real Crunch Boards would eventually launch a new acquisition channel. This problem asks you to prove your marts handle that without you rewriting anything.

1. Invent a fourth channel, `partnerships`, and `INSERT` 6 new `raw_users` rows (user_ids 73–78) signing up in July 2025, plus matching `raw_events` (at least 4 activate) and `raw_subscriptions` rows (at least 2 convert) and a `raw_marketing_spend` row for July. Make up realistic values — you're the one authoring this scenario.
2. Extend `dim_date` with a `2025-07-01` row (`is_forecast = FALSE` now, since you're back-filling real July data) if it isn't already there from Lecture 1.
3. Re-run `fct_acquisition`, `fct_mrr_monthly`, and `fct_unit_economics` (your `build.sql` from Challenge 1, if you built one) and confirm `partnerships` shows up correctly in all three **without modifying a single line of SQL** — only new rows, no schema or query changes.
4. If any mart *doesn't* pick up the new channel automatically, that's a real design flaw (probably a hardcoded channel list somewhere). Fix it, and note what you fixed and why in your write-up.

**Deliver** `new-channel.sql` (the inserts) and `new-channel.md`: what you invented, the resulting `partnerships` row in each mart, and whether anything needed fixing.

---

## Problem 4 — How much sample size did the checklist experiment actually need? (60 min)

Exercise 3 found `onboarding_checklist_v2`'s 25-point lift wasn't significant at n=12 per arm (p≈0.132). This problem finds out exactly how much more data would have settled it.

1. Simulate **doubling** the experiment: duplicate the 24 assignment rows once (same 66.7%/91.7% activation rates, now n=24 per arm: 16/24 control, 22/24 treatment). Recompute the two-proportion z-test. *(Expected: z≈2.132, p≈0.033 — now significant at α=0.05.)*
2. Simulate **quadrupling** it (n=48 per arm: 32/48 control, 44/48 treatment). Recompute again. *(Expected: z≈3.016, p≈0.0026.)*
3. Compare this to your Exercise 3, Task 4 power-analysis estimate of the minimum sample needed. Did the doubled sample (n=24/arm) cross the significance line at roughly the sample size your power analysis predicted, or did it happen earlier/later than expected? If your numbers don't line up closely, say what you think explains the gap (power analysis is a probabilistic estimate, not a guarantee — a single simulated dataset crossing the line a bit earlier or later than the "expected" sample size is normal, not a sign your math was wrong).

**Deliver** `power-check.py` and a short `power-check.md` with the Task 3 comparison.

---

## Problem 5 — Write the missing metric definitions (45 min)

`metrics.md` from the mini-project covered the cutoff, activated user, paying customer, MRR, CAC, LTV, and the experiment's success metric. This problem extends it.

Write full entries, in the exact plain-language + source-table format from Lecture 1/2, for:

1. **Logo churn rate** (from Problem 2) — definition, source table, and the one limitation worth naming (small `n` on `Scale`).
2. **LTV:CAC ratio** — formalize it as its own named metric (not just a derived column), stating explicitly which LTV method (cohort-sum vs. reciprocal) it's built on and why.
3. One metric of your own choosing that a real Crunch Boards team would ask for next (pick from: net-new MRR, quick ratio, expansion revenue, time-to-first-value) — you don't need a mart that computes it yet; writing a precise definition *before* the data exists is exactly the RevOps skill Week 6 introduced, and it's worth practicing one more time here at the end of the course.

**Deliver**: append all three to your `metrics.md` from the mini-project.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 75 min |
| 3 | 60 min |
| 4 | 60 min |
| 5 | 45 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md) if you haven't already — that's the whole course, finished.
