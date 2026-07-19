# Exercise 1 — Define an Activation Event

**Goal:** Measure unbounded and windowed reach for every candidate event, feel the effect of window leakage first-hand, and write a one-page justified definition of Crunch Boards' activation event.

**Estimated time:** 60 minutes.

## Setup

Connected to `crunch_scale` (or `crunch_scale.db`), 400 rows in `users`, 5017 in `events`. New `solutions.sql` under `-- Task N` comments, plus a written file `activation-event.md`.

## Tasks

1. **Unbounded reach.** For each of the six candidate events (`verified_email`, `created_first_board`, `created_first_card`, `invited_teammate`, `completed_first_task`, `connected_integration`), count distinct users who ever fired it, and the percentage of all 400 signups. *(Expected: 322/80.5%, 256/64.0%, 209/52.2%, 116/29.0%, 101/25.2%, 37/9.2%.)*

2. **7-day windowed reach.** Repeat Task 1, but only count a user as having "reached" the event if it happened within 7 days of their `signup_at`. *(Expected: verified_email and created_first_board and created_first_card are unchanged — nearly everyone who ever does these does them fast. `invited_teammate` drops to 107/26.8%. `completed_first_task` drops sharply to 49/12.2%. `connected_integration` drops to 9/2.2%.)*

3. **Explain the gap.** In one to two sentences per event, explain *why* `completed_first_task` and `connected_integration` lose so much more reach under the 7-day window than `invited_teammate` does. *(Hint: think about where each event sits in the sequence, and re-read Lecture 2 §4 Trap 3.)*

4. **24-hour reach for the leading candidate.** Compute the 24-hour windowed reach for `invited_teammate` alone. *(Expected: 2 users, 0.5%.)* What does this number tell you about whether a "day-1 activation" framing would even be meaningful for this event?

5. **Score each candidate against the four criteria.** Build a small table (in `activation-event.md`, not SQL) with columns: candidate | specific & observable? | early (within ~7 days for most who'll ever do it)? | reach (use Task 1's number) | plausibly product-value (one clause). Fill it in for all six candidates using Lecture 1's criteria.

6. **Write the definition.** In `activation-event.md`, write your final activation-event definition for Crunch Boards in this exact shape: *"A user is activated when they `<event>` within `<window>` of signup."* Justify the choice in 3–5 sentences, citing at least one number from Tasks 1–2 and explicitly naming the candidate you rejected and why.

## Done when…

- [ ] All 6 unbounded and 6 windowed reach numbers in `solutions.sql`, matching the "Expected" hints.
- [ ] Task 3's explanation correctly identifies that later-sequence events lose more reach under a short window because they take longer, on average, to reach at all.
- [ ] `activation-event.md` contains the criteria table (Task 5) and a one-sentence definition with a stated window (Task 6).
- [ ] The written definition names the rejected alternative and the specific reason (reach, lateness, or reverse causation).

## Stretch

- Recompute Task 2 with a 3-day window instead of 7. How much does `invited_teammate`'s reach drop? Is 7 days still the right window, or would you argue for something shorter/longer?
- For users who reached `invited_teammate` but *not* within 7 days (116 − 107 = 9 users), look up their actual TTV in hours. Are they mostly just-over-the-line stragglers, or clearly a different pattern? (Preview of Exercise 2.)

## Submission

Commit `solutions.sql` and `activation-event.md` to your portfolio under `c38-week-03/exercise-01/`.
