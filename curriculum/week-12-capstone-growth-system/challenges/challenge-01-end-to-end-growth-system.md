# Challenge 1 — Build the End-to-End Growth System

**Time:** ~2 hours. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Every mart you've built this week so far exists as a one-off `CREATE TABLE`/`CREATE VIEW` you typed by hand, in whatever order you happened to work through the exercises. That's fine for learning; it's not fine for a warehouse someone else has to trust. A teammate — or you, in a month, after the schema has drifted — needs to be able to run **one script** and get the entire growth system back, byte-for-byte identical, from a fresh database. That's the job this challenge asks you to do: turn Exercises 1–3 into a single idempotent build, and add the one mart none of the exercises built directly — a proper retention-cohort **view**, queryable by anyone without re-deriving the age-window CTE from scratch every time.

## Your task

Produce a `challenge-01.md` plus a `build.sql` (or a small set of numbered `.sql` files, your call — document the choice) containing:

1. **One idempotent build script** that, run twice in a row against a fresh `crunch_boards` database (raw tables already seeded), produces an **identical** warehouse both times. Use the truncate-and-reload or `DROP...CREATE` pattern from Week 6, Lecture 3 — whichever you choose, state it in `challenge-01.md` and say why. It must build, in dependency order: all five `stg_*` views, `dim_date`, `dim_user`, `fct_acquisition`, `fct_mrr_monthly`, `fct_unit_economics`, `fct_experiment_results`, and the new `vw_retention_cohorts` from Task 2.
2. **`vw_retention_cohorts`** — a reusable **view** (not a one-off query) that returns, for every `(signup_channel, age_months)` pair with at least one observation, the pooled retention percentage and the underlying `n`. This is Exercise 2 Task 4's query, generalized into something a teammate could `SELECT * FROM vw_retention_cohorts WHERE signup_channel = 'referral';` against forever, instead of re-copying a CTE. Add a `n_is_thin` boolean column, `TRUE` when `n < 10` for this dataset's scale (state in your doc why you picked 10 rather than Week 5's 15 — hint: this capstone's whole warehouse is smaller than Week 5's seed).
3. **A run log.** Run your build script **twice**, back to back, and paste the row counts of every table/view both times into `challenge-01.md` as proof of idempotency. If any count differs between runs, your build isn't idempotent yet — find and fix the bug before submitting.
4. **A dependency diagram** (ASCII is fine, following this week's README/lecture style) showing which objects depend on which, so a teammate knows what order things must build in.
5. **One paragraph** naming one thing you would add to this warehouse if the capstone continued for another month (a new source table, a new fact grain, a data test) and why it's the next highest-priority gap, not just a random nice-to-have.

## Constraints

- The build script must be **runnable start to finish with one command** (`psql -f build.sql crunch_boards` or `sqlite3 crunch_boards.db < build.sql`) — no manual steps between statements.
- Every `stg_*`/`dim_*`/`fct_*` object from Exercises 1–3 must be included — this challenge does not ask you to redesign them, only to assemble and harden what you already built.
- `vw_retention_cohorts` must be a real `CREATE VIEW`, not a saved query string in a comment.
- No spreadsheet, no external orchestration tool — one SQL file (or a small numbered set), run by the database engine itself.

## Hints

<details>
<summary>On idempotency for a mixed views + tables warehouse</summary>

Views (`stg_*`) are naturally idempotent — `CREATE OR REPLACE VIEW` (Postgres) or `DROP VIEW IF EXISTS` + `CREATE VIEW` (both engines) always reflects current raw data with no state to go stale. Tables (`dim_user`, `fct_*`) are the ones that need explicit handling — either `DROP TABLE IF EXISTS x; CREATE TABLE x AS ...` (simplest, safe for a warehouse this size) or a `TRUNCATE`/`DELETE` + `INSERT` pattern (matches Week 6's convention more closely, and matters more once tables get large enough that `DROP`/recreate becomes slow — not a concern at 72 rows, but name the tradeoff in your doc anyway).

</details>

<details>
<summary>On the "thin" threshold</summary>

Week 5's Lumen Metrics dataset had 36 and 28 customers per channel — its "don't trust n<15" rule reflects that scale. Crunch Boards has 8–12 paying customers per channel. Applying Week 5's literal threshold here would flag almost every data point as unusable, which is true in an absolute-precision sense but useless as a working rule for a warehouse this size. Picking a scaled-down threshold and *saying why* is the actual skill — a rule of thumb that ignores your dataset's actual scale isn't rigor, it's cargo-culting a number from a different context.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Idempotency | Asserted, not shown | Two full run logs, side by side, with identical counts |
| `vw_retention_cohorts` reusability | A `SELECT` copy-pasted from Exercise 2 | A real view with `n` and `n_is_thin` columns, queryable by channel with a `WHERE` |
| Dependency clarity | Objects built in whatever order compiles | An explicit diagram + a build order a new teammate could follow without asking you a question |
| Threshold reasoning | Reused Week 5's `n<15` without comment | States the capstone's smaller scale and picks (and justifies) an appropriately scaled threshold |
| Forward-thinking | Skips the "what's next" paragraph or names something trivial | Names a specific, well-reasoned next gap tied to a real limitation surfaced this week |

## Submission

Commit `challenge-01.md` and `build.sql` (or your numbered set) to your portfolio under `c38-week-12/challenge-01/`.
