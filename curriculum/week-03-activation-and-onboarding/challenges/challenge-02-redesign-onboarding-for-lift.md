# Challenge 2 — Redesign Onboarding for Activation Lift

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer — but the math must be right.**

## The scenario

You've diagnosed the funnel (Lecture 3, Exercise 3): 209 users create a first card, but only 116 go on to invite a teammate — a 44.5% drop, the worst single step in onboarding. Your head of growth wants a concrete proposal by end of day: what would you actually *change* in the product, and how much activation and retention lift do you honestly expect from it?

This challenge has two parts: the **product proposal** (qualitative) and the **quantified projection** (SQL + arithmetic). Both are graded — a great idea with no numbers, or accurate numbers with no idea behind them, are both incomplete answers.

## Part 1 — The proposal

Propose **two or three concrete onboarding changes** aimed specifically at the `created_first_card → invited_teammate` step. For each, state:

- What changes in the product (be specific — "add a step" isn't specific, "after a user creates their first card, show a modal with a pre-filled invite link and the copy '[Board name] works better with a teammate — invite one now?' with a 'skip for now' option" is).
- Which of Lecture 1's four activation-event criteria it targets (usually: making the collaborative step *earlier*, more *obvious*, or lower-friction).
- One plausible risk or downside (e.g. an intrusive modal could increase early churn instead of invites — onboarding changes aren't free).

Ideas to react to or improve on (don't just copy these — sharpen or replace them):

- A blocking or semi-blocking "invite a teammate" prompt immediately after first-card creation, with a skip option.
- A two-sided incentive (e.g. both inviter and invitee unlock a feature) shown at the same seam.
- Restructuring the product default itself — e.g. a new board starts with a suggested "invite your team" card already in it, so the *first* card a user ever touches is the invite prompt, not a to-do item.

## Part 2 — The quantified projection

This is where you turn the proposal into a number, using only material from this week.

1. **Restate the baseline.** From Exercise 3: 209 users reach `created_first_card`; 116 convert to `invited_teammate` (55.5%). From Lecture 2: inviters retain at week-4 at 65.5%; non-inviters at 15.1% (a +50.4pp lift). The current overall week-4 retention rate is 119/400 = 29.75%.

2. **Pick a target conversion rate** for the card→invite step, post-redesign, and **justify it** in one sentence (e.g. "I'm assuming a well-placed, skippable prompt recovers roughly a third of the current 44.5% drop, landing conversion around 70%" — any reasoned number is acceptable; an unreasoned one isn't).

3. **Project the new inviter count**, using your target rate:
   `projected_new_inviters = target_rate × 209 (current created_first_card population)`
   Report the delta versus the current 116.

4. **Project the retention effect.** Assume the *newly* converted inviters get the same retention lift as today's inviters (65.5% vs. the 15.1% baseline they'd otherwise have gotten) — this is a simplifying assumption, and you must say so explicitly. Compute:
   `extra_week4_active = delta_inviters × (0.655 − 0.151)`
   Then the new total week-4-active count and the new overall week-4 retention rate.

5. **Run a 3-point sensitivity range** — conservative, base, and optimistic target conversion rates (pick your own three, or use 60% / 70% / 80% as a starting point) — and report all three projected overall retention rates in a small table.

6. **State the honest caveat.** In 2–3 sentences: what would have to be true for this projection to hold (the "same lift for new converts" assumption in particular), and what's the *only* way to actually confirm the number (tie back to Week 8 — this projection is the hypothesis an A/B test would go on to validate, not a promise).

## Deliverable

`redesign-memo.md` containing Parts 1 and 2 in full, plus `redesign-projection.sql` with the SQL you used to pull the baseline numbers in step 1 (you may hand-compute steps 2–5's arithmetic in the memo, but show your work).

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Proposal specificity | "Improve onboarding to encourage invites" | A named, concrete product change with copy/placement detail |
| Target rate justification | An unexplained number | A one-sentence, reasoned assumption for the target |
| Arithmetic | Missing or wrong | Follows steps 3–4 correctly, delta and new rate both shown |
| Sensitivity | Single point estimate only | A 3-point range (conservative/base/optimistic) |
| Honesty | Presents the projection as a guaranteed result | Explicitly labels it a projection resting on a stated assumption, and names the experiment that would confirm it |

## Submission

Commit `redesign-memo.md` and `redesign-projection.sql` to your portfolio under `c38-week-03/challenge-02/`.
