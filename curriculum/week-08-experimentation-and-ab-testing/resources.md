# Week 8 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/>. macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with pandas and numpy**: `pip install pandas numpy`. This week's sizing, significance, and confidence-interval math all runs in plain Python — no separate stats package is required, but if you want one later for cross-checking, `statsmodels` (`pip install statsmodels`, see `statsmodels.stats.proportion`) implements the same tests this week derived from scratch.

## Required reading (this week's core)

- **Evan Miller — "How Not To Run an A/B Test"**: <https://www.evanmiller.org/how-not-to-run-an-ab-test.html>
  *Why: the single most-cited explanation of the peeking problem, written for practitioners, not statisticians — read this before you ever watch a live test's dashboard again.*
- **Evan Miller — A/B testing sample size calculator**: <https://www.evanmiller.org/ab-testing/sample-size.html>
  *Why: a great tool for sanity-checking the `required_n_per_arm` function you wrote by hand in Exercise 1 — your numbers should land close to this calculator's.*
- **Kohavi, Tang, Xu — *Trustworthy Online Controlled Experiments* (Cambridge University Press, 2020), Ch. 1–4, 19–20** — the standard textbook reference for this entire week, from Microsoft/Bing/LinkedIn/Airbnb experimentation-platform veterans.
  *Why: Lecture 1's brief structure, Lecture 2's power math, and Lecture 3's five traps all trace back to this book's treatment of the same material at production scale.*
- **Fabijan et al. — "Diagnosing Sample Ratio Mismatch in Online Controlled Experiments"** (Microsoft/Booking.com research, freely available): search "Fabijan Diagnosing Sample Ratio Mismatch" for the PDF.
  *Why: real-world case studies of exactly the kind of SRM bug Exercise 3 and Challenge 2 ask you to catch — written by the teams who built SRM detection into production experimentation platforms.*
- **PostgreSQL — Window functions** (used for the cumulative peeking-trajectory query): <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: Homework Problem 4's single combined query, and Lecture 3's peeking-trajectory example, both lean on `SUM(...) OVER (PARTITION BY ... ORDER BY ...)`.*

## Reference (keep in tabs)

- **Python `math` module** (for `erf`, used to convert a z-score to a p-value without a stats library): <https://docs.python.org/3/library/math.html#math.erf>
  *Why: every significance calculation this week builds the p-value from `erf` directly, so you can see exactly what's happening instead of calling a black-box `.pvalue` attribute.*
- **PostgreSQL — Aggregate expressions (`FILTER`)**: <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES>
  *Why: the clean way to compute conversion counts and rates in one query, used throughout this week's SQL.*
- **`statsmodels.stats.proportion` documentation** (for cross-checking, once you trust your hand-rolled formulas): <https://www.statsmodels.org/stable/stats.html#proportion>
  *Why: `proportions_ztest` and `proportion_confint` implement exactly what Lecture 2 derived by hand — a good sanity check, not a replacement for understanding the derivation.*
- **Wikipedia — Simpson's Paradox**: <https://en.wikipedia.org/wiki/Simpson%27s_paradox>
  *Why: more worked examples beyond the NimbleCart case in Lecture 3, including the classic UC Berkeley admissions data — public-domain statistics history worth knowing regardless of this course.*

## Practice beyond the seed data

- **Optimizely — "Stats Engine" blog series** (free, no signup for the blog): search "Optimizely Stats Engine sequential testing"
  *Why: a production experimentation vendor's public explanation of *why* fixed-horizon testing struggles with peeking, and one real approach (always-valid p-values) to letting teams look whenever they want without inflating the false-positive rate.*
- **Airbnb Engineering blog — experimentation platform posts** (free, search "Airbnb experimentation platform engineering blog")
  *Why: real production writeups of building SRM detection, guardrail metrics, and novelty-effect monitoring at scale — the same five traps from Lecture 3, described from the infrastructure side.*

## Deeper background (optional this week)

- **Ron Kohavi, Diane Tang, Ya Xu — "Online Controlled Experiments and A/B Testing" (Encyclopedia of Machine Learning and Data Mining entry, freely available as a preprint PDF)** — search the title.
  *Why: a condensed, single-paper version of the full textbook above — good if you want the theory without committing to the whole book yet.*
- **Georgi Georgiev — *Statistical Methods in Online A/B Testing*** — a practitioner-written book covering Bayesian alternatives to the frequentist approach this week teaches (library or purchase).
  *Why: this course teaches the frequentist z-test/CI approach because it's the industry default and requires no additional libraries — Georgiev's book is the natural next step if you want to see the Bayesian alternative (credible intervals, posterior probability of a real effect) that some teams prefer.*

## Glossary

| Term | Definition |
|------|------------|
| **Hypothesis** | A falsifiable if/then/because claim about what a change will do to a specific metric, and why. |
| **Primary metric** | The single, pre-committed number that decides a test's ship/no-ship outcome. |
| **Guardrail metric** | A number that must not get meaningfully worse, regardless of what the primary metric shows. |
| **Randomization unit** | The entity (visitor, session, request) that gets assigned to a variant; must match the unit of analysis. |
| **Baseline rate (p₁)** | The primary metric's current value under control, before the test. |
| **MDE** | Minimum detectable effect — the smallest lift worth being able to detect, a business decision, not a statistical one. |
| **α (alpha)** | Significance level — tolerance for a Type I error (false positive); conventionally 0.05. |
| **Power (1 − β)** | Tolerance for correctly detecting a true effect of size MDE; conventionally 80%. |
| **Type I error** | A false positive — calling a nonexistent effect significant. |
| **Type II error** | A false negative — missing a real effect because the test was underpowered. |
| **Two-proportion z-test** | The significance test comparing two observed conversion rates against the null hypothesis that they're equal. |
| **Confidence interval (CI)** | A range of plausible values for the true effect size, at a stated confidence level (typically 95%). |
| **Peeking** | Checking a test's significance repeatedly and stopping at the first favorable read, which inflates the real false-positive rate above the stated α. |
| **Sample-ratio mismatch (SRM)** | A statistically significant deviation between the observed arm split and the designed split, usually signaling a broken assignment mechanism. |
| **Novelty effect** | A treatment effect inflated early by curiosity, which fades as the change becomes routine. |
| **Primacy effect** | A treatment effect understated early because users need time to adjust to or trust a change. |
| **Multiple comparisons** | Testing several metrics or subgroups without correction, inflating the real chance of at least one spurious "significant" result. |
| **Bonferroni correction** | Dividing α by the number of comparisons to keep the overall false-positive rate at the intended level. |
| **Simpson's paradox** | An aggregate result that reverses direction compared to every subgroup, caused by an unevenly distributed confounding variable. |
| **Chi-square goodness-of-fit test** | The statistical test (used for SRM detection) comparing observed category counts to an expected distribution. |

---

*Broken link? Open an issue or PR.*
