# Week 9 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with `pandas` and `numpy`:** `pip install pandas numpy`. Verify with `python3 -c "import pandas, numpy; print(pandas.__version__, numpy.__version__)"`.

## Required reading (this week's core)

- **Van Westendorp Price Sensitivity Meter — plain-language overview.** Search "Van Westendorp Price Sensitivity Meter" for any of the several accessible market-research primers covering the four-question method and the PMC/PME/OPP/IDP intersections (this technique predates the modern web and has no single canonical URL — cross-reference two or three summaries and they should agree on the mechanics covered in Lecture 1).
  *Why: Lecture 1 builds the method from first principles, but seeing it explained by a market-research practitioner in different words cements it.*
- **PostgreSQL — Aggregate functions, incl. `FILTER` and ordered-set aggregates:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
  *Why: `FILTER (WHERE ...)`, `PERCENTILE_CONT`, and the regression aggregates (`REGR_SLOPE`, `REGR_INTERCEPT`, `REGR_R2`) all live here — this week's whole SQL toolkit in one page.*
- **SQLite — Recursive Common Table Expressions:** <https://www.sqlite.org/lang_with.html>
  *Why: SQLite has no `generate_series`; the price-grid trick from Lecture 1 needs a recursive CTE instead.*
- **Investopedia — "Price Elasticity of Demand":** <https://www.investopedia.com/terms/p/priceelasticity.asp>
  *Why: a clean, worked-example treatment of arc vs. point elasticity, with the exact formulas Lecture 3 uses.*

## Reference (keep in tabs)

- **NumPy — `polyfit` and `linalg.lstsq` (least-squares fitting):** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
  *Why: the SQLite-side (and cross-check) path for Lecture 3's demand-curve fit.*
- **pandas — `.groupby()`, `.median()`, `.apply()`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.groupby.html>
  *Why: every segment-level and tier-level rollup this week goes through one of these three.*
- **PostgreSQL — `generate_series`:** <https://www.postgresql.org/docs/current/functions-srf.html>
  *Why: the fast way to build a price grid on Postgres for the PSM curves.*
- **PostgreSQL — `CASE` expressions:** <https://www.postgresql.org/docs/current/functions-conditional.html>
  *Why: every tier-assignment query this week is a `CASE` expression.*

## Practitioner writing on pricing (vendor content, read critically)

- **ProfitWell / Price Intelligently — pricing and packaging guides.** Search "profitwell packaging tiers value metric" and "profitwell how to raise prices without losing customers." *Why: among the more data-literate public writing on value-metric selection, grandfathering, and discount-offer design — vendor-authored, so cross-check specific numeric claims, but the frameworks are solid.*
- **Patrick Campbell / Price Intelligently — Van Westendorp critiques and extensions.** Search "van westendorp limitations saas pricing." *Why: useful counterpoint — Van Westendorp was built for physical consumer goods, not subscription software, and knowing its blind spots (it doesn't account for competitors, or for a value metric at all) makes you a sharper user of it, not a weaker one.*

## Practice beyond the seed data

- **Kaggle — search "SaaS pricing" or "customer churn" datasets.** *Why: real (if messy) data to practice the elasticity-fitting and cohort-forecasting techniques from Lecture 3 on something other than the course seed.*
- **Rebuild this week's model for a product you actually use.** Pick any SaaS tool you pay for, guess its likely value metric, and sketch what a Van Westendorp survey of its actual pricing page tiers might reveal — you won't have real respondent data, but reasoning through *which* value metric a real company chose (and why) is excellent standalone practice for Lecture 2.

## Glossary

| Term | Definition |
|------|------------|
| **Willingness to pay (WTP)** | The maximum price a customer would pay for a product — unobservable directly; estimated via survey (stated) or usage data (behavioral). |
| **Van Westendorp PSM** | A four-question survey method (too cheap / bargain / expensive / too expensive) that estimates an acceptable price range from cumulative response curves. |
| **PMC** | Point of Marginal Cheapness — Too Cheap ∩ Getting Expensive; the lower bound of the acceptable price range. |
| **PME** | Point of Marginal Expensiveness — Bargain ∩ Too Expensive; the upper bound of the acceptable price range. |
| **OPP** | Optimal Price Point — Too Cheap ∩ Too Expensive; the price of minimum combined resistance. |
| **IDP** | Indifference Price Point — Bargain ∩ Getting Expensive; a secondary central price estimate. |
| **Conjoint analysis** | A survey technique that infers the dollar value of individual features by analyzing trade-offs across many priced product bundles. |
| **Behavioral WTP** | Willingness-to-pay signals inferred from actual usage or purchase behavior, not stated survey answers. |
| **Value metric** | The unit a price scales with (seats, usage volume, tracked users, …) — the axis packaging is built on. |
| **Good-Better-Best** | A three-tier pricing structure, each tier priced/featured for a different customer segment's value. |
| **Cannibalization** | When a tier boundary lets a high-value customer rationally choose a cheaper tier and still get most of what they need. |
| **Tier cliff** | A hard usage cap that turns a small usage increase into a disproportionate full-tier price jump. |
| **Price elasticity of demand (E)** | The percentage change in quantity demanded per percentage change in price; `|E| > 1` is elastic, `|E| < 1` is inelastic. |
| **Revenue-maximizing price (p\*)** | The price where `dR/dp = 0`, equivalently where `|E| = 1`, on a fitted demand curve. |
| **Grandfathering** | Letting existing customers keep an old price for a defined notice period before a price increase applies to them. |
| **Expansion revenue** | New MRR from existing customers (e.g., a tier upgrade) independent of any price change. |

---

*Broken link? Open an issue or PR.*
