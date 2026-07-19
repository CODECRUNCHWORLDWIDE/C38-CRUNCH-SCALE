# Exercise 2 — Design a Three-Tier Package

**Goal:** Map all 30 `accounts_usage` rows into the `Starter`/`Growth`/`Scale` tiers from Lecture 2, using the exact caps and prices given below, and measure the MRR impact — in SQL with `CASE`, and again in pandas.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM accounts_usage;   -- must print 30
SELECT SUM(current_price) FROM accounts_usage;  -- must print 1170.00
```

Use these tier definitions throughout (the same ones Lecture 2 derived from the data — don't redesign them here, Challenge 1 is where you'll adjust a tier):

| Tier | MTU cap | Price |
|---|---:|---:|
| Starter | `mtu <= 3000` | $29.00 |
| Growth | `mtu <= 30000` | $69.00 |
| Scale | uncapped | $179.00 |

Create `solutions.sql` and `analysis.py`.

## Tasks

### SQL (Tasks 1–4)

1. **Assign tiers.** Write the `CASE` query that returns `account_id, segment, mtu, current_price, new_tier, new_price` for all 30 accounts, ordered by `mtu`. *(Expected: account 10, `indie`, mtu 2900 → `Starter`/$29; account 11, `team`, mtu 4200 → `Growth`/$69; account 21, `agency`, mtu 35000 → `Scale`/$179.)*

2. **Per-tier rollup.** `GROUP BY new_tier` on your Task 1 query and compute `COUNT(*)` and `SUM(new_price)` per tier. *(Expected: `Starter` n=10, MRR $290.00; `Growth` n=10, MRR $690.00; `Scale` n=10, MRR $1,790.00.)*

3. **Total impact.** Compute `SUM(current_price)` and `SUM(new_price)` across all 30 accounts in one query, plus the dollar and percent uplift. *(Expected: old $1,170.00 → new $2,770.00, uplift +$1,600.00 / +136.8%.)*

4. **Winners and losers.** Count how many accounts have `new_price < current_price`, how many have `new_price > current_price`, and how many are unchanged. *(Expected: 10 accounts see a price **decrease**, 20 see an increase, 0 unchanged.)*

### pandas (Tasks 5–6)

5. **Rebuild Task 1–3 in pandas.** Load `accounts_usage` into a `DataFrame` (typed in by hand or via `pandas.read_sql`), write the tier-assignment function, and reproduce the Task 3 total-impact numbers. Confirm they match exactly.

6. **The segment-level story.** Group your pandas result by `segment` (not `new_tier` — segment is the original label) and compute `mean(current_price)` vs `mean(new_price)` per segment. Write two sentences: which segment's average bill fell, and by how much? Which segment's average bill rose the most, in dollar terms?

## Expected result (spot checks)

- Task 2 → `Starter` n=10 / $290.00; `Growth` n=10 / $690.00; `Scale` n=10 / $1,790.00.
- Task 3 → old MRR $1,170.00, new MRR $2,770.00, uplift +$1,600.00 (+136.8%).
- Task 4 → 10 decreases, 20 increases, 0 unchanged.
- Task 6 → `indie` average bill: $39.00 → $29.00 (−$10.00). `agency` average bill: $39.00 → $179.00 (+$140.00, the largest per-account increase).

## Done when…

- [ ] `solutions.sql` has all 4 SQL tasks, matching the expected values exactly.
- [ ] `analysis.py` reproduces the same total-impact numbers independently in pandas.
- [ ] You can explain, out loud, why "MRR went up 136.8%" is not, by itself, a reason to ship this change unchanged tomorrow — name the rollout problem it creates (this is a preview of Lecture 3).

## Stretch

- Recompute the whole exercise with the `Growth` cap moved from 30,000 to 50,000 MTU (same $69 price). Which accounts change tier? What happens to total MRR, and why is a higher cap **not** simply "better for customers" — think about which accounts would now be *underpriced* relative to Lecture 1's `agency` WTP range.
- For the 10 accounts that got a price **decrease**, compute what ScopeIQ's total MRR would have been if those 10 had instead been left on the flat $39 (i.e., no discount for `indie` — only price the `team`/`agency` increases). How much MRR would ScopeIQ be leaving on the table by NOT lowering the `Starter` price, in the form of avoidable churn risk from the segment most likely to feel overcharged? (There's no single numeric answer here — argue it from Lecture 1's `indie` WTP range.)

## Submission

Commit `solutions.sql` and `analysis.py` to your portfolio under `c38-week-09/exercise-02/`.
