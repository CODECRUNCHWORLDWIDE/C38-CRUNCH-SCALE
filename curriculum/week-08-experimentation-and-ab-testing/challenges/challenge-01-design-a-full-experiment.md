# Challenge 1 — Design a Full Experiment

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

LoopCart's product lead pitches this in a hallway conversation: *"Product pages should show 'Only 3 left in stock!' when inventory is low — urgency converts. Let's just ship it to everyone."*

That's a launch idea, not an experiment. Your job is to turn it into one, using every piece of machinery this week gave you — Lecture 1's brief structure, Lecture 2's sizing math, and at least one defense from Lecture 3 baked into the design itself, not bolted on after the fact.

## Your task

Produce `challenge-01.md` containing a complete experiment spec, in the same shape as Lecture 1, section 7's finished LoopCart brief, **plus** the sizing work behind it. Specifically:

1. **Hypothesis** — write it in the if/then/because form from Lecture 1, section 2. Name the actual mechanism you think urgency messaging would change (be specific: is it building genuine information, e.g. real scarcity, or manufacturing pressure regardless of true stock levels? Your hypothesis should say which, because it changes your guardrails).

2. **Primary metric** — exactly one, defined precisely enough (numerator, denominator, unit) that two different engineers would write the identical SQL query from your sentence alone.

3. **At least two guardrail metrics**, each with a pre-committed threshold. At least one of them must address a risk specific to *urgency* messaging that a generic checkout test wouldn't need to worry about (think about what happens after the sale, not just at the moment of it).

4. **Randomization unit**, with one sentence justifying the choice against the "unit of randomization must match unit of analysis" rule from Lecture 1, section 5. Product-page urgency messaging raises a wrinkle checkout tests don't: a single visitor might view *many* product pages, from *many* different sessions, over the test window. Address this directly — don't just pick "visitor" and move on.

5. **Sample size and duration** — state a baseline rate and an MDE (invent reasonable numbers and say why they're reasonable — e.g., "product pages get ~9,000 views/day, and a redesign this size should be expected to move conversion by at least 8% relative to be worth the engineering cost"), then run the actual sizing formula from Exercise 1. Show your work, not just a final number.

6. **One pre-launch pitfall (Lecture 1, §6) and one in-test trap (Lecture 3)** that this specific experiment is most at risk of, each with the concrete defense you're building into the design (not a generic "we'll be careful").

## Constraints

- Numbers must be **internally consistent** — if you claim 9,000 views/day and a 21-day test, your duration math in step 5 has to actually use those numbers, not silently reset to a different scenario.
- The urgency-specific guardrail in step 3 cannot be AOV or refund rate copied from the LoopCart checkout example — think about what's unique to inventory-scarcity messaging specifically (hint: what happens to trust, or to behavior, if the stated inventory count doesn't match reality, or if it's shown even when there's no real signal behind it?).
- Step 6 must name traps **specific to this test**, not a generic list — e.g., don't just say "we'll watch for peeking," say *why* this particular test (traffic pattern, seasonality, messaging fatigue) makes one trap more likely than another.

## Hints

<details>
<summary>On the randomization-unit wrinkle</summary>

If you randomize per page-view, the same visitor could see urgency messaging on one product and not on another in the same session — which contaminates any user-level outcome (like "did they buy anything this session") but might be *fine* for a page-level outcome (like "did they buy this specific product"). The fix isn't automatically "randomize by visitor instead" — that has its own cost (visitors bucketed for the whole test window may see stale inventory-count copy for weeks). State which outcome you're actually optimizing for, and let that decide the unit, the same way Lecture 1 §5 worked through LoopCart's checkout case.

</details>

<details>
<summary>On the urgency-specific guardrail</summary>

Real experimentation teams that have shipped scarcity messaging have run into exactly this: if "Only 3 left!" is shown from a stale inventory feed and a customer discovers 40 were actually in stock, that's a trust hit that shows up in support tickets, reviews, or repeat-purchase rate — weeks after the test window closes, not inside it. A guardrail that only looks *during* the test window can miss this category of harm entirely. Say so, and say what you'd do about it (e.g., a longer post-launch holdback, a manual accuracy audit of the inventory feed before ever launching this test).

</details>

## How success is judged

| Signal | Weak submission | Strong submission |
|--------|------------------|--------------------|
| Hypothesis | Restates the pitch ("urgency converts") | Names the specific mechanism and commits to real vs. manufactured scarcity |
| Primary metric | Vague ("purchases go up") | Precise numerator/denominator, unambiguous SQL-translatable |
| Guardrails | Copies AOV/refund rate from the lecture unchanged | Includes at least one guardrail genuinely specific to urgency messaging's failure modes |
| Randomization unit | Picks "visitor" with no justification | Directly addresses the product-page multi-session wrinkle and ties the choice to the primary metric's unit of analysis |
| Sizing | No real numbers, or formula misapplied | Correct application of Exercise 1's formula, with stated (defensible) baseline and MDE assumptions |
| Traps | Generic list of all 5 traps from Lecture 3 | Two specific traps, argued for *this* test, each with a concrete design-level defense |

## Submission

Commit `challenge-01.md` (spec + sizing work + reasoning) to your portfolio under `c38-week-08/challenge-01/`.
