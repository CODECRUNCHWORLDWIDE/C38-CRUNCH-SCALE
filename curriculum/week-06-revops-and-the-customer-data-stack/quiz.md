# Week 6 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 7. A mix of multiple-choice and short "what does this return / what would happen?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In the raw → staging → marts pattern, which statement is correct?

- A) Raw data should be cleaned and corrected in place as soon as errors are found.
- B) Raw data is landed exactly as the source system gave it and never modified; corrections happen in staging or later.
- C) Staging tables should contain business-level joins across many source systems.
- D) Marts are optional — most teams query staging directly.

---

**Q2.** What is the key practical difference between ETL and ELT?

- A) ETL is newer and always faster.
- B) In ELT, transformation happens inside the warehouse in SQL, after raw data is loaded unmodified; in ETL, transformation happens before loading.
- C) ELT never uses SQL.
- D) There is no real difference; the terms are interchangeable.

---

**Q3.** In this course's plain-SQL sense, a "semantic layer" is best described as:

- A) A separate paid software product you must install.
- B) The place — a mart's SQL definition plus a written glossary — where a metric like MRR is defined exactly once, so everyone queries the same computation instead of re-deriving it.
- C) A synonym for the raw layer.
- D) A type of database index.

---

**Q4.** `stg_subscriptions` in this week's design intentionally keeps **both** of Ben's Stripe subscription rows (an old canceled one and a new active one) rather than deduping to only the latest. Why?

- A) Because Stripe requires it.
- B) Because both rows represent genuinely distinct real-world subscription periods, and a monthly revenue history needs to attribute January/February revenue to the old plan and March onward to the new one.
- C) Because SQL doesn't allow filtering out old rows.
- D) It's a mistake that should be fixed by deduping.

---

**Q5.** What does "conformed" mean when describing `dim_user`?

- A) It only contains active customers.
- B) It is built exactly once and reused, unmodified, as the join target for every fact table in the warehouse — no fact table builds its own competing user dimension.
- C) It is a temporary table dropped after each query.
- D) It stores revenue data directly.

---

**Q6.** Why does `fct_mrr_monthly` join against a `dim_date` spine (`CROSS JOIN dim_user`) rather than simply grouping `stg_subscriptions` by month?

- A) `CROSS JOIN` is required syntax in PostgreSQL.
- B) Without the spine, a user with $0 revenue in a given month (before signup, or after cancellation) would be missing from the result entirely instead of correctly showing `0`.
- C) It's purely a performance optimization with no correctness impact.
- D) `dim_date` is required for `GROUP BY` to work at all.

---

**Q7.** An annual subscription's `amount_cents` is normalized to a monthly figure:

- A) Every time a query touches it, using a `CASE` in each query.
- B) Once, at the staging boundary (`stg_subscriptions`), so every downstream mart and query can just sum `mrr_cents` without re-deriving the conversion.
- C) Never — annual plans are excluded from MRR entirely.
- D) In the raw layer, before it's inserted.

---

**Q8.** A subscription is considered active for a given month in this week's `fct_mrr_monthly` design if:

- A) It was ever active at any point during the month, even briefly.
- B) It started on or before that month's last day, and either was never canceled or was canceled after that month's last day (i.e., still active "as of the last day of the month").
- C) Its `subscription_id` is odd-numbered.
- D) `raw_events` contains an `activated` event for that user that month.

---

**Q9.** Elin (user 5) signed up and canceled entirely within February 2025. Every one of her rows in `fct_mrr_monthly` shows `mrr_cents = 0`, for every month. What does this demonstrate?

- A) A bug in the `fct_mrr_monthly` query that must be fixed before the mini-project.
- B) A known, real limitation of month-end MRR snapshots: a customer who both starts and fully churns within a single calendar month contributes to no monthly snapshot at all, even though they generated real revenue.
- C) That Elin was never a real paying customer.
- D) That `dim_date` is missing a row for February.

---

**Q10.** Which pattern is generally preferred in this course for rebuilding a small mart table (thousands, not billions, of rows) on every pipeline run?

- A) `INSERT`-only, with no deletion — old rows are simply left in place.
- B) Truncate-and-reload inside a transaction (`TRUNCATE`/`DELETE` then `INSERT`, wrapped in `BEGIN`/`COMMIT`) — simple, correct, and easy to reason about at this scale.
- C) Manually deleting rows one at a time based on a spreadsheet audit.
- D) Dropping and recreating the entire database.

---

**Q11.** What specifically goes wrong if a non-idempotent, `INSERT`-only load is accidentally run twice?

- A) Nothing — SQL automatically prevents duplicate inserts.
- B) The table ends up with duplicate rows for the same key (e.g., two rows for the same `user_id`/`month_date`), silently doubling any `SUM()` computed over it.
- C) The database refuses to start.
- D) Only the most recent insert's rows survive.

---

**Q12.** A data test written as `SELECT user_id, month_date, COUNT(*) FROM fct_mrr_monthly GROUP BY user_id, month_date HAVING COUNT(*) > 1;` is an example of which test category?

- A) Not-null
- B) Freshness
- C) Uniqueness
- D) Accepted values

---

**Q13.** In this week's reconciliation, when Stripe and the internal app database disagree on a customer's plan or price, which system does the warehouse treat as the source of truth for revenue, and why?

- A) The app database, because it has the internal `user_id` already.
- B) Whichever system was updated most recently, checked case by case.
- C) Stripe, because it is the billing system and this warehouse has explicitly designated it as money's source of truth — the app database is used for product/feature-gating state, not revenue.
- D) Neither — disagreements are simply averaged.

---

**Q14.** Which of these is a **specific, defensible** reason a warehouse with tested, idempotent SQL loads beats a spreadsheet as a company's revenue system of record — not just "spreadsheets are old-fashioned"?

- A) Spreadsheets can't display currency symbols.
- B) A warehouse enforces constraints (`UNIQUE`, `NOT NULL`) that reject bad concurrent writes, and a bad number can be traced back to the exact `SELECT` and unmodified raw data that produced it — a spreadsheet's formula chain offers neither guarantee.
- C) Spreadsheets are always slower to open than a database connection.
- D) Databases are free and spreadsheet software costs money.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — raw data is landed as-is and never modified; that untouched copy is your audit trail. Corrections belong in staging (or later), where they're visible and traceable, not silently applied to raw.
2. **B** — ELT loads raw data first and transforms it afterward, inside the warehouse, in SQL. That's exactly the raw → staging → marts pattern this week builds.
3. **B** — no special tool required; it's the SQL definition (in a mart) plus a written glossary entry, both pointing at one computation everyone shares.
4. **B** — Ben's two rows are real, sequential subscription periods, not duplicates. Deduping to "latest only" would have made his January/February revenue silently disappear from `fct_mrr_monthly`.
5. **B** — "conformed" means built once, reused everywhere. If two different fact tables each build their own user dimension, you're back to the three-numbers-for-one-metric problem Lecture 1 opened with.
6. **B** — this is a correctness issue, not a performance one. Without the date spine, a customer with zero revenue in a given month is simply absent from a naive `GROUP BY`, indistinguishable from "we have no data for them," instead of correctly showing `$0`.
7. **B** — normalized once, at the staging boundary, in `stg_subscriptions`. Every mart and query downstream just sums `mrr_cents`.
8. **B** — the "as of the last day of the month" convention, matching the written metric definition in Lecture 3.
9. **B** — this is a real, known, documented limitation of month-end snapshots, not a bug — a customer who starts and churns within one calendar month never appears "active" at any month's end.
10. **B** — truncate-and-reload in a transaction is simple, correct, and appropriate at this scale; upsert (`ON CONFLICT`) is reserved for tables too large to fully rebuild every run.
11. **B** — duplicate rows for the same key, silently doubling any aggregate computed over the table. No error is raised — the number is just quietly wrong.
12. **C** — uniqueness. It returns rows only when the same `(user_id, month_date)` key appears more than once, which is exactly the signature of a doubled, non-idempotent load.
13. **C** — Stripe is the billing system and is explicitly designated as revenue's source of truth in this warehouse's design (Lecture 2, Section 4); the app database's subscription record is used for product-side concerns instead.
14. **B** — the specific, checkable guarantees (constraint enforcement on writes, full audit trail back to unmodified raw data) that a spreadsheet's formula chain simply doesn't provide.

</details>

**Scoring:** 12+ → start Week 7. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; every later week of this course assumes this warehouse is trustworthy.
