# Week 8 — Homework

Five problems, ~5 hours total, spread across the week. A mix of hands-on SQL/pandas against the `checkout_sessions` seed, written analysis, and one build-your-own-data task. Commit each.

All queries run against the `checkout_sessions` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Reproduce every headline number (60 min)

Without looking back at the lecture files, compute each of these from memory (peek only if truly stuck, then note which ones you peeked at):

1. Conversion rate, both arms, plus the raw counts.
2. The two-proportion z-test and p-value on the primary metric.
3. The 95% confidence interval for the difference in conversion rate.
4. AOV per arm, and the relative change.
5. The SRM chi-square check on the two arm sizes.

**Deliver** `reproduce.py` (or `.sql` + `.py`) with all 5 computations and a one-line note on which (if any) you needed to check a lecture for.

---

## Problem 2 — Device-level breakdown (60 min)

Lecture 2 and Exercise 2 computed the headline numbers **overall**. This problem breaks conversion rate and AOV down **by `device`**, the same discipline Lecture 3, section 5 used to check for Simpson's paradox.

1. Conversion rate by `variant, device` (4 rows).
2. AOV by `variant, device` (converted sessions only).
3. Compute each device segment's **share of sessions** within its arm (e.g., mobile is what % of control's 50 sessions vs. what % of treatment's 50?). Is the split even enough between arms that a Simpson's-paradox-style reversal is unlikely to be hiding here?
4. Write two sentences: does the treatment effect (higher conversion) hold in **both** device segments, or does one segment tell a different story than the overall number?

**Deliver** `by-device.sql` (or `.py`) with the 3 computations, plus `by-device-notes.md` with the two sentences.

*(Spot check: mobile conversion goes from 43.3% to 66.7%; desktop from 40.0% to 60.0% — both segments move the same direction as the overall result.)*

---

## Problem 3 — Explain the concepts, precisely (45 min)

In `concepts-writeup.md`, answer in prose (no more than 450 words total), as if writing for a product manager who has never taken a statistics course:

1. Explain the difference between α (significance level) and power, using a one-sentence LoopCart example for each — specifically, what mistake does each one guard against?
2. Explain why a 95% confidence interval matters even when a result is already "statistically significant" — use the pilot's actual interval ([+2.9, +41.1] percentage points) as your example.
3. Explain, to someone who wants to "just check the dashboard every morning and call it once it looks good," precisely why that policy inflates the real false-positive rate above 5% — reference the specific formula from Lecture 3, section 4.
4. Name one guardrail metric from this week's material that you think would need a *longer* observation window than the primary metric to be trustworthy, and explain why in one sentence.

---

## Problem 4 — The peeking trajectory (60 min)

The pilot's `assigned_at` timestamps span a full week. This problem makes you watch what "checking every day" would have actually shown a LoopCart analyst watching the test live.

1. For each cutoff date **2026-06-01 through 2026-06-07** (cumulative, i.e. "all sessions with `assigned_at` on or before this date"), compute: sessions per arm, conversions per arm, z-score, and p-value.
2. Put the 7 rows in a markdown table in `peeking-trajectory.md`, then write 3 sentences: on which day would a "stop at first p<0.05" policy have ended the test, how many days early is that versus the full 7-day window, and — separately from whether this *particular* draw happened to be a true effect — why that policy is still the wrong one to have in place going forward.
3. Write the **single SQL query** (not seven separate ones) that produces all 7 cumulative `(day, variant, cum_sessions, cum_conversions)` rows in one result set, using a window function over `DATE(assigned_at)` (Lecture 3, section 1 sketched the shape of this query — write your own version, don't copy it).

**Deliver** `peeking-trajectory.md` (table + reasoning) and `peeking.sql` (the single window-function query).

*(Spot check: by day 5, cumulative n=36/36, p ≈ 0.059 — still just above the 0.05 line; by day 6, n=43/43, p ≈ 0.031 — now below it.)*

---

## Problem 5 — Extend the dataset, then size a follow-up (75 min)

Make the data your own and use it to practice sizing a real follow-up test — same spirit as prior weeks' extension homework, applied to an experiment instead of a single table.

1. Invent **20 new `checkout_sessions` rows** (`session_id` 101–120, `visitor_id` 1101–1120): 10 more control, 10 more treatment, with a **much smaller, more realistic** conversion gap than the pilot's (e.g., control converts at roughly the pilot's 42% rate, treatment at something like 46–50% — a modest, believable lift, not the pilot's dramatic jump).
2. Load them (you now have 120 rows total).
3. Recompute the overall z-test and p-value with your 20 new rows included. Note in `extend-notes.md` whether adding a smaller, more realistic effect on top of the pilot's data pulls the combined result's significance up, down, or leaves it about the same — and why that makes sense given what a smaller true effect does to the numerator of the z-test formula.
4. Using **only your 20 new rows'** rates as the baseline and target (i.e., pretend the pilot never happened and this modest sample is your only signal), run the sizing formula from Exercise 1 to compute how many sessions per arm a *properly powered* version of your smaller-effect test would actually need.
5. Then **undo** it cleanly: delete your 20 new rows and confirm you're back to 100 rows / the original counts.

**Deliver** `extend.sql` (inserts, the recomputed z-test, and the cleanup) plus `extend-notes.md` (your reasoning for steps 3–4).

---

## Time budget

| Problem | Time |
|--------:|-----:|
| 1 | 60 min |
| 2 | 60 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 75 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
