# Exercise 3 — Run the Capstone Experiment

**Goal:** Build `fct_experiment_results`, test `onboarding_checklist_v2` for statistical significance, and write a ship/hold/extend recommendation you can defend in one paragraph.

**Estimated time:** 90 minutes.

## Setup

Exercises 1–2 complete. Continuing in `solutions.sql`.

## Tasks

1. **Confirm random assignment didn't accidentally correlate with channel.** Before trusting any lift number, check that `control` and `treatment` aren't secretly different populations:

   ```sql
   SELECT ea.variant, u.signup_channel, COUNT(*)
   FROM stg_experiment_assignments ea
   JOIN dim_user u ON u.user_id = ea.user_id
   GROUP BY ea.variant, u.signup_channel
   ORDER BY ea.variant, u.signup_channel;
   ```

   Note the channel mix for each arm in a comment. *(This dataset's assignment wasn't blocked by channel, so don't expect a perfectly even split — if one arm is wildly skewed toward a single channel, say so; it would complicate the read, though for this dataset it doesn't reverse the conclusion.)*

2. **`fct_experiment_results`.** Build it exactly as in Lecture 2, Section 5. *(Expected: `control: n=12, activated=8, rate=66.7%`, `treatment: n=12, activated=11, rate=91.7%`.)*

3. **Significance test.** Run the two-proportion z-test from Lecture 2, Section 5 in Python. *(Expected: `z≈1.508`, `p≈0.132`.)* State, in your own words and without hedging into "it depends," whether this result clears the conventional α = 0.05 bar.

4. **Minimum detectable sample size.** This dataset ran with 12 per arm. Using the standard two-proportion power formula (or an online/library power calculator — cite which one you used), estimate roughly how many users per arm you'd need to detect a 25-point lift (66.7% → 91.7%) at 80% power and α = 0.05. Compare that number to the 12 you actually had. *(You don't need to derive the power formula by hand — using `statsmodels.stats.power.NormalIndPower` or an equivalent tool and reporting the number, with the tool named, is the expected deliverable.)*

5. **Downside-risk check.** The checklist is a UI change, not a pricing or infrastructure change. In 2–3 sentences, argue whether that changes how much statistical certainty you'd want before a partial rollout (say, to 50% of new signups) versus before rolling it out to 100% and retiring the control experience entirely.

6. **Write the recommendation.** One paragraph, following the four-part shape from Lecture 3, Section 4 (ask → evidence → uncertainty → checkpoint), specifically about `onboarding_checklist_v2`. It must name the lift, the p-value, the sample size, and a concrete next step (a target sample size or a target date to re-check).

## Done when…

- [ ] `fct_experiment_results` exists with the exact numbers above.
- [ ] You've stated plainly whether p≈0.132 clears α=0.05 — no hedging.
- [ ] Task 4's sample-size estimate is a specific number with a tool named, not a vibe.
- [ ] Your Task 6 recommendation is a single paragraph, names all four required facts, and ends with a concrete next checkpoint (not "let's monitor it").

## Stretch

- Segment the experiment result by channel (`paid_search` vs. `organic` vs. `referral` within the May–June cohorts). With only 24 total users this will be extremely thin per cell — compute it anyway, then write one sentence explaining why you would **not** lead a recommendation with a channel-segmented result at this sample size, even if one channel's cell looks dramatic.
- If you have access to a slightly larger simulated dataset (ask an instructor, or generate one yourself with a fixed random seed matching this week's generation approach), re-run the same test at n=48 or n=96 per arm and see whether the same underlying 25-point-style lift becomes significant. This is the fastest way to build real intuition for why sample size is the lever, not the lift itself.

## Submission

Commit `solutions.sql` and a short `experiment-writeup.md` (Task 6's paragraph plus supporting numbers) to your portfolio under `c38-week-12/exercise-03/`.
