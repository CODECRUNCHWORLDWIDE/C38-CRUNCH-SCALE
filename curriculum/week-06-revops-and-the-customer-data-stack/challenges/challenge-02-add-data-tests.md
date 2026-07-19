# Challenge 2 — Add Data Tests to a Warehouse

**Time:** ~90 minutes. **Difficulty:** Medium. **No single right answer on coverage, but every test must be correct.**

## The scenario

Exercise 3 found three real discrepancies in Crunch Flow's data by hand. That's not sustainable — a RevOps analyst doesn't want to manually audit every subscription every week. Your job this challenge: build a **SQL test suite** that would have caught all three discrepancies (and a handful of other realistic failure modes) automatically, the moment they entered the warehouse, using nothing but plain `SELECT` statements that return zero rows when healthy.

## Your task

Produce a `challenge-02.md` (your design notes) plus a `tests.sql` file containing **at least 10 tests**, each as a single `SELECT` that returns **zero rows on healthy data**. Your suite must include, at minimum, one test from each category below (Lecture 3, Section 2):

1. **Not-null** — at least one required column on `dim_user` or `stg_subscriptions`.
2. **Uniqueness** — `fct_mrr_monthly` must never have two rows for the same `(user_id, month_date)`. This is the single most important test in the whole suite — it's the one that would have caught the naive-reload bug from Lecture 3, Section 1.
3. **Accepted values** — `plan` in any staging or mart table must only ever be `Starter`, `Growth`, or `Scale` — never a legacy name (`Basic`/`Pro`/`Enterprise`) leaking through unconformed, and never a typo.
4. **Referential integrity** — every `fct_mrr_monthly.user_id` exists in `dim_user`; every `fct_mrr_monthly.month_date` exists in `dim_date`.
5. **Range / sanity** — `mrr_cents` is never negative; a plan's `mrr_cents` roughly matches its known list price (catch Kira's billing-cycle bug specifically — write a test that would flag her $24.92 as implausible for a Scale-tier row).
6. **Cross-source reconciliation** — a test that fails if `stg_subscriptions_current` and `stg_app_subscriptions` (both `status = 'active'`) disagree on `plan` or `mrr_cents` for the same `user_id`, **and** a second test that fails if a user is `active` in one system but absent/inactive in the other (this is Exercise 3, Tasks 2 and 3, turned into permanent, automatic tests).
7. **Freshness** — the mart's most recent `month_date` is not implausibly old relative to `CURRENT_DATE`.

For each test, write in `challenge-02.md`:

- **What it checks** (one sentence).
- **What real-world failure it catches** — tie it to a specific, concrete scenario (a stuck webhook, a fat-fingered price, a duplicated load), not a vague "data quality."
- **Confirm it currently passes** (returns 0 rows) against this week's seed data — except the two cross-source reconciliation tests from item 6, which should **intentionally fail**, returning exactly the 3 discrepancy rows from Exercise 3. Explain in writing why a failing test here is *correct* — the data genuinely has bugs your Exercise 3 work already found and documented, and the test suite is supposed to surface them, not hide them.

Finally, write a short `run_tests.sql` (or a one-paragraph description if you'd rather script it in Python/pandas) that runs the whole suite in sequence and would make a CI-style script exit non-zero the moment any test returns a row.

## Constraints

- Every test is a plain `SELECT`. No dbt, no external testing framework — the whole point is that this pattern needs no special tooling.
- A test that can never fail (e.g., `SELECT 1 WHERE FALSE` with no real condition) doesn't count — every test must be traceable to a real column and a real failure mode.
- At least 2 of your 10+ tests must be ones **not** explicitly listed above — invent a failure mode from this week's data that you think a real RevOps pipeline should guard against, and justify it.

## Hints

<details>
<summary>On the "range / sanity" test for Kira's bug</summary>

You know the three real plan prices: Starter $29, Growth $99, Scale $299 (monthly-normalized). A reasonable sanity test asserts every `mrr_cents` value in `fct_mrr_monthly` where `mrr_cents > 0` is within, say, 5% of one of those three known prices (Opal's annual-normalized $249.17 needs its own tolerance band, or an explicit allowlist — decide and document which). Kira's stale $24.92-equivalent figure, if it ever leaked into a mart, would fail this test loudly instead of silently reporting a customer paying implausibly little for a Scale plan.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Coverage | Only the easy not-null/uniqueness tests | All 7 categories represented, plus 2 self-invented tests |
| Correctness | A test that "looks right" but doesn't actually fail on the seeded bugs | Verified: the reconciliation tests genuinely return the 3 known discrepancy rows |
| Real-world grounding | Generic "data quality" language | Each test ties to a specific, named failure mode (stuck webhook, typo, duplicate load) |
| Judgment | Tests everything indiscriminately | Explains *why* a failing test here is correct, not a bug to "fix" by loosening the test |

## Submission

Commit `challenge-02.md`, `tests.sql`, and `run_tests.sql` to your portfolio under `c38-week-06/challenge-02/`.
