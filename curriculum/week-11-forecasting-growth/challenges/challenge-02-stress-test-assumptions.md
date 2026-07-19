# Challenge 2 — Stress-Test a Forecast's Assumptions

**Goal:** A forecast has already been written and approved. Your job isn't to build a new one — it's to find the assumption inside it that's quietly doing too much work, quantify what happens if that assumption is wrong, and propose a fix.

**Estimated time:** 60 minutes.

## The memo

This landed in your inbox. It's short, it's confident, and Finance has already dropped its headline number into next quarter's board deck.

> **To:** Growth & Finance leadership
> **From:** VP of RevOps
> **Re:** Q4 2026 MRR forecast — approved for board deck
>
> Team — here's the number for the deck. **Q4 2026 ending MRR: $165,218.**
>
> Method: bottoms-up bridge, same shape as always. New MRR starts at $5,350 in January and grows $150/month, in line with the trailing trend. Expansion holds at 3.2% and contraction at 0.85% — both straight off the trailing-3-month average, no changes there.
>
> The one update this quarter: **churn.** We shipped the new onboarding flow in June 2025, and the June cohort's revenue retention curve is running meaningfully ahead of every earlier cohort — it's already several points better at every age we've measured. Since onboarding is now permanent for every new signup, and it's *the* biggest lever we've pulled all year, we're modeling churn coming down to **1.5%** for 2026, roughly half where it's been running. This is the single most important line in the model — it's most of why 2026 looks so much stronger than a simple continuation of 2025 would suggest.
>
> Let me know if there are questions before this goes in the deck Friday.

Something in this memo doesn't hold up to the standard this week has been teaching. Find it.

## Tasks

1. **Locate the assumption doing the heavy lifting.** Read the memo again and identify, precisely, which single number the VP is asserting with the least direct support from the data. (Every other assumption in the memo is explicitly tied to "the trailing-3-month average" — one isn't.)

2. **Check the churn claim against the actual bridge data.** Pull `mrr_bridge_actuals` for the months *after* June 2025 (July–December) — the months that should already reflect any company-wide benefit of the new onboarding flow, since existing accounts and any churn happening during this window are covered by it. What is the actual trailing-3-month churn rate, computed the same way Lecture 1 §2 computed it? Does the real number support "churn is heading to 1.5%," or does it contradict it?

3. **Diagnose the specific reasoning error.** The June cohort's revenue-retention curve genuinely *is* running ahead of earlier cohorts (Lecture 2 confirmed this at ages 0–6). Explain, precisely, why "this one new cohort's age-based retention curve is a few points better" does **not** logically support "the whole company's blended monthly churn rate should be cut in half." (Hint: think about what fraction of Crunch Flow's total MRR the June-and-later cohorts actually represent by 2026, versus the much larger base of pre-June customers who were never touched by the onboarding change at all — and think about the difference between a cohort-level revenue-retention curve and a company-level monthly churn rate; they aren't even the same kind of number.)

4. **Quantify the exposure.** Rebuild the Q4 2026 forecast twice: once exactly as the memo describes (churn = 1.5%, everything else unchanged), and once with churn held at the trailing-3-month **actual** rate from Task 2 instead, everything else identical. Report both Q4 ending MRR figures and the dollar gap between them, in both dollars and as a percentage of the memo's headline number.

5. **Propose the fix — as a corrected assumption, not just a lower number.** Write the one or two sentences you'd send back to the VP. State what churn rate you'd actually put in the model, tie it to a specific, defensible piece of evidence (not "it feels safer"), and say what would have to be true before you'd be comfortable lowering it toward 1.5% (for example: what fraction of total MRR would need to come from post-onboarding cohorts before their better retention meaningfully moves the *company-wide* blended rate?).

## What a strong submission looks like

- Task 1 names the exact assumption, not a vague "the forecast feels too optimistic."
- Task 2 shows real numbers pulled from the actual post-June data — not an assertion that the memo is wrong, but a computed contradiction.
- Task 3's explanation correctly distinguishes a **cohort-level, age-indexed retention curve** from a **company-level, calendar-indexed monthly churn rate** — conflating the two is exactly the error the memo makes, and naming that distinction precisely is the core skill this challenge is testing.
- Task 4 reports both dollar figures and states the gap as a percentage — a number like "~12%" lands harder in a memo than "meaningfully lower."
- Task 5's fix is falsifiable: it says what evidence would change your mind, not just what number you'd prefer.

## Submission

`challenge-02.md` with your reasoning for each task, plus the queries/code you ran, in your portfolio under `c38-week-11/challenge-02/`.
