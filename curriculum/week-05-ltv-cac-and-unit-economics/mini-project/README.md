# Mini-Project — Two-Channel Unit Economics Model

> Model the unit economics of Lumen Metrics — a subscription SaaS business — for its two growth channels, `paid_search` and `organic_content`, entirely in SQL and pandas. Compute contribution margin, LTV, fully-loaded CAC, payback, and the LTV:CAC ratio for each. Then write a scale/pause/fix recommendation a founder could act on Monday morning.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the same deliverable a growth or finance analyst produces every quarter at a real subscription company: not a dashboard, not a deck full of logos — one defensible model and one clear written call. You've built every piece of this already, in the exercises and challenges. The mini-project asks you to assemble it into one coherent artifact and stand behind a recommendation.

---

## Deliverable

A directory in your portfolio `c38-week-05/mini-project/` containing:

1. `model.sql` — every SQL query behind the model: new customers per channel/month, the pooled retention curve, fully-loaded CAC (blended and trailing), and any view(s) you build.
2. `model.py` — the pandas layer: your `unit_economics()`-style function (or equivalent), applied to both channels, producing the final comparison table.
3. `report.md` — the comparison table, every number's method labeled (which LTV method, which CAC basis), and your written recommendation.
4. `notes.md` — a short reflection (see the end).

Everything runs against the seed from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used.

---

## What the model must include

Build **one comparison table**, one row per channel, with at minimum:

- `mrr`, `margin_per_month` (80% gross margin)
- `avg_monthly_churn` (from the pooled retention curve, ages 1–6)
- `ltv_floor_6mo` (cohort-sum LTV — the honest, truncated floor)
- `ltv_projected` (churn-reciprocal LTV)
- `cac_blended` (fully-loaded, full year) and `cac_trailing` (fully-loaded, Q4)
- `ltv_cac_blended` and `ltv_cac_trailing`
- `naive_payback_months` and a stated answer to "does retention-adjusted payback ever complete, at this CAC?" (yes/no, with the month it crosses if yes)

Then, in `report.md`, answer these directly:

1. **Which channel is unit-profitable today**, on which CAC basis (blended or trailing)? Be exact — "marginal" and "unprofitable" are different verdicts and you should use them correctly per Lecture 3's classification.
2. **Is the company's revenue growth masking a unit-economics problem?** Use the company-wide MRR trend from Lecture 3 §1 alongside your channel table to answer this with numbers, not a hunch.
3. **If you could move next quarter's marketing dollar to exactly one channel, which one, and why?** Your answer must reference the ratio, its trend (improving/flat/worsening), and at least one number from your own model — not a general statement about paid vs. organic acquisition.
4. **What would have to be true for the *other* channel to become worth scaling?** Name a specific, testable change (e.g., a churn rate, a CAC ceiling) — not "if it got better."

---

## Milestones

Pace yourself; don't try to build the whole model in one sitting.

- **Milestone 1 (40 min):** `model.sql` — new customers per channel/month, the retention curve query, and the CAC queries (blended + trailing). Confirm every number against your Exercise 1/2 answers before moving on.
- **Milestone 2 (40 min):** `model.py` — build the reusable unit-economics function and run it for both channels under both CAC scenarios. Assemble the final comparison `DataFrame`.
- **Milestone 3 (50 min):** `report.md` — write the table and answer the four required questions. This is where the judgment happens; don't rush it.
- **Milestone 4 (30–40 min):** `notes.md` reflection, plus a final pass checking every number in `report.md` traces back to `model.sql`/`model.py`.

---

## Rules

- **Two channels only.** No inventing a third channel or additional data — everything comes from `customers` and `channel_costs`.
- **Label every LTV and CAC number with its method.** A bare number with no method stated is treated as unsupported, same as in the challenges.
- **Contribution margin, not revenue, drives every LTV figure.** Any calculation using raw `mrr` in place of `mrr * 0.80` is marked wrong, on principle, even if the arithmetic is correct.
- **State your CAC choice for the recommendation explicitly** (blended vs. trailing) and defend it in one sentence — this is the same judgment call Lecture 2 walked through.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 35% | Every number in `report.md` matches what `model.sql`/`model.py` actually produce |
| Method transparency | 20% | Every LTV and CAC figure states its method; blended vs. trailing is never conflated |
| Recommendation quality | 25% | The scale/pause/fix call follows directly from the model, addresses the trend (not just the snapshot), and answers all four required questions |
| Code quality | 10% | `model.sql` and `model.py` are readable, commented, and reproducible from the seed alone |
| Reflection | 10% | `notes.md` shows genuine engagement with where the model is weakest |

---

## Reflection (`notes.md`, ~200 words)

1. Where is your model weakest — which number are you least confident in, and why (small sample size, an assumption, a method choice)?
2. If Lumen Metrics gave you three more months of data, which specific number would you expect to change the most, and in which direction?
3. Which felt more natural to build the comparison table in — SQL or pandas — and why? Where did you reach for the other one instead?
4. One question this model *can't* answer with two channels and one year of data — what would you need to answer it? (Foreshadows Week 6, where this model gets fed by a real warehouse pipeline instead of a hand-loaded seed.)

---

## Why this matters

This is the artifact that ends a growth argument with a number instead of a vibe. "Our MRR is up 836%" and "our best channel returns $1.26 for every dollar we put into it, and it's improving" are both true about the same company at the same time — and only the second sentence tells a founder what to do with next quarter's budget. Keep this model; Week 6 plugs it into a real warehouse mart so it updates itself instead of being rebuilt by hand every quarter.

When done: push, then take the [quiz](../quiz.md) and start [Week 6 — RevOps & the Customer Data Stack](../../week-06-revops-and-the-customer-data-stack/).
