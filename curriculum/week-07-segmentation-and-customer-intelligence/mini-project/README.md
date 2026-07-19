# Mini-Project — Build and Profile a Customer-Segmentation Model

> Combine RFM and behavioral signals into a single customer-segmentation model in SQL, run a k-means clustering pass in pandas over the same customers, and ship a written recommendation naming the single highest-value segment Crunch Flow should target next quarter — with every claim traceable to a query.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

Every exercise and challenge this week handed you one technique at a time. Real segmentation work has to combine them, notice where they disagree, and land on a single defensible recommendation anyway — because a VP doesn't have time for three separate decks. This mini-project is that synthesis, done once, completely, by you.

---

## Deliverable

A directory in your portfolio `c38-week-07/mini-project/` containing:

1. `segmentation.sql` — one SQL script that builds, in order: the RFM scoring pipeline, the behavioral tier pipeline, and a combined table joining both plus `customers.mrr`, one row per customer, all 30 rows.
2. `cluster.py` — a Python script (or notebook exported to `.py`/`.md`) that pulls a feature matrix from your combined SQL view, scales it, sweeps `k` from 2–7 with elbow + silhouette, and fits the best `k`.
3. `segment_report.md` — the profile-and-recommendation write-up (structure below).
4. `results.csv` — the final per-customer table: `customer_id`, `company_name`, `rfm_segment`, `behavior_segment`, `cluster`, and the 5 numeric features, exported from your pipeline (`df.to_csv("results.csv", index=False)` is fine).

Everything must run against the Crunch Flow seed on PostgreSQL or SQLite for the SQL half, and pandas/scikit-learn for the clustering half. **No spreadsheet touches this data at any point** — `results.csv` is an *export* for the record, not a working file you edit by hand.

---

## Step 1 — Build the combined segmentation table (45 min)

In `segmentation.sql`: chain the `rfm_base` → `NTILE` scoring → `rfm_segment` CASE from Lecture 1, the usage-profile → `behavior_segment` CASE from Lecture 2, and join both to `customers` on `customer_id`. The final `SELECT` should return, per customer: `customer_id`, `company_name`, `plan`, `mrr`, `recency_days`, `frequency`, `monetary`, `rfm_segment`, `total_events`, `power_events`, `days_since_last_event`, `behavior_segment`. Sanity-check: 30 rows, no `NULL`s in either segment column (the two never-purchased customers must still get a valid `rfm_segment`, not a blank).

## Step 2 — Cluster in pandas (45 min)

In `cluster.py`: `pd.read_sql_query` your Step 1 view (or re-derive the 5 numeric features directly — your choice), scale with `StandardScaler` (remembering the `recency_days` clip from Lecture 3), sweep `k=2..7` printing inertia and silhouette for each, and fit the `k` with the best silhouette score. Add the resulting `cluster` label back onto the `DataFrame` and export `results.csv`.

## Step 3 — Profile every group, three ways (45 min)

In `segment_report.md`, build three tables:

1. **RFM segment profile** — for each `rfm_segment`: `n`, `total_mrr`, `total_addon_spend`, `avg_power_events`.
2. **Behavioral segment profile** — for each `behavior_segment`: `n`, `avg_total_events`, `avg_power_events`, `% of segment that's on the Scale plan`.
3. **Cluster profile** — for each `cluster`: `n`, and the mean of all 5 numeric features, plus a one-sentence persona name you assign.

## Step 4 — Write the recommendation (30 min)

In `segment_report.md`, close with:

- **The single highest-value segment to target next quarter** — you may name it using any of the three lenses (an `rfm_segment`, a `behavior_segment`, or a `cluster`), but your choice must be justified using evidence from **at least two of the three** profiles built in Step 3.
- **The specific action** — one paragraph, concrete enough to execute (same bar as Challenge 2's targeting cards): trigger query, action, and a 60-day success metric.
- **One place the three techniques disagreed** — name a customer or small group where RFM, behavioral tier, and cluster assignment didn't all tell the same story, and explain what you think is actually true about that customer.
- **One honest limitation** of your model — something you know this segmentation gets wrong or can't see, in the spirit of Lecture 3 section 6's discussion of `Loyal Core` vs. `At Risk` merging in the cluster solution. What data would you need to add to fix it?

---

## Rules

- **No spreadsheets, anywhere, at any point.** `results.csv` is a machine-generated export, not a place you go to sort or filter by hand.
- **Every claim in `segment_report.md` must be traceable** to a specific query in `segmentation.sql` or a specific `groupby` in `cluster.py` — if a reader can't find where a number came from, rewrite the sentence to say where it came from.
- **State your assumptions.** Any threshold you changed from the lecture defaults (a different `k`, a different power-events cutoff, a different recency window) gets one sentence explaining why.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| SQL segmentation pipeline | 25% | Correct `NULL` handling, correct `NTILE` inversion, all 30 customers land in exactly one `rfm_segment` and one `behavior_segment` |
| Clustering pipeline | 20% | Correct scaling, an honest `k` selection backed by the silhouette sweep (not just copying Lecture 3's `k=5`) |
| Segment profiling | 20% | All three profile tables built from real queries/groupbys, not estimated |
| Recommendation | 20% | Highest-value segment named with evidence from ≥2 lenses; action is concrete and measurable |
| Honesty about limitations | 15% | Names a real disagreement between techniques and a real gap in the model, not a token caveat |

---

## Reflection (part of `segment_report.md`, ~200 words)

1. Where did the three segmentation techniques agree most strongly, and why do you think that particular pattern was easy for all three to find independently?
2. Where did they disagree, and what does that disagreement tell you about the limits of a snapshot feature matrix (recall Lecture 3 section 6)?
3. If Crunch Flow gave you one more table to add to this analysis (subscription-cancellation history, support-ticket volume, NPS scores — your choice), which would most improve your recommendation, and specifically what would it let you measure that you can't measure today?
4. Rank the three techniques (RFM, behavioral, clustering) by how much you'd trust each one **on its own**, with no other evidence, to flag a customer about to churn. Defend the ranking in two sentences.

---

## Why this matters

Every later week of C38 that touches customers — pricing experiments, forecasting, retention interventions — will assume you can build exactly this: not one clever segmentation technique in isolation, but a synthesized view that a real operator can act on with numbers behind every claim. This project is the first time you've had to make all three techniques agree to disagree honestly, in public, in a document someone else will read and act on.

When done: push, then take the [quiz](../quiz.md).
