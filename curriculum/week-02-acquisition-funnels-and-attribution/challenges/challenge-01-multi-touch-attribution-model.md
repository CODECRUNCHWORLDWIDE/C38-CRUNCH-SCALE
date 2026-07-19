# Challenge 1 — Build a Multi-Touch Attribution Model

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer** for Part 3 (but Parts 1–2 do have checkable numbers).

## The scenario

Crunch's growth lead has seen Lecture 2's first-touch vs. last-touch swing ($49 → $198 for `Google Search - Brand`) and doesn't trust either one alone anymore. She wants two things from you: a **linear** model and a **position-based (40/20/40)** model, built independently — not copy-pasted from the lecture notes — and reconciled to the penny. Then she wants your opinion on a third model she's heard of but never seen built: **time-decay**, which gives more credit to touches closer to the conversion.

## Part 1 — Linear attribution, from scratch

Without re-reading Lecture 2 §4, write a query that:

1. Counts each user's total touches.
2. Splits each conversion's revenue evenly across that user's touches.
3. Sums the split revenue by channel.
4. Reconciles: `SUM` of your `revenue_credited` column across all channels must equal `SELECT SUM(revenue) FROM conversions` ($1,590).

*(Expected, to check yourself against: Organic Search $323.50, Email $547.67, Meta Ads $198.00, Affiliate $174.50, Google Search - Nonbrand $173.17, Google Search - Brand $98.67, Referral $74.50.)*

If your numbers don't match, the usual culprit is integer division truncating the split (`revenue / n_touches` with both as integers) — fix with an explicit float cast or `* 1.0`.

## Part 2 — Position-based (40/20/40), from scratch, with edge cases named

Build the position-based model. Before you write SQL, write down **in a comment** your rule for each case:

- Exactly 1 touch → ?
- Exactly 2 touches → ?
- 3 or more touches → ?

(Lecture 2 §5 used 100% / 50-50 / 40-20-40-split-among-middle. You may use the same rule or propose your own — but state it explicitly and be consistent.) Then implement it and reconcile to $1,590 the same way as Part 1.

*(Expected if you use the lecture's rule: Organic Search $323.50, Email $557.60, Google Search - Nonbrand $183.10, Affiliate $174.50, Meta Ads $168.20, Google Search - Brand $108.60, Referral $74.50.)*

## Part 3 — Propose a time-decay model (open-ended)

Time-decay attribution gives touches closer to the conversion more credit than touches further away, using a decay function instead of fixed positions. A common form: weight a touch by `2^(-days_before_conversion / half_life)`, then normalize each user's weights to sum to 1 before applying them to revenue.

1. Pick a half-life (7 days is a reasonable default given this dataset's ~2–5 day conversion windows — but justify your choice).
2. Implement it in SQL. You'll need `days_before_conversion` per touch (conversion date minus touch date), the decay formula (`POWER(0.5, days / half_life)` works on Postgres; SQLite needs `POWER` emulated as `EXP(days / half_life * LN(0.5))`), and a per-user weight sum to normalize against.
3. Run it, reconcile to $1,590, and compare the result to your Part 1 and Part 2 tables for at least three channels.
4. **Write 200–300 words** recommending, to Crunch's growth lead, which of the three models (linear, position-based, time-decay) she should adopt as the team's default — and which one or two she should keep running *alongside* it as a sanity check. There's no required answer; defend your reasoning with numbers from your own queries.

## Constraints

- Every model's total must reconcile to $1,590. An unreconciled model is a bug, not a valid answer.
- State your edge-case rules **before** writing the SQL that implements them, not after seeing the output.
- Part 3 requires actual SQL, not just prose about what time-decay "would" do.

## How success is judged

| Signal | Weak | Strong |
|--------|------|--------|
| Reconciliation | Totals don't match $1,590, unnoticed | Every model reconciles; mismatches are caught and fixed |
| Edge-case handling | Silent, inconsistent, or crashes on 1-/2-touch users | Explicit, stated rule applied uniformly |
| Time-decay implementation | Hand-waved, no working query | Runs, reconciles, produces per-channel numbers |
| Recommendation (Part 3.4) | Picks a model with no numeric support | Cites specific channel-level differences across models to justify the pick |

## Stretch

- Make the half-life a query parameter (a CTE literal you can change in one place) and re-run with a 3-day and a 14-day half-life. How sensitive is `Google Search - Brand`'s credit to that choice?
- User 9 touched Meta Ads twice before converting via Email. Under time-decay, does the *second* Meta touch (closer to conversion) outweigh the *first*? Compute both weights and confirm.

## Submission

Commit `challenge-01.md` (with your written reasoning and edge-case rules) and `challenge-01.sql` (all queries) to your portfolio under `c38-week-02/challenge-01/`.
