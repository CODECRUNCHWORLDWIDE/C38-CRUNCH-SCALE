# Week 7 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/>. macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with `pandas` and `scikit-learn`** — required this week for Lecture 3, Exercise 3, and the mini-project's clustering pass: `pip install pandas scikit-learn`.

## Required reading (this week's core)

- **PostgreSQL — Window functions (`NTILE`, `ROW_NUMBER`):** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `NTILE` is the mechanical core of every RFM score this week — get fluent with the general window-function pattern, not just the one call.*
- **scikit-learn — k-means user guide:** <https://scikit-learn.org/stable/modules/clustering.html#k-means>
  *Why: the canonical reference for everything Lecture 3 builds — how the algorithm works, its assumptions, and its known failure modes.*
- **scikit-learn — `StandardScaler`:** <https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html>
  *Why: the exact tool Lecture 3 uses to make features comparable before clustering — read the "why scale" rationale in the linked user guide section, not just the API signature.*
- **scikit-learn — Silhouette analysis example (with plots):** <https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html>
  *Why: seeing the silhouette plots for several values of `k` side by side makes the "which `k` is actually well-separated" judgment far more concrete than the number alone.*
- **pandas — `read_sql_query`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql_query.html>
  *Why: the bridge between this week's SQL and this week's pandas — the pattern you'll reuse in every later week that mixes both tools.*

## Reference (keep in tabs)

- **PostgreSQL — Aggregate functions and `NULL` behavior:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
  *Why: the precise rule behind why `SUM` over zero matching rows returns `NULL` while `COUNT` returns `0` — the exact trap Lecture 1's `COALESCE` fixes.*
- **PostgreSQL — `CASE` expressions:** <https://www.postgresql.org/docs/current/functions-conditional.html>
  *Why: every segment-tier classification this week (`rfm_segment`, `behavior_segment`) is a `CASE` expression — know the "first match wins" evaluation order cold.*
- **SQLite — Date and time functions (`julianday`):** <https://www.sqlite.org/lang_datefunc.html>
  *Why: SQLite's date-arithmetic reference for every recency calculation this week, if you're running the SQLite fallback instead of PostgreSQL.*
- **scikit-learn — Clustering performance evaluation:** <https://scikit-learn.org/stable/modules/clustering.html#clustering-performance-evaluation>
  *Why: silhouette score is one of several cluster-quality metrics — this page surveys the alternatives (Davies-Bouldin, Calinski-Harabasz) if you want to go deeper than this week requires.*

## Practice beyond the seed data

- **Kaggle — "Online Retail" dataset** (free, requires a free Kaggle account): search "Kaggle Online Retail dataset UCI"
  *Why: a real, much larger transactional dataset — a natural next step for practicing RFM at a scale where quintile boundaries stop feeling arbitrary and start feeling statistically meaningful.*
- **scikit-learn — "Comparing different clustering algorithms" gallery:** <https://scikit-learn.org/stable/auto_examples/cluster/plot_cluster_comparison.html>
  *Why: k-means is the right first tool, not the only tool — seeing it fail on non-spherical clusters next to DBSCAN and hierarchical clustering builds honest intuition for when to reach for something else.*

## Deeper background (optional this week)

- **Peppers & Rogers, "Managing Customer Relationships"** — the book-length treatment of RFM's origins in direct-mail marketing and its evolution into modern CRM (library or purchase).
  *Why: useful historical context for why RFM's three axes were chosen, and what it was originally built to optimize (catalog mailing costs) versus how it's used today.*
- **"An Introduction to Statistical Learning," Chapter 12 (Unsupervised Learning)** — free PDF: <https://www.statlearning.com/>
  *Why: the clearest rigorous treatment of k-means, hierarchical clustering, and the honest limits of unsupervised methods — a level deeper than this week's applied treatment.*

## Glossary

| Term | Definition |
|------|------------|
| **RFM** | Recency, Frequency, Monetary — a three-axis customer-value scoring method built from purchase history. |
| **Recency** | How long ago a customer's most recent purchase (or action) occurred; lower is generally better. |
| **Frequency** | How many times a customer has purchased (or acted); higher is generally better. |
| **Monetary** | Total amount a customer has spent; scored last in RFM priority because a single large purchase can mislead. |
| **`NTILE(n)`** | A window function that splits ordered rows into `n` as-equal-as-possible buckets, numbered 1 to `n`. |
| **Sentinel value** | An intentionally out-of-range placeholder (e.g., `9999` days) used to make a missing or edge-case value sort correctly without special-casing every downstream query. |
| **Behavioral segment** | A customer group defined by product-usage patterns (feature adoption, activity recency) rather than purchase history. |
| **Power action / power event** | A deep-adoption product action (automation, integration, API use) that predicts churn resistance more strongly than raw activity volume. |
| **k-means** | An unsupervised clustering algorithm that partitions data into `k` groups by minimizing within-cluster distance to each group's center. |
| **Feature scaling / standardization** | Rescaling numeric columns to a comparable range (typically mean 0, standard deviation 1) so no column dominates a distance-based algorithm purely because of its raw units. |
| **Inertia** | The sum of squared distances from each point to its assigned cluster center; always decreases as `k` increases. |
| **Elbow method** | Picking `k` by finding where increasing `k` further stops meaningfully reducing inertia. |
| **Silhouette score** | A per-point measure (averaged across all points) of how much closer a point is to its own cluster than to the nearest other cluster; ranges from -1 to +1. |
| **Persona** | A named, human-interpretable description of a segment or cluster, grounded in its actual feature values and real member rows — not the raw cluster index or tier label alone. |

---

*Broken link? Open an issue or PR.*
