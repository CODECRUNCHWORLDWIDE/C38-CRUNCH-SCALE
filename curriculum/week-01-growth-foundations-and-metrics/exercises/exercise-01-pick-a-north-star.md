# Exercise 1 — Pick a North Star for Three Products

**Goal:** Apply Lecture 1's five-question test to products *other* than StreakLab, so the test becomes a reusable tool instead of a fact you memorized about one app.

**Estimated time:** 45 minutes. No database needed — this is written reasoning.

## Setup

Re-read Lecture 1, section 2 (the five-question test) and section 3 (StreakLab worked example) before starting. Keep the test table format handy — you'll build one for each product below.

Create a file `exercise-01.md`.

## The three products

For **each** product below:

1. Propose **two** candidate north-star metrics (do not reuse "total signups" or "revenue" as your *final* answer for more than one product — force yourself to think about what's actually different).
2. Run **both** candidates through the five-question test (a small table, same shape as Lecture 1's).
3. State your **chosen** north star in one precise sentence — precise enough that someone else could write the `WHERE`/`HAVING` clause from your sentence alone.
4. Name **one** input metric that would sit underneath your chosen north star, and which team would own moving it.

### Product A — a ride-hailing app (like Uber/Lyft)

Two-sided marketplace: riders request trips, drivers accept them. Revenue is a cut of each completed fare.

### Product B — a B2B project-management SaaS tool (like Asana/Linear)

Teams (not individuals) sign up. A single account can have 1 to 200 teammates. Priced per active seat per month.

### Product C — a free, ad-supported news app

No subscription tier at all — 100% of revenue is ad impressions. Content is articles and short videos; users scroll a feed.

## Constraints

- A north-star candidate must be something you could imagine as a **query result**, not an abstract value word like "delight" or "trust."
- At least one of your two candidates per product must **fail** at least one of the five questions — show you can recognize a bad candidate, not just confirm a good one.
- For Product C specifically: explain why "ad impressions served" is tempting and why it's dangerous as *the* north star, even though it's literally where the revenue comes from. (This is the closest analogue to the "MRR is not StreakLab's north star" argument from Lecture 1 — make the same kind of case in your own words.)

## Done when…

- [ ] `exercise-01.md` has all three products, each with two candidates tested against all five questions.
- [ ] Each product has one clearly stated final north star, in one precise sentence.
- [ ] Each product lists one input metric and an owning team.
- [ ] Product C's writeup explicitly argues why "ad impressions" is a poor north star despite being revenue-adjacent.

## Stretch

Pick a real product you personally use daily (a habit app, a game, a messaging app — anything). Write its likely north star from the outside, the way an analyst would have to before ever seeing internal data. State what you'd need access to in order to confirm your guess.

## Submission

Commit `exercise-01.md` to your portfolio under `c38-week-01/exercise-01/`.
