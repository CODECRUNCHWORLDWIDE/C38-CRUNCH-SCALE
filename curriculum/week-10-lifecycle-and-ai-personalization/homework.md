# Week 10 — Homework

Five problems, ~4.5 hours total, spread across the week. These mirror the churn-modeling workflow onto a second, opposite problem (propensity to *expand*, not churn), stress-test the model's window and subgroup assumptions, and put a dollar figure on when a model-driven intervention is actually worth running.

All work continues against `crunch_flow_scale.db` and the feature table / scored population you built in the exercises, unless a problem says to add something new.

---

## Problem 1 — Mirror the workflow: propensity to expand (90 min)

Lecture 2, Section 5 flagged that the `expansion_upsell_nudge` cell used a placeholder rule ("low risk + high value") instead of an actual model. Fix that.

1. **Build the candidate population.** From your 236-row feature table, keep only customers who did **not** churn in the label window and are **not** already on the `Scale` plan (there's nowhere to expand to). *(Expected: **121** candidates.)*
2. **Build a label.** In a real company, this would come from actual plan-upgrade events. This week's seed doesn't log them, so build a documented **proxy label** instead — this is itself the point of the exercise: recognizing when you're forced to substitute a proxy for a real outcome, and saying so in writing. Use this seeded generator (same discipline as every other label this week: fixed seed, reproducible):

   ```python
   rng = np.random.default_rng(43)
   plan_boost = cand["plan"].map({"Starter": -0.2, "Growth": 0.5})
   logit = (-1.7 + 0.06*cand["events_90d"] + 0.35*cand["engagement_trend"]
            - 0.25*cand["support_tickets_90d"] + 0.0015*cand["tenure_days"]
            + plan_boost + rng.normal(0, 0.6, size=len(cand)))
   cand["expanded_in_window"] = (rng.random(len(cand)) < 1/(1+np.exp(-logit))).astype(int)
   ```

   *(Expected base rate: **47.9%**, 58 of 121.)*
3. **Train and evaluate**, same recipe as Exercise 2 (logistic regression, `test_size=0.30, random_state=42, stratify=y`, `class_weight="balanced"`). *(Expected: train **84** rows (40 positive), test **37** rows (18 positive), ROC-AUC **≈0.72**, PR-AUC **≈0.72**.)*
4. **Compare the coefficient signs to intuition.** `engagement_trend` should be the strongest positive driver, `events_31_60d` and `support_tickets_90d` should push down. Do your results match? If a sign surprises you, is it a real finding or an artifact of the proxy label's design?

**Deliver** `expansion-model.py` and a `expansion-model.md` write-up: what real event(s) would you ask a product engineering team to start logging so this proxy label could be replaced with a real one, and roughly how long would you need to wait for that data to accumulate before retraining on it?

---

## Problem 2 — Does a shorter label window help or hurt? (45 min)

Lecture 1 chose a 4-month label window somewhat arbitrarily. Test a shorter one.

1. Rebuild the label query with `LABEL_END = '2025-10-31'` (a 2-month window) instead of `2025-12-31`. *(Expected: same 236-row population, but only **46** positives — a **19.5%** base rate, down from 36.9%.)*
2. Retrain the exact Exercise 2 logistic regression recipe against this new label. *(Expected: train 165 (32 positive), test 71 (14 positive), ROC-AUC **≈0.575**, PR-AUC **≈0.264**.)*
3. **Compare.** The 4-month model scored 0.669 ROC-AUC; the 2-month model scores notably lower. Propose two different explanations for why a *shorter* window could produce a *harder* prediction problem here (hint: think about both the absolute number of positive examples the model has to learn from, and how much "warning time" 8 months of pre-cutoff features gives you for an event landing in month 9 vs. month 11).

**Deliver** `window-comparison.py` and 2–3 sentences addressing the comparison question in a `window-comparison.md`.

---

## Problem 3 — Is the intervention worth its cost? (45 min)

A model score is free; acting on it isn't. Size the interventions from Lecture 2's decision table in dollars.

1. From your scored population, compute `count`, `mean(mrr_usd)`, and `sum(mrr_usd)` grouped by `action`. *(Expected `csm_call`: **12** customers, mean **$99.00**/mo, sum **$1,188.00**/mo → **$14,256/year** in combined annual value.)*
2. **CSM call breakeven.** Assume a CSM call has a fully-loaded cost of **$150** (prep + call + follow-up time). Using the *mean* annual value of a `csm_call`-segment customer (`$99 × 12 = $1,188`), compute the breakeven save probability: the minimum chance the call would need to have of actually preventing a cancellation for its expected value to exceed its cost. *(Expected: **≈12.6%**.)* Is that a plausible save rate for a well-run CSM outreach, given what Challenge 2 found for a much cheaper automated email (a 45%-true, ~24%-observed relative save rate)?
3. **Win-back email breakeven.** Using the `winback_email_discount` segment's mean MRR (**$28.08**/mo → **$337/year**) and assuming the email + discount costs **$8** per customer to run (platform cost + average discount redeemed), compute its breakeven save probability. Compare it to your Task 2 answer — does the math explain why this action is automated and the other is a human call, beyond just "one is more expensive to run per customer"?
4. **A segment you'd cut.** Using this same breakeven logic, identify one action in the six-cell table you'd argue against running at all, or would run at a smaller scale than Lecture 2 proposed, and justify it in one paragraph with numbers.

**Deliver** `breakeven.py` (or a short calculation script) and `breakeven.md` with your Task 4 argument.

---

## Problem 4 — Audit the model for a blind spot (45 min)

Aggregate metrics can hide a subgroup the model handles badly. Find one.

1. Filter your scored population to `plan == 'Scale'`. *(Expected: **33** customers.)* Compute their actual churn base rate (`churned_in_window.mean()`). *(Expected: **15.2%**, 5 of 33.)*
2. Compute their `risk_band` distribution. *(Expected: **zero** Scale customers land in the `"high"` band — all 33 are `"low"` or `"medium"`.)*
3. **The finding.** Cross Tasks 1 and 2: of the 5 Scale customers who actually churn in the label window, how many were ever flagged `"high"` risk and routed to a `csm_call`? *(Expected: **zero**.)* Compute "recall within the Scale subgroup at the high-risk threshold" and compare it to the overall test-set recall from Exercise 2 (38.5%).
4. **Why this happened.** In 3–4 sentences, explain the likely mechanism: Scale plan carries a strong negative (protective) coefficient in the model (Lecture 1, Section 7's `plan_starter`/`plan_growth` coefficients implicitly make `Scale` the safest reference level) — and the model saw relatively few Scale examples during training. Is this a bug in your code, or a genuine limitation of training on 236 rows with only 33 in the affected subgroup? What would you do about it before trusting this model to route your highest-value accounts?

**Deliver** `subgroup-audit.md` with all four numbered findings and your Task 4 explanation. This finding belongs in the mini-project's `model_card.md` limitations section if you haven't already put something like it there.

---

## Problem 5 — Extend the guardrails (45 min)

Lecture 2, Section 5 named four guardrail concerns but only sketched them informally. Formalize three.

Write full, specific rules (not general principles) for:

1. **Frequency capping** — the exact rule (in plain English, then as a `WHERE`/`HAVING` SQL condition against a hypothetical `lifecycle_sends` log table) that prevents any customer from receiving more than one lifecycle touch in a 14-day window, across *all* action types, not just within one.
2. **Discount leakage prevention** — a rule that would have caught it if the `winback_email_discount` logic accidentally started firing for `csm_call`-segment customers too (Lecture 2, Section 5's specific worry). Write it as a SQL assertion (a query that should return zero rows on healthy routing) in the style of this course's data tests from Week 6.
3. **A metric to monitor the NBA system itself** — not a customer-level metric, but a system-level one a RevOps lead would put on a dashboard to catch this whole pipeline silently breaking (e.g., "action distribution drifts more than X% week over week" or "% of customers with `action IS NULL`"). Define it with the same rigor as Week 6's `metrics.md` entries: definition, computed-in table, source of truth, owner.

**Deliver**: append all three to a `guardrails.md`, in your own words, each with the concrete rule/query — not a restatement of the lecture's prose.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 90 min |
| 2 | 45 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 45 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md) if you haven't already.
