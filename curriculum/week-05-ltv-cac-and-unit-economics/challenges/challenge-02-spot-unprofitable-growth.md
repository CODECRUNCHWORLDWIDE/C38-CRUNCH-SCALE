# Challenge 2 — Spot an Unprofitable Growth Story

**Time:** ~60 minutes. **Difficulty:** Medium–Hard. **No single right answer on the write-up, but the math is checkable.**

## The scenario

Lumen Metrics' VP of Growth sends this memo to the founders ahead of next quarter's budget planning. Every number in it is real — pulled from the same `customers` and `channel_costs` tables you've used all week. **Nothing is fabricated.** And the memo's conclusion is still wrong, because of *how* several of the true numbers were computed and combined.

> **Subject: Q1 budget ask — let's double down on growth**
>
> Team — the numbers this year speak for themselves. MRR grew from $546 in January to $5,108 in December — **836% growth in twelve months.** Our growth engine works.
>
> Breaking it down by channel:
>
> - **`paid_search` is remarkably efficient.** We spent $32,400 on Google/Meta ads this year and brought in 36 customers — that's just **$900 per customer acquired.** Our LTV on these customers, based on their $149/month subscription and typical retention, comes out to **$1,678.** That's a **1.9:1 LTV:CAC ratio** — solidly above the industry "healthy" threshold most VCs cite.
> - **`organic_content` is our cheapest channel by far.** Total spend on content tools and promotion was only $4,800 for the whole year against 28 new customers — **$171 per customer.** We should be shifting more of next quarter's budget here immediately.
> - **On average, customers stick around 4.3 months** across both channels. Even where CAC runs a bit higher than we'd like, we're recouping it fast.
>
> **Ask: increase total growth spend by 30% next quarter, split evenly between both channels, to keep the MRR line moving the way it has been.**
>
> — VP of Growth

## Your task

In `challenge-02.md`:

1. **Verify or refute each of the memo's four numbered claims** (MRR growth, `paid_search` efficiency + LTV:CAC, `organic_content` cost, average tenure) against the seed data. For each one: show the query or calculation that checks it, state whether the number itself is *accurate*, and — separately — state whether the *conclusion drawn from it* is sound.
2. **Recompute `paid_search`'s real LTV:CAC ratio** using a fully-loaded CAC and a margin-based LTV, and show how far it is from the memo's claimed 1.9:1.
3. **Recompute `organic_content`'s real cost per customer** fully loaded, and explain what the memo's $171 figure is actually measuring (it's not wrong — it's incomplete).
4. **Address the "4.3 months" claim directly.** Is that number biased, and if so, in which direction — does it make the real picture look better or worse than it is? Name the statistical reason (you learned the term in Lecture 1).
5. **Write your own one-paragraph recommendation** on the budget ask — approve as written, approve with changes, or reject — backed by the corrected numbers, not the memo's.

## Constraints

- Don't just say "this is wrong" — show the corrected number and the gap between it and the memo's claim, for every point you refute.
- At least one of the memo's four claims involves *two* separate errors stacked together. Find both.
- Your recommendation must directly engage with the memo's specific ask ("30% more spend, split evenly") — don't write a generic unit-economics essay instead.

## Hints

<details>
<summary>On the "$1,678 LTV, 1.9:1 ratio" claim</summary>

Two lectures gave you two different corrections that both apply here at once. Which LTV method does "$1,678" match from Lecture 1? Which CAC does "$900" match from Lecture 2? Recompute the ratio using the *correct* version of both, not just one.

</details>

<details>
<summary>On "4.3 months average tenure"</summary>

Nineteen of `paid_search`'s 36 customers and twenty-three of `organic_content`'s 28 customers have a `NULL` `churn_month` — they haven't finished being customers yet. An average that includes their (still-running) tenure as if it were their final tenure has a name. Which direction does it bias the number, and why does that matter for a claim about "recouping CAC fast"?

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Verification | Trusts the memo's numbers at face value | Independently recomputes every claim from the seed data |
| Error identification | Says "this seems off" | Names the specific mechanism (media-only CAC, revenue-based LTV, censoring bias) |
| The stacked error | Misses that one claim compounds two mistakes | Explicitly separates and corrects both |
| Recommendation | Generic "be careful with growth" | A specific approve/change/reject call tied to the corrected LTV:CAC and the memo's actual ask |

## Submission

Commit `challenge-02.md` to your portfolio under `c38-week-05/challenge-02/`.
