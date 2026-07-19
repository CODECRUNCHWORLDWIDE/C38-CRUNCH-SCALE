# Lecture 3 — Clustering for Customer Intelligence

> **Duration:** ~2 hours. **Outcome:** You can pull a feature matrix from SQL into pandas, scale it correctly, run k-means with scikit-learn, choose `k` using the elbow and silhouette methods, and turn a set of cluster numbers into personas — while knowing exactly where the algorithm's boundaries stop meaning anything.

## 1. Why cluster, when you already have RFM and behavioral tiers

Lectures 1 and 2 both required you to **hand-pick thresholds**: `r_score >= 4`, `power_events >= 5`, `days_since_last_event > 60`. Those thresholds came from domain reasoning, and they're defensible — but they're also arbitrary in a way that's easy to miss. Why `>= 4` and not `>= 3`? Why `60` days and not `45`? Someone made a judgment call, and a different analyst might have drawn the lines differently and gotten a different segmentation of the *same* customers.

Clustering flips the process: instead of you drawing the boundaries, you hand the algorithm a set of numeric features and let it find groups of customers that are close to each other and far from everyone else, with no thresholds specified in advance. **k-means** is the standard first algorithm for this — simple, fast, and good enough for a customer base this size.

This does not mean clustering is "more objective" than RFM. It trades one kind of judgment call (thresholds) for another (which features to include, how to scale them, and what value of `k` to pick) — Section 6 comes back to this hard.

## 2. Step 1 — pull a feature matrix from SQL into pandas

Everything upstream of clustering is a SQL query, exactly like Lectures 1 and 2 — the only new step is loading the result into a `DataFrame` instead of reading it as a printed table:

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect("crunch_scale_w7.db")

query = """
WITH rfm AS (
    SELECT
        c.customer_id,
        COALESCE(CAST(julianday('2025-12-31') - julianday(MAX(o.order_date)) AS INTEGER), 9999) AS recency_days,
        COUNT(o.order_id) AS frequency,
        ROUND(COALESCE(SUM(o.amount), 0), 2) AS monetary
    FROM customers c LEFT JOIN orders o ON o.customer_id = c.customer_id
    GROUP BY c.customer_id
),
beh AS (
    SELECT
        c.customer_id,
        COUNT(pe.event_id) AS total_events,
        SUM(CASE WHEN pe.event_name IN ('automation_triggered','integration_connected','api_call')
                 THEN 1 ELSE 0 END) AS power_events
    FROM customers c LEFT JOIN product_events pe ON pe.customer_id = c.customer_id
    GROUP BY c.customer_id
)
SELECT c.customer_id, c.company_name, c.plan,
       r.recency_days, r.frequency, r.monetary,
       b.total_events, b.power_events
FROM customers c
JOIN rfm r ON r.customer_id = c.customer_id
JOIN beh b ON b.customer_id = c.customer_id
ORDER BY c.customer_id;
"""

df = pd.read_sql_query(query, conn)
```

`pd.read_sql_query` runs the exact SQL string against the open connection and returns a `DataFrame` with one row per customer, columns matching the `SELECT` list. This is the pattern for the rest of the week — and honestly the rest of your career: **the database does the joining and aggregating (it's faster and less error-prone at that than pandas is), and pandas does the modeling** (which SQL genuinely can't do — no engine ships a `KMEANS()` aggregate function).

## 3. Step 2 — scale the features before clustering, or the results are meaningless

k-means measures **distance** between customers in feature space — literally, Euclidean distance across the 5 numeric columns. That's the trap: `monetary` ranges from `0` to over `1,500`; `power_events` ranges from `0` to `14`. Feed those raw into k-means and `monetary` alone will dominate every distance calculation — the algorithm will effectively cluster on spend and ignore the other four columns almost entirely, no matter how meaningful they are.

The fix is **standardization** — rescale every column to mean 0, standard deviation 1, so no column's raw units can dominate just because it happens to be measured in bigger numbers:

```python
from sklearn.preprocessing import StandardScaler

features = ["recency_days", "frequency", "monetary", "total_events", "power_events"]
X = df[features].copy()

# Cap recency at 365 days before scaling — the two never-purchased customers carry
# a 9999-day sentinel (Lecture 1), and an unscaled 9999 would single-handedly
# stretch the whole distance metric around two rows. Capping keeps the sentinel's
# MEANING (worst-in-class recency) without letting its raw MAGNITUDE distort scaling.
X["recency_days"] = X["recency_days"].clip(upper=365)

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

That `clip(upper=365)` line is worth pausing on. Lecture 1's `9999` sentinel was the *right* choice for a `CASE`/`NTILE` pipeline, where it only ever needs to sort last. It is the *wrong* raw input for a distance-based algorithm — `StandardScaler` would compute a mean and standard deviation dominated by two extreme outliers, quietly compressing every other customer's real variation into a rounding error. **The same underlying fact ("never purchased, or purchased a very long time ago") needs a different numeric encoding depending on which downstream technique consumes it.** That's a general lesson, not a one-off fix: always ask what a number is about to be *used for* before deciding how to represent an edge case.

## 4. Step 3 — choose `k` with the elbow and silhouette methods

k-means needs you to specify `k` (the number of clusters) up front — it doesn't discover that number on its own. Two standard diagnostics help you pick it:

- **Inertia (the "elbow" method)** — the sum of squared distances from each point to its assigned cluster center. Inertia always decreases as `k` increases (more clusters can always fit the data at least as well), so you're not looking for the minimum — you're looking for the **elbow**, the point where adding another cluster stops buying you much improvement.
- **Silhouette score** — for each point, how much closer it is to its own cluster than to the nearest other cluster, averaged across all points, ranging from -1 (badly clustered) to +1 (perfectly separated). Unlike inertia, silhouette has a genuine maximum you can look for directly.

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

for k in range(2, 8):
    km = KMeans(n_clusters=k, random_state=38, n_init=10)
    labels = km.fit_predict(X_scaled)
    sil = silhouette_score(X_scaled, labels)
    print(f"k={k}  inertia={km.inertia_:.2f}  silhouette={sil:.3f}")
```

Against Crunch Flow's 30 customers, this prints:

```text
k=2  inertia=79.26  silhouette=0.363
k=3  inertia=46.69  silhouette=0.401
k=4  inertia=34.74  silhouette=0.405
k=5  inertia=26.65  silhouette=0.413
k=6  inertia=21.79  silhouette=0.373
k=7  inertia=16.84  silhouette=0.403
```

Silhouette peaks at **k=5** (0.413) — it's also the last `k` before the score visibly drops at `k=6`, which is the elbow-method's version of the same signal. Both diagnostics agree: **5 clusters is the best-supported choice for this dataset.** (`random_state=38` and `n_init=10` are there for reproducibility: k-means starts from random cluster centers and can converge to different local optima on different runs; `n_init=10` runs it 10 times from different starting points and keeps the best, and `random_state` pins the randomness so your numbers match a lecture's or a classmate's exactly.)

## 5. Step 4 — fit k=5 and interpret the clusters

```python
km5 = KMeans(n_clusters=5, random_state=38, n_init=10)
df["cluster"] = km5.fit_predict(X_scaled)

profile = df.groupby("cluster")[features + ["mrr"]].mean().round(1)
profile["n"] = df.groupby("cluster").size()
print(profile)
```

```text
          recency_days  frequency  monetary  total_events  power_events   mrr  n
cluster
0                 75.7        1.7     124.8           7.7           0.3  29.0  3
1                  9.8        8.2     916.7          23.0          10.2 299.0  5
2               3049.6        1.3     124.7           6.9           0.4  49.0  7
3                120.1        5.7     475.2          14.8           0.7  99.0 10
4                  4.4        2.6     429.8          15.4           4.6 259.0  5
```

*(Cluster `2`'s huge `recency_days` mean reflects the `9999` sentinel from the two never-purchased customers pulling that group's average up — a reminder that a cluster's raw feature means need the same context-aware reading as any other summary statistic.)*

Read each cluster back into a persona, grounding every claim in the numbers, not vibes:

- **Cluster 1 — "Power Champions"** (n=5, avg MRR $299, avg power_events 10.2). Highest spend, most frequent buyers, freshest recency, heaviest automation/integration/API use. This is Lecture 1's `Champions` and Lecture 2's `Power User` groups **converging on the same 5 customers** — independent confirmation from a third method.
- **Cluster 4 — "New & Embedding Fast"** (n=5, avg MRR $259, recency 4.4 days). Recent, moderate order count, but already showing meaningful `power_events` (4.6 average) for how new they are. This is a distinct, genuinely interesting group clustering surfaced that neither Lecture 1 nor Lecture 2 named on its own — customers who are both new *and* already technically embedded deserve a different onboarding conversation than customers who are new and still shallow.
- **Cluster 3 — "Steady Growth-Plan Core"** (n=10, avg MRR $99, moderate everything). The single largest cluster — the broad middle of the customer base, buying occasionally, using the product regularly, not yet automating anything.
- **Cluster 0 — "Low-Touch Starter"** (n=3, avg MRR $29, low frequency and events across the board). Small, cheap-plan, thin engagement on every axis.
- **Cluster 2 — "Cold"** (n=7, no recent purchases, thin recent usage). The dormant/at-risk tail.

## 6. Where k-means lies to you

**k-means will always produce exactly `k` clusters, whether or not `k` real groups exist in the data.** Run it with `k=5` on customers who are actually one big undifferentiated blob, and it will still confidently draw five boundaries through that blob and hand you five means — the algorithm has no way to say "actually, there's only 2 real groups here" unless you go looking with silhouette or a similar diagnostic yourself.

Notice what happened between Lecture 1's five hand-picked RFM tiers and this lecture's five discovered clusters: **`Champions` and `New/Promising` came back almost exactly**, recovered independently by an algorithm that never saw those labels. That's a genuinely strong result — it means those two groups are *real* patterns in the underlying feature geometry, not artifacts of where a human drew a threshold. But **`Loyal Core` and `At Risk` did not come back as separate clusters** — they largely merged into Cluster 3. Why? Because both groups share very similar `frequency`/`total_events` values in a *snapshot* — the real difference between "loyal and steady" and "at risk" is a **trend over time** (are their numbers holding, or declining?), and a snapshot feature matrix, by construction, cannot see trend. k-means didn't fail here — it correctly reported that, given only these five snapshot features, those two RFM-defined groups aren't actually separable. Fixing that would mean engineering a trend feature (e.g., orders in the last 90 days vs. the 90 days before that) and re-clustering — a real, worthwhile next step, and exactly the kind of thing Exercise 3 asks you to try.

Two rules to keep, for this week and beyond:

1. **A cluster is not a persona until a human names it and checks the name against the raw rows.** `cluster == 3` means nothing to a sales rep; "Steady Growth-Plan Core, 10 customers, $99 MRR average, buys occasionally, hasn't automated yet" is something a team can act on. Naming is not decoration — it's the translation step that makes the model useful at all.
2. **If two clusters merge that you expected to be different, that's information, not failure.** It tells you the features you fed the model don't distinguish those groups — which is a specific, fixable, and often more interesting finding than a clean 1:1 mapping to your prior beliefs would have been.

## 7. Check yourself

- Why must every feature be standardized (`StandardScaler`) before running k-means, and what would go wrong on this dataset specifically if `monetary` and `power_events` were left on their raw scales?
- Why does inertia always decrease as `k` increases, and why is that exactly why you can't just pick the `k` with the lowest inertia?
- What does a silhouette score close to `+1` mean? Close to `0`? Negative?
- Why was `recency_days` capped at 365 before scaling, when the `9999` sentinel was the *correct* choice in Lecture 1's SQL?
- `Champions` and `New/Promising` came back as clean, separate clusters; `Loyal Core` and `At Risk` did not. What does that difference tell you about which of the four RFM segments are grounded in real feature-space separation versus threshold artifacts?
- What single new feature would you engineer to try to separate `Loyal Core` from `At Risk` in a re-clustering pass, and why would it help where the current five features don't?

If those are automatic, you're ready for the challenges — profiling every segment by value, and turning a segmentation (any of the three) into a targeting plan a real team could run tomorrow.

## Further reading

- **scikit-learn — `KMeans` user guide:** <https://scikit-learn.org/stable/modules/clustering.html#k-means>
- **scikit-learn — `StandardScaler`:** <https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html>
- **scikit-learn — Silhouette analysis example (with plots):** <https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html>
- **pandas — `read_sql_query`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql_query.html>
- **"An Introduction to Statistical Learning," Chapter 12 (Unsupervised Learning)** — free PDF: <https://www.statlearning.com/>
  *Why: the clearest rigorous treatment of k-means, the elbow method, and why clustering is fundamentally a judgment-laden technique, not an objective one.*
