# Week 9 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 10. A mix of multiple-choice and short "what does this mean" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In the Van Westendorp PSM, which two curves are **decreasing** as price rises?

- A) Too Expensive and Getting Expensive
- B) Too Cheap and Bargain
- C) Too Cheap and Too Expensive
- D) Bargain and Getting Expensive

---

**Q2.** The **OPP (Optimal Price Point)** is defined as the intersection of:

- A) Bargain and Getting Expensive
- B) Too Cheap and Getting Expensive
- C) Bargain and Too Expensive
- D) Too Cheap and Too Expensive

---

**Q3.** ScopeIQ's blended (all-30-respondent) PSM said "raise the price to ~$60." Segmenting by `indie`/`team`/`agency` revealed that recommendation was:

- A) Correct for every segment equally
- B) Wrong for `indie` (already near their upper comfort zone at $39) and dramatically underpriced for `agency`
- C) Only useful for setting the `Scale` tier's price
- D) Impossible to check without a new survey

---

**Q4.** A candidate value metric is "good" when it:

- A) Is whatever is easiest to query from the existing database
- B) Correlates with value received, is visible/predictable to the customer, and scales with their growth
- C) Maximizes the number of paywalled features
- D) Is identical across every customer segment

---

**Q5.** ScopeIQ chose Monthly Tracked Users (MTU) over seat count as its value metric because:

- A) MTU is easier to bill for
- B) Seat count barely varies across ScopeIQ's customers, while MTU varies enormously and tracks real usage/value
- C) Competitors use MTU
- D) MTU is required by the Van Westendorp method

---

**Q6.** The tier-boundary "cliff" problem refers to:

- A) A customer's usage exceeding the top tier's cap entirely
- B) A small increase in usage crossing a hard cap and triggering the *full* jump to the next tier's price, disproportionate to the usage change
- C) A tier having too few features
- D) Setting a tier's price above the segment's `too_expensive` threshold

---

**Q7.** In Lecture 2's repackaging, 10 of 30 accounts (`indie`) saw their price **decrease** from $39 to $29. This is:

- A) A pricing mistake that should be corrected
- B) Irrelevant to the overall MRR outcome
- C) The correct outcome — flat pricing was overcharging `indie` relative to their WTP, and value-based packaging fixes both directions at once
- D) Proof that Van Westendorp doesn't work for this segment

---

**Q8.** In the fitted linear demand model `Q(p) = a + b·p`, the revenue-maximizing price `p*` is found by:

- A) Setting `p* = a`
- B) Setting `dR/dp = 0` and solving, giving `p* = −a / (2b)`
- C) Taking the average of all tested prices
- D) Setting `Q(p*) = 0`

---

**Q9.** `|E| > 1` (elastic demand) means that raising price in that range:

- A) Always increases revenue
- B) Has no effect on revenue
- C) Loses revenue, because quantity falls proportionally faster than price rises
- D) Only applies to existing customers, never new ones

---

**Q10.** ScopeIQ's `price_experiment` measured conversion elasticity for:

- A) Existing, paying customers deciding whether to accept a price increase
- B) Brand-new prospects seeing a price for the first time on the signup page
- C) `Scale`-tier accounts only
- D) The `Starter` tier only

---

**Q11.** Existing-customer churn elasticity to a price increase is typically **lower in magnitude** than new-visitor conversion elasticity, primarily because:

- A) Existing customers don't notice price changes
- B) Existing customers have already built switching costs, gotten realized value, and (with grandfathering) get advance notice — all factors a first-time prospect doesn't have
- C) Existing customers are contractually required to accept any price
- D) New visitors are less price-sensitive than existing customers

---

**Q12.** In the Growth-tier forecast (Lecture 3 §6), 90-day MRR went **up** while customer count went **down**. This happened because:

- A) The forecast contains an arithmetic error
- B) New signups were miscounted
- C) The revenue gained per retained/new customer at the higher price outweighed the revenue lost from customers who churned in response to the increase
- D) Grandfathering was not applied

---

**Q13.** Expansion revenue (e.g., `Starter` customers organically upgrading to `Growth`) should be:

- A) Credited entirely to the price-change forecast, since it happens on the same tier
- B) Ignored entirely, since it's unrelated to pricing
- C) Modeled separately from the price-change effect, since it would happen regardless of whether the price changed
- D) Subtracted from the price-change forecast

---

**Q14.** A retention discount offer (e.g., an annual-prepay rate below the old monthly price) during a price increase primarily works by:

- A) Increasing the new list price further
- B) Giving at-risk existing customers a lower-friction alternative to churning entirely, trading a lower price for reduced churn and a longer commitment
- C) Replacing the need for a grandfathering period
- D) Applying only to new signups, never existing customers

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — Too Cheap and Bargain both decrease as price rises (fewer people call a higher price "too cheap" or "a bargain" the higher it goes). Too Expensive and Getting Expensive both increase.
2. **D** — OPP = Too Cheap ∩ Too Expensive, the point of minimum combined resistance. (A) is IDP, (B) is PMC, (C) is PME.
3. **B** — the blended curve averaged three very different WTP worlds into one number that fit none of them; segmenting revealed `indie` was already near its ceiling at $39 while `agency` was priced so low it risked signaling low quality.
4. **B** — correlation with value, visibility/predictability to the customer, and scaling with growth are the three properties from Lecture 2 §1. Ease of querying and feature-paywall count are not relevant criteria.
5. **B** — ScopeIQ's seat count is roughly constant across all customer sizes (a handful of growth/product people regardless of company size), while MTU varies by nearly 500x across the customer base and tracks real usage.
6. **B** — the cliff is specifically about a *hard cap* causing a disproportionate price jump for a small usage change, not about exceeding the top tier or feature/price placement in general.
7. **C** — value-based packaging corrects mispricing in both directions simultaneously; `indie` was overpaying relative to WTP under the old flat price, so a price cut for that segment is the model working correctly, not a flaw.
8. **B** — `R(p) = a·p + b·p²`; setting the derivative `dR/dp = a + 2b·p` to zero and solving for `p` gives `p* = −a/(2b)`.
9. **C** — when `|E| > 1`, the percentage drop in quantity exceeds the percentage rise in price, so revenue (`price × quantity`) falls net.
10. **B** — `price_experiment` was run on the signup page against prospects who had never bought before; it says nothing directly about existing-customer reaction.
11. **B** — switching costs, sunk realized value, and advance notice from grandfathering are the three mechanisms Lecture 3 §3 names; existing customers absolutely do notice price changes, and neither option (A) nor (D) nor (C) is accurate.
12. **C** — this is the classic pricing-power signature: revenue per retained/new customer rose enough at the new price to outweigh the smaller customer base, so total MRR rose even as headcount fell.
13. **C** — expansion revenue (customers naturally growing into a higher tier) happens independent of any price change on that tier, so it must be isolated in its own scenario/line to avoid crediting the price change with revenue it didn't cause (or the reverse).
14. **B** — a retention discount gives an at-risk customer a middle option between "pay the full new price" and "churn entirely," typically in exchange for a longer commitment (e.g., annual instead of monthly) — it complements, rather than replaces, grandfathering.

</details>

**Scoring:** 12+ → start Week 10. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; this week's concepts (WTP, packaging, elasticity) compound directly into Week 10's experimentation methods.
