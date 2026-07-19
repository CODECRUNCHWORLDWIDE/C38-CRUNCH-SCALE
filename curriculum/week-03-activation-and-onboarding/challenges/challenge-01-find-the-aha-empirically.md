# Challenge 1 — Find the Aha Moment Empirically

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch Boards' head of growth has seen Lecture 2's lift table and is not fully convinced. She asks you three pointed questions:

> "`created_first_card` has +32.9pp lift and 52.2% reach — way more reach than `invited_teammate`'s 29%. Why not just make *that* the activation event, since more people would count as 'activated'?"
>
> "`completed_first_task` has +47.6pp lift, almost as good as `invited_teammate`'s +50.4pp. If I had to pick one to put a big 'you're activated!' banner on, why does the *order* of the two events matter at all — a lift's a lift, isn't it?"
>
> "Show me the actual query, not just a table somebody could have typed by hand."

Your job is to answer all three — in writing, backed by SQL you actually ran.

## Your task

Produce a short memo (`aha-memo.md`) and a supporting `aha-analysis.sql`, structured as:

1. **The full lift table.** Extend the Lecture 2 §2 `flags` CTE to cover **all six** candidate events (`verified_email`, `created_first_board`, `created_first_card`, `invited_teammate`, `completed_first_task`, `connected_integration`), and produce one query that outputs `candidate`, `n_did`, `pct_reach`, `retained_if_did`, `retained_if_not`, `lift_pp` for all six in a single result set (not six separate hand-copied blocks — use `UNION ALL` over six `SELECT`s against the same `flags` CTE, or an unpivoted approach if you're comfortable with one).

2. **Answer the reach question.** Directly address the head of growth's first question: why doesn't higher reach alone make `created_first_card` the better choice, even though it has almost double the reach of `invited_teammate`? Reference the actual lift numbers (32.9pp vs. 50.4pp) in your answer — reach isn't free if the thing more people reach is a weaker signal.

3. **Answer the ordering question.** Directly address the second question — the reverse-causation trap from Lecture 2 §4. Back it up with a query: what fraction of `invited_teammate`-doers go on to also do `completed_first_task`? What fraction of `completed_first_task`-doers had *already* done `invited_teammate` first? Use these numbers to argue why picking the earlier event in a causal chain, not just the higher-lift one, is the correct call — even when the lift numbers are close.

4. **State your final recommendation** — restate the activation event definition from Exercise 1, now backed by the full six-candidate comparison, in no more than 4 sentences.

## Constraints

- Every number in your memo must come from a query you can point to in `aha-analysis.sql`. No hand-typed lift table.
- Address all three of the head of growth's questions explicitly — don't just restate the lecture's conclusion and hope it answers them.
- Acknowledge, in one sentence, what this analysis has *not* proven (tie back to Lecture 2 §5 — correlation vs. causation) and what would upgrade it to proof.

## Hints

<details>
<summary>On the reach vs. lift tradeoff</summary>

Think of it as a 2×2: high-reach/low-lift events are common but weak signals (almost everyone does them, so they don't discriminate who retains); low-reach/high-lift events (like `connected_integration`) are strong signals but too rare to be an org-wide metric. The best activation event sits in the high-lift, *workable*-reach corner — it doesn't need to be the single highest-reach candidate, it needs to be the best available combination of the two.

</details>

<details>
<summary>On the ordering query</summary>

You want something like: `COUNT(*) FILTER (WHERE invited AND completed_task) / COUNT(*) FILTER (WHERE invited)` for "of inviters, what fraction also completed a task" and the mirror image for "of task-completers, what fraction had already invited before completing." If the second number is high, it's strong evidence `completed_first_task` is mostly a *consequence* of having already invited, not an independent signal.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Full lift table | Hardcodes numbers from the lecture | Runs a real six-candidate query and reports its actual output |
| Reach question | "More reach is better" or "lift is better," asserted | Explicitly weighs both axes and explains the tradeoff with numbers |
| Ordering question | Ignores it or answers by assertion | Runs the "who-did-what-first" query and uses its output as evidence |
| Humility | Presents the pick as proven fact | States plainly it's a hypothesis, and names what would prove it (Week 8) |

## Submission

Commit `aha-memo.md` and `aha-analysis.sql` to your portfolio under `c38-week-03/challenge-01/`.
