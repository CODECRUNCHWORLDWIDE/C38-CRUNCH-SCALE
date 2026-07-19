# Challenge 1 — Repackage for a Target Segment

**Estimated time:** 60 minutes.

## The situation

Two weeks after the Starter/Growth/Scale repackaging ships, four `Growth`-tier accounts — the ones nearest the top of the `Growth` MTU range — email support asking for **SSO/SAML login**. Under Lecture 2's feature map, SSO is `Scale`-only. Sales' first instinct is "tell them to upgrade to `Scale` at $179" — a **+$110/mo (+159%)** jump from their current $69. Before that email goes out, your job is to check whether that's a price a `team`-segment customer would actually accept, and if not, design something they would.

## Tasks

### 1. Identify the accounts (SQL)

Join `accounts_usage` to `wtp_survey` on `account_id = respondent_id`, and pull the four `team`-segment accounts with the highest `mtu` (closest to the `Growth` ceiling) along with their individual `too_expensive` threshold from the survey. *(Expected: accounts 17–20, MTU 19,200–29,500, `too_expensive` values 115, 122, 130, 140.)*

### 2. Test the "just upgrade them" option (SQL or by hand)

Compare the proposed `Scale` price ($179) against **every** `team`-segment respondent's `too_expensive` value in `wtp_survey` — not just the four requesters. *(Expected: $179 exceeds the `too_expensive` threshold of all 10 `team` respondents — the highest is $140. Not one surveyed `team` customer would accept $179; forcing the upgrade is not a real option, it's a churn trigger with a price tag.)*

### 3. Design an SSO add-on

Propose a flat-dollar add-on price on top of the existing $69 `Growth` price (not a new tier — an add-on the customer opts into). Justify it against two constraints:

- It should sit comfortably **below** the `team` segment's `too_expensive` values from Task 1/2 (leave real headroom, don't just barely clear the bar).
- It should be **meaningfully less** than the full `Growth`→`Scale` price gap ($110) — you're selling one feature, not the whole upgrade; pricing it near the full tier gap defeats the purpose of offering an add-on at all.

State your chosen price and the reasoning in one or two sentences, referencing the specific numbers from Task 1.

### 4. Model the revenue impact

Using your Task 3 price, compute the MRR impact of three scenarios for the full 10-account `Growth` tier:

- **(a) Give it away** — SSO added to `Growth` at no extra charge. MRR impact: $0, but note in one sentence what this does to the case for ever charging for SSO later.
- **(b) Your add-on** — the 4 requesting accounts adopt it at your Task 3 price. Assume some fraction of the remaining 6 `Growth` accounts adopt it speculatively too (state and justify your assumed fraction — even 0% is a valid, stated assumption if you argue why).
- **(c) Force the full `Scale` upgrade** — apply Task 2's finding to argue why this scenario's "MRR impact" number is misleading on its own (a customer who churns rather than upgrade contributes **negative** MRR, not the full new-tier price).

## Deliverable

A short memo (`repackage-memo.md`, 300–500 words) with:

- The Task 1 query and its output.
- The Task 2 comparison and your one-sentence verdict on forcing the `Scale` upgrade.
- Your chosen add-on price with justification (Task 3).
- The three-scenario MRR comparison (Task 4), with a clear recommendation for which scenario ScopeIQ should ship.

## Done when…

- [ ] Your add-on price is justified against a specific number from `wtp_survey`, not just "it felt reasonable."
- [ ] Your memo states, explicitly, why forcing a full `Scale` upgrade is not simply "more revenue if they take it" — the churn side of that trade is named, not ignored.
- [ ] Every assumption (the speculative-adoption fraction in Task 4b) is written down as an assumption, not presented as a measured fact.

## Submission

Commit `repackage-memo.md` to your portfolio under `c38-week-09/challenge-01/`.
