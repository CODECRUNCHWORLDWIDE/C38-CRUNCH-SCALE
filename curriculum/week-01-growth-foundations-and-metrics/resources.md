# Week 1 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with pandas** — needed starting Week 5 for modeling; installing it now saves a mid-course interruption: `pip install pandas`.

## Required reading (this week's core)

- **Dave McClure, "Startup Metrics for Pirates" (2007)** — the original AARRR talk, still the field's shared vocabulary: search "Dave McClure Startup Metrics for Pirates slideshare" — widely mirrored, free.
  *Why: everything in Lecture 2 traces back to this one deck.*
- **Reforge — "How to Choose a North Star Metric"** (free public blog post): <https://www.reforge.com/blog/north-star-metric>
  *Why: the modern, practitioner-level treatment of exactly Lecture 1's five-question test, from people who've run growth teams at scale.*
- **Amplitude — "North Star Playbook"** (free, no signup required to read): <https://amplitude.com/north-star>
  *Why: dozens of real north-star examples across product categories — good for Exercise 1's "products other than StreakLab."*
- **PostgreSQL — Common Table Expressions (`WITH`):** <https://www.postgresql.org/docs/current/queries-with.html>
  *Why: every metric-tree query this week and beyond leans on CTEs — get fluent now.*
- **PostgreSQL — Aggregate expressions (`FILTER`):** <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES>
  *Why: the clean way to compute multiple conditional counts in one query, used constantly from this week onward.*

## Reference (keep in tabs)

- **PostgreSQL — Date/Time functions and `INTERVAL` arithmetic:** <https://www.postgresql.org/docs/current/functions-datetime.html>
  *Why: trailing-window metrics (like "3+ check-ins in the last 7 days") are pure interval arithmetic — this is the exact reference for it.*
- **PostgreSQL — `JSON`/`JSONB` types:** <https://www.postgresql.org/docs/current/datatype-json.html>
  *Why: the production-grade way to store flexible event properties, previewed in Lecture 3, section 2.*
- **PostgreSQL — Window functions:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: not needed yet, but the natural next step after `FILTER`-based aggregates — Week 4's retention cohorts lean on these heavily.*
- **Segment — "Spec: Tracking Plan" conventions** (free docs, industry-standard event-naming practices): <https://segment.com/docs/getting-started/04-full-installation/>
  *Why: the closest thing the industry has to a style guide for `event_name` vocabularies — compare it to StreakLab's six-event schema.*

## Practice beyond the seed data

- **Amplitude's public demo dashboards** (no signup for the marketing examples): <https://amplitude.com/amplitude-demo>
  *Why: see a real (if idealized) AARRR-style funnel rendered from real product data — useful context before you build your own in the mini-project.*
- **Mode Analytics — "The Analyst's Guide to Growth Metrics"** (free blog series): search "Mode Analytics growth metrics guide"
  *Why: more worked examples of turning ambiguous growth questions into precise SQL, in the same spirit as this course's challenges.*

## Deeper background (optional this week)

- **Sean Ellis & Morgan Brown, *Hacking Growth*** — the book that popularized "growth team" as an operating model; Chapter 2 covers North Star Metrics specifically (library or purchase).
  *Why: the fullest treatment of everything Lecture 1 compresses into two hours.*
- **Alistair Croll & Benjamin Yoskovitz, *Lean Analytics*** — the "One Metric That Matters" framing, an earlier cousin of the north-star concept (library or purchase).
  *Why: useful contrast — OMTM argues for a metric that changes *by stage of company*, a wrinkle worth knowing about even though this course treats the north star as more durable.*

## Glossary

| Term | Definition |
|------|------------|
| **Metric** | A number, computed the same way every time, that summarizes something real. |
| **North-star metric (NSM)** | The single metric a whole company organizes around; a leading indicator of durable customer value. |
| **Input metric** | A smaller, team-ownable metric that mechanically feeds the north star. |
| **Vanity metric** | A metric that can rise without the business getting healthier — usually a raw, spendable, cumulative count. |
| **Actionable metric** | A metric a team can deliberately move, where the movement reliably signals something real. |
| **AARRR** | Acquisition, Activation, Retention, Revenue, Referral — the five-stage growth funnel. |
| **Activation** | The first moment a user experiences a product's core value. |
| **Leading indicator** | A metric that moves *before* the outcome it predicts (e.g., engagement before churn). |
| **Lagging indicator** | A metric that reflects an outcome *after* it happened (e.g., revenue, churn). |
| **Event** | An immutable, timestamped record of something a user did. |
| **Fact table** | A table of events — one row per happening, always timestamped, append-only. |
| **Dimension table** | A table of "nouns" — entities like users, with attributes true at a point in time. |
| **Event schema** | The `event_name` vocabulary plus the columns every event row carries. |
| **Raw / staging / marts** | The three layers of a warehouse: untouched source of truth → cleaned/typed → pre-aggregated for fast querying. |
| **Metric tree** | A north star decomposed into the input metrics that combine to produce it, each with a runnable query. |
| **`FILTER (WHERE ...)`** | PostgreSQL syntax for aggregating over a subset of grouped rows without a `CASE` expression. |

---

*Broken link? Open an issue or PR.*
