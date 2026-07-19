# Exercise 3 — Cluster Customers with k-means

**Goal:** Build the SQL → pandas → scikit-learn pipeline from Lecture 3 yourself: pull a feature matrix, scale it, choose `k` with the elbow/silhouette methods, fit k-means, and interpret the clusters against Exercises 1 and 2.

**Estimated time:** 60 minutes.

## Setup

```bash
pip install pandas scikit-learn
```

Create `exercise-03.py` (or a notebook). Comment each section with `# Task N`.

## Tasks

1. **Pull the feature matrix.** Write the combined SQL query from Lecture 1's `rfm` CTE + Lecture 2's `beh` CTE (recency_days, frequency, monetary, total_events, power_events — 5 features), and load it into a `DataFrame` with `pd.read_sql_query`. *(Expected: `df.shape == (30, 9)` — `customer_id`, `company_name`, `plan`, and the 5 numeric features, plus whatever you named the join.)*

2. **Scale correctly.** Build `X` from the 5 numeric columns, `.clip(upper=365)` the `recency_days` column (see Lecture 3 section 3 for why), then fit a `StandardScaler` and transform. Print `X_scaled.mean(axis=0)` and `X_scaled.std(axis=0)`. *(Expected: means very close to `0`, standard deviations very close to `1`, for all 5 columns.)*

3. **Elbow + silhouette sweep.** Loop `k` from 2 to 7, fit `KMeans(n_clusters=k, random_state=38, n_init=10)` each time, and print `k`, `inertia_`, and `silhouette_score`. *(Expected: silhouette peaks at `k=5` with a score of approximately `0.41`. Your exact digits may vary slightly by scikit-learn version — the *peak location* at `k=5` should not.)*

4. **Fit `k=5` and profile.** Fit the final model at `k=5`, add a `cluster` column to `df`, then `groupby("cluster")` and report the mean of each of the 5 features plus `mrr`, and the cluster size `n`. *(Expected: one cluster of size 5 with the highest `monetary` and `power_events` averages — this is the Champions cluster.)*

5. **Name the personas.** For each of the 5 clusters, write one sentence identifying which customers are in it (by `company_name` or by cross-referencing Exercise 1/2's tier columns) and a plausible business name for the group (`"Power Champions"`, etc. — invent your own names, don't just copy Lecture 3's).

## Expected result (spot checks)

- Task 1 → `df.shape[0] == 30`.
- Task 2 → all 5 scaled columns have mean ≈ 0, std ≈ 1.
- Task 3 → silhouette score peaks at `k=5`.
- Task 4 → exactly one cluster of size 5 has the highest `power_events` mean.
- Task 5 → your 5 persona names, each grounded in at least one concrete number from Task 4's profile table.

## Done when…

- [ ] `exercise-03.py` runs top to bottom without errors and prints all 5 tasks' output.
- [ ] Your Task 3 loop uses the **same** `random_state` for every value of `k` you compare, so the comparison is fair.
- [ ] Task 4's cluster sizes sum to 30.
- [ ] Every persona name in Task 5 is backed by a number, not just an adjective — "the highest-spending cluster, avg monetary $X" is a defensible sentence; "the good customers" is not.

## Stretch

- Re-run the whole pipeline with `k=4` and separately with `k=6`. For each, name what changed compared to the `k=5` solution — did a cluster split into two, or did two clusters merge into one? Which `k` would you actually recommend shipping to a sales team, and why might "best silhouette score" and "most useful to a human" not be the same answer?
- Engineer one new feature — `recent_frequency`: the count of orders in the **last 90 days only** (`WHERE order_date >= '2025-10-03'`) — add it to the feature matrix, re-scale, and re-cluster at `k=5`. Check whether this new feature succeeds in separating `Loyal Core` from `At Risk` where the original 5 features couldn't (Lecture 3, section 6). *(Spoiler, don't peek until you've run it yourself: a single 90-day recency-of-purchase feature usually isn't enough on its own either — most `At Risk` customers have `recent_frequency = 0`, but so do several genuine `Loyal Core` customers who simply haven't needed an add-on this quarter. That's not a failed exercise — it's a real finding: separating "quiet but fine" from "quietly leaving" needs a feature that captures* trend *, like a ratio of this-90-days activity to the previous 90 days, not just a single recent-window count.)* Show the before/after cluster assignments for a few `Loyal Core` and `At Risk` customers side by side, and write two sentences on what a trend-ratio feature would need to look like.

## Submission

Commit `exercise-03.py` (or your notebook, exported to `.py` or `.md`) to your portfolio under `c38-week-07/exercise-03/`.
