# Week 7 — Exercises

Three exercises, ~60 min each, hands-on against the Crunch Flow seed data. Do them in order — Exercise 3 pulls its feature matrix from queries you'll have already written by hand in Exercises 1 and 2.

1. **[Exercise 1 — Build RFM scores in SQL](exercise-01-build-rfm-scores.md)** — write the full recency/frequency/monetary/`NTILE`/tier pipeline yourself, from a blank query editor.
2. **[Exercise 2 — Build a behavioral cohort](exercise-02-behavioral-cohort.md)** — segment customers by feature usage instead of purchase history, and cross-tab against Exercise 1's tiers.
3. **[Exercise 3 — Cluster customers with k-means](exercise-03-kmeans-clusters.md)** — pull a feature matrix into pandas, scale it, choose `k`, and interpret the clusters.

## Before you start

- You've read all three lectures.
- You loaded the seed data from the [week README](../README.md), and `SELECT COUNT(*) FROM customers; SELECT COUNT(*) FROM orders; SELECT COUNT(*) FROM product_events;` return **30**, **125**, **393**.
- You have a `psql crunch_scale_w7` (or `sqlite3 crunch_scale_w7.db`) session open, and for Exercise 3, a Python environment with `pandas` and `scikit-learn` installed (`pip install pandas scikit-learn`).

## Suggested workflow

- Write each query **before** running it. Predict roughly what you expect the output to look like (how many rows, which customers should end up in which tier), then run it and check your prediction. When you're wrong, that gap is exactly where the learning happens — don't just fix the query and move on without figuring out *why* you were wrong.
- Save your work: `exercise-01.sql`, `exercise-02.sql`, and `exercise-03.py` (or a notebook). You'll build on all three in the challenges and the mini-project.
- "Today" is **2025-12-31** for every recency calculation this week — say it explicitly in a comment at the top of each file so a reader never has to guess.

## Data tooling reminder

Every number in this week's exercises comes from a SQL query against `customers`/`orders`/`product_events`, or a pandas `DataFrame` built from one. If you catch yourself wanting to paste rows into a spreadsheet to eyeball a sort — don't. Write the `ORDER BY`, or the pandas `.sort_values()`, instead.
