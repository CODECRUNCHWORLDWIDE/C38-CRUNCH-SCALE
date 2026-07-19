# Mini-Project — A Full Pricing-Change Model for ScopeIQ

> Take ScopeIQ from a single flat $39 price to a fully-justified, fully-forecast pricing change — willingness to pay, tier design, and a 90-day revenue forecast that includes churn, a retention discount, **and** expansion revenue — entirely in SQL and pandas. Then write the recommendation memo a CEO would actually read before a board meeting.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the same deliverable a monetization/growth analyst produces before any real pricing change ships: not a single number, one coherent model with every assumption labeled, and one clear written call. You've built every piece of this already — Exercise 1 gave you the WTP curves, Exercise 2 gave you the tiers, Exercise 3 and Challenge 2 gave you the demand curve and the forecast mechanics. The mini-project asks you to assemble all three into one model and stand behind a recommendation, including the two levers (discounts, expansion revenue) the exercises deliberately left out so you'd meet them fresh here.

---

## Deliverable

A directory in your portfolio `c38-week-09/mini-project/` containing:

1. `model.sql` — every SQL query behind the model: the WTP curves and PMC/PME/OPP, the tier assignment and MRR rollup, and any aggregate queries against `subscription_base`.
2. `model.py` — the pandas layer: the elasticity fit, the churn/retention forecast function, the discount-mitigation adjustment, and the expansion-revenue calculation, assembled into one final comparison.
3. `report.md` — every table, every number's method labeled, and your written recommendation.
4. `notes.md` — a short reflection (see the end).

Everything runs against the seed from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used.

---

## What the model must include

### Part A — Willingness to pay (reuses Exercise 1)

- The blended PMC/PME/OPP for all 30 respondents, and the per-segment medians.
- One paragraph: why the blended numbers alone would have led ScopeIQ to a wrong, one-size-fits-all price.

### Part B — Tier design (reuses Exercise 2)

- The Starter/Growth/Scale assignment for all 30 `accounts_usage` rows, with the total MRR uplift.
- A one-sentence justification for each tier's price, citing its segment's WTP range from Part A.

### Part C — The price-change forecast, with the two levers the exercises skipped

Extend Challenge 2's forecast method to ScopeIQ's proposed `Growth` increase ($69→$79) with **both** of the following layered on top of the base churn/new-signup forecast from Lecture 3 §6:

1. **Discount mitigation.** Existing `Growth` customers who would otherwise churn from the increase can instead lock in **$759/year** (effective $63.25/month) by prepaying annually. Assume **25%** of the customers your base model predicts would churn take this offer instead of leaving. Recompute retained customers and existing-base MRR with this group included at their discounted rate.
2. **Expansion revenue.** Independent of the price change, `Starter` customers organically outgrow the 3,000 MTU cap and upgrade into `Growth` at a rate of **3% of the `Starter` base per month**. Over the same 90-day window, compute how many `Starter` customers expand into `Growth`, and add their contribution — at the **new** $79 `Growth` price — to the `Growth` tier's day-90 totals. Report the expansion revenue as its own line (their new $79 minus what they'd have paid staying on `Starter` at $29), separate from the price-change effect.

Produce one final table: `Growth` tier MRR and customer count at day 90, under **four** scenarios —

| Scenario | What's included |
|---|---|
| (1) No change, no expansion | pure counterfactual baseline |
| (2) No change, with expansion | isolates expansion's effect on its own |
| (3) Price change, no discount, with expansion | Lecture 3's base forecast + expansion |
| (4) Price change, with discount, with expansion | the full model — everything together |

Compare scenario (4) against scenario (2) — **not** scenario (1) — for your headline "net impact of the price change" number. Scenario (1) mixes the price change's effect with expansion's effect, which Lecture 3 §5 told you not to do.

## Sanity checks

Your Part C numbers should land close to these (small differences from rounding order in your intermediate steps are fine; a difference of more than a few dollars or a couple of customers means a logic bug, most likely in which base a percentage is taken against):

- Scenario (3) day-90: **≈ $55,142 MRR, 698 customers** (this is Lecture 3 §6's original result, unchanged).
- Discount mitigation alone (before adding expansion): reduces churn from 32 to **≈ 24**, adds **≈ $506/mo** versus scenario (3).
- Expansion alone, 90 days: **≈ 104 `Starter` accounts** upgrade, contributing **≈ $8,216/mo** to `Growth`'s total (of which **≈ $5,200/mo** is the incremental expansion value over what they'd have paid on `Starter`).
- Scenario (4) day-90: **≈ $63,800–$64,000 MRR**, **≈ 805–815 customers**.
- Headline net impact (scenario 4 vs. scenario 2, isolating the price change + discount program from expansion): **≈ +$4,200–$4,300/mo (≈ +7%)**, customers **≈ −50 to −55 (≈ −6%)** — a *better* trade than the undiscounted forecast from Challenge 2, because the discount program saved real customers.

Then, in `report.md`, answer these directly:

1. **Ship, hold, or redesign the `Growth` increase?** State your recommendation in the first sentence of the memo, then back it with your scenario (4) vs. scenario (2) comparison.
2. **Is the discount program worth running?** Quantify what it costs (customers who take a discount instead of paying full price) against what it saves (customers who'd otherwise churn entirely). Use your own numbers, not the sanity-check ranges.
3. **Should the `Starter`→`Growth` expansion be credited to the pricing team or the product team?** There's no single right answer — argue it using the distinction Lecture 3 §5 draws between price-change effects and expansion effects.
4. **What's the single riskiest assumption in this whole model?** Name one number (an elasticity, the 25% discount take-rate, the 3% expansion rate) and say what would have to be true for it to be badly wrong — and what you'd measure first, if given one week, to de-risk it before shipping.

---

## Milestones

Pace yourself; don't try to build the whole model in one sitting.

- **Milestone 1 (35 min):** `model.sql` — Part A (WTP curves + segment medians) and Part B (tier assignment + MRR rollup). Confirm every number against your Exercise 1/2 answers before moving on.
- **Milestone 2 (55 min):** `model.py` — the Part C forecast: base churn/new-signup model, discount-mitigation adjustment, expansion-revenue calculation, and the four-scenario comparison table.
- **Milestone 3 (50 min):** `report.md` — assemble the tables and write the four required answers. This is where the judgment happens; don't rush it.
- **Milestone 4 (20–30 min):** `notes.md` reflection, plus a final pass checking every number in `report.md` traces back to `model.sql`/`model.py`.

---

## Rules

- **Every dollar figure states its method.** "MRR: $63,864" with no note of which scenario, and whether discount/expansion are included, is treated as unsupported.
- **Scenario (4) vs. scenario (2), not scenario (1), for the headline number.** Mixing price-change effects with expansion effects in one number is marked wrong, on principle, even if the arithmetic underneath is correct — Lecture 3 §5 exists specifically to prevent this mistake.
- **State every assumption rate explicitly** (the 25% discount take-rate, the 3% expansion rate, your elasticity choices) in `report.md`, not buried only in code comments.
- **All four Part C scenarios must be present** in your final table, even though only two of them (2 and 4) drive the headline recommendation — Finance will ask about the other two.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 35% | Every number in `report.md` matches what `model.sql`/`model.py` actually produce, and lands within the sanity-check ranges |
| Method transparency | 20% | Every figure states its scenario/method; price-change effects and expansion effects are never conflated |
| Recommendation quality | 25% | The ship/hold/redesign call follows directly from the model, uses the correct scenario comparison, and answers all four required questions |
| Code quality | 10% | `model.sql` and `model.py` are readable, commented, and reproducible from the seed alone |
| Reflection | 10% | `notes.md` shows genuine engagement with where the model is weakest |

---

## Reflection (`notes.md`, ~200 words)

1. Which number in your model are you least confident in — the discount take-rate, the expansion rate, or one of the elasticities — and why?
2. If ScopeIQ ran the actual price increase and gave you 90 days of real data back, which specific number in this model would you expect to be furthest from what you predicted?
3. Which felt more natural to build in — SQL or pandas — for the four-scenario comparison? Where did you reach for the other one instead?
4. One question this model can't answer with a 30-respondent survey and a 5-point price experiment — what would you need to answer it with more confidence? (Foreshadows Week 10, where you'll learn to validate a change like this with a real controlled experiment instead of a forecast alone.)

---

## Why this matters

"We raised the price and MRR went up" and "we raised the price, ran the numbers first, knew within a few percent what would happen, and it landed almost exactly there" are two very different companies to work for — only the second one can tell you, before the fact, whether a pricing change is worth the risk of shipping it. Keep this model; Week 10 hands you the tool that checks whether a forecast like this one was actually right, instead of just plausible.

When done: push, then take the [quiz](../quiz.md).
