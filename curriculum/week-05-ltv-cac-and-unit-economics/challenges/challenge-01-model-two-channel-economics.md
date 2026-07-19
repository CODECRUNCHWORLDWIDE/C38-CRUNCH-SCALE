# Challenge 1 — Model Unit Economics for Two Channels

**Time:** ~60 minutes. **Difficulty:** Medium. **No single right answer on the recommendation.**

## The scenario

Lumen Metrics' head of growth has asked you for one artifact: a table that puts `paid_search` and `organic_content` side by side on every unit-economics metric that matters, plus a one-paragraph read on what it would take for each channel to reach "healthy" (LTV:CAC ≥ 3:1). This is the exact deliverable a growth or RevOps analyst produces every quarter — not a five-page deck, one honest table and a clear paragraph.

## Your task

Using SQL and/or pandas against this week's seed data, build a single table with one row per channel and these columns:

| Column | What it is |
|---|---|
| `channel` | `paid_search` or `organic_content` |
| `mrr` | monthly recurring revenue per customer |
| `margin_per_month` | contribution margin per customer per month (80% gross margin) |
| `avg_monthly_churn` | from the retention curve, ages 1–6 |
| `ltv_projected` | churn-reciprocal LTV |
| `cac_blended` | fully-loaded, full-year CAC |
| `cac_trailing` | fully-loaded, Q4-only CAC |
| `ltv_cac_blended` | `ltv_projected ÷ cac_blended` |
| `ltv_cac_trailing` | `ltv_projected ÷ cac_trailing` |
| `naive_payback_months` | using `cac_blended` |

Then write up, in `challenge-01.md`:

1. **The table** (as a markdown table or a pasted `DataFrame`), with the query/code that produced every column.
2. **One paragraph per channel**: is it currently healthy, marginal, or unprofitable, on which CAC basis, and what's the single biggest lever that would move it toward "healthy"? (Options to consider for each channel: lower CAC, raise retention/lower churn, raise MRR/ARPU, or some combination. Be specific — "improve retention" is not an answer; "cut month-1 churn from ~9% to ~5%" is.)
3. **One sentence stating which method you used for LTV** (cohort-sum floor vs. reciprocal projection) **and why** you chose it for this deliverable.

## Constraints

- Use the actual seed data — no invented numbers. If you extend the model (e.g., "what if churn dropped to X%"), clearly label that row as a scenario, separate from the observed table.
- State which CAC (blended or trailing) you'd hand to a decision-maker asking "what should we do *next quarter*," and defend the choice in one sentence.
- Round dollars to 2 decimals, rates/ratios to 2–3 decimals, consistently.

## Hints

<details>
<summary>On the "biggest lever" question</summary>

Run the numbers both ways before you answer. For `paid_search`: does cutting CAC 25% get it to 3:1 faster than cutting churn in half? For `organic_content`: it's already trending toward 3:1 on trailing CAC alone — does it need a lever at all, or does it just need more time and continued investment at the current cadence? The answer isn't obvious until you actually compute both scenarios.

</details>

<details>
<summary>On picking blended vs. trailing CAC for the recommendation</summary>

Lecture 2 §2 already made the case: blended is a historical report card, trailing is closer to what the *next* dollar costs. If your recommendation is about where to put *next quarter's* budget, which one answers that question?

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Reproducibility | Numbers stated with no query/code shown | Every column traceable to a query or script |
| Method transparency | Uses "LTV" and "CAC" without saying which version | States cohort-sum vs. reciprocal, blended vs. trailing, explicitly |
| Lever specificity | "Improve retention" / "lower CAC" | A specific number, tested against the model (e.g., "churn from 8.9% to 6% moves the ratio from 0.67 to X") |
| Consistency | Recommendation doesn't match the table | Verdict follows directly from the numbers shown |

## Submission

Commit `challenge-01.md` (plus any `.sql`/`.py` files it references) to your portfolio under `c38-week-05/challenge-01/`.
