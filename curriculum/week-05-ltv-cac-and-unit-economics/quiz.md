# Week 5 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 6. A mix of multiple-choice and short calculations — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** A customer's `churn_month` is `NULL` as of the data cutoff. This means:

- A) The customer will never churn.
- B) The customer has not churned **as of the cutoff** — their eventual outcome is still unknown.
- C) The customer's data is missing and should be excluded from all analysis.
- D) The customer churned on the exact cutoff date.

---

**Q2.** What does "right-censoring" mean in the context of this week's customer data?

- A) Customers on the right side of a chart are censored from the report.
- B) Some customers haven't been observed long enough to know their true final tenure yet.
- C) Only customers who signed up in the second half of the year are included.
- D) Customer names are redacted for privacy.

---

**Q3.** Lumen Metrics' gross margin is 80%. A customer paying $149/month has a contribution margin of:

- A) $149.00
- B) $119.20
- C) $29.80
- D) $80.00

---

**Q4.** Why does pooling retention by **age since signup** (rather than by calendar month) matter?

- A) It doesn't matter — both approaches give identical results.
- B) It lets you compare, e.g., "month 3 after signup" across cohorts that signed up in different calendar months, turning several thin cohorts into one usable curve.
- C) Calendar-month pooling is illegal under GAAP.
- D) Age-based pooling removes the need for a data cutoff.

---

**Q5.** In this week's data, `paid_search`'s pooled retention ticked slightly **up** from age 5 to age 6 (52.4% → 55.6%). The correct interpretation is:

- A) Customers un-churned, proving the product improved.
- B) This is likely sample-size noise at a thin age (n=18), not a real reversal in churn behavior.
- C) The retention query has a bug and must be rewritten.
- D) Age 6 customers are contractually locked in.

---

**Q6.** The reciprocal-formula LTV (`margin ÷ avg_monthly_churn`) assumes:

- A) Churn accelerates every month.
- B) A constant monthly churn rate ("hazard") that holds indefinitely.
- C) All customers churn in month 1.
- D) Revenue, not margin, should be used.

---

**Q7.** A revenue-based LTV came out to $2,500 for a company with a 75% gross margin. The margin-based LTV is approximately:

- A) $2,500 (no difference)
- B) $3,333
- C) $1,875
- D) $625

---

**Q8.** `organic_content`'s media-only CAC ($171.43) is dramatically lower than its fully-loaded CAC ($2,571.43). The gap is mostly explained by:

- A) A calculation error — they should be equal.
- B) The channel's allocated team cost (content writers, editor, tools), which media-only CAC excludes entirely.
- C) Refunds issued to organic customers.
- D) Currency conversion.

---

**Q9.** `paid_search`'s fully-loaded CAC was $2,000.00 in every single month of the year. The most likely reason is:

- A) The channel had zero customers some months.
- B) Both its monthly spend ($6,000) and its monthly new-customer count (3) were flat all year, so the ratio never moved.
- C) CAC is always constant by definition.
- D) The formula used only counts December.

---

**Q10.** "Blended CAC" and "trailing CAC" differ from each other because:

- A) They're always identical; the terms are interchangeable.
- B) Blended CAC averages the whole observed period; trailing CAC uses only the most recent period, which better reflects the current/marginal cost of the next customer.
- C) Trailing CAC only applies to `paid_search`.
- D) Blended CAC excludes `team_cost`.

---

**Q11.** Naive payback period assumes:

- A) The customer churns immediately.
- B) The customer keeps paying at 100% retention, forever, with no churn.
- C) CAC is always zero.
- D) Contribution margin equals revenue.

---

**Q12.** `paid_search`'s LTV ($1,342.73) is **less than** its CAC ($2,000.00). What does this say about its retention-adjusted payback period?

- A) It's exactly 16.78 months, same as the naive calculation.
- B) It's undefined — at current churn and CAC, the average customer's cumulative contribution margin never fully recovers what was spent to acquire them.
- C) It's negative.
- D) Payback periods don't apply to LTV values below CAC.

---

**Q13.** Using the standard SaaS heuristics from Lecture 2, an LTV:CAC ratio of **1.26** is best classified as:

- A) Healthy
- B) Marginal — profitable in theory, but thin, and below the 3:1 comfort threshold
- C) Unprofitable
- D) Not calculable

---

**Q14.** Lumen Metrics' total MRR grew every month in 2025 (from $546 to $5,108). Which statement is most accurate?

- A) This proves the company is unit-profitable on every channel.
- B) Revenue growth and unit profitability are separate claims; a company can grow its top line while one or more channels lose money on every customer acquired.
- C) MRR growth is irrelevant to a unit-economics analysis.
- D) 836% growth automatically implies a healthy LTV:CAC ratio.

---

**Q15.** A colleague computes "average customer tenure" as 4.3 months by averaging `months_active` across all 64 customers (including the ones with `NULL` `churn_month`, counted through the cutoff date). This estimate is:

- A) Unbiased and exactly correct.
- B) Biased **upward** — it overstates true average tenure.
- C) Biased **downward** — it understates true average tenure, because still-active customers haven't finished accumulating tenure yet.
- D) Meaningless because `NULL` values should have caused an error.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — `NULL` in `churn_month` means "hasn't churned as of the cutoff," not "will never churn." That distinction is right-censoring, and it's the core idea of the whole week.
2. **B** — right-censoring means some observations (recent signups) haven't been tracked long enough to know their true final outcome yet.
3. **B** — `$149 × 0.80 = $119.20`. Contribution margin, not revenue, is what LTV should be built on.
4. **B** — age-based pooling lets a January cohort's "month 3" and a September cohort's "month 3" combine into one statistically meaningful curve, instead of eleven separate thin cohorts.
5. **B** — by age 6, the pooled sample is only 18 customers; a couple of extra survivors shifts the percentage noticeably. Rule of thumb from Lecture 1: don't trust a retention point under ~15 customers.
6. **B** — the reciprocal formula (`margin ÷ churn_rate`) is a geometric series that only holds if the monthly hazard rate is constant forever — an assumption, not a fact.
7. **C** — revenue-based LTV overstates the margin-based figure by exactly `1 ÷ margin%`. Going the other way, margin-based = revenue-based × margin% = `$2,500 × 0.75 = $1,875`.
8. **B** — the fully-loaded CAC includes `team_cost` (the content team's salaries), which media-only CAC leaves out entirely — and for `organic_content`, that's the vast majority of the real cost.
9. **B** — flat $6,000/month spend divided by a flat 3 new customers/month gives the same $2,000.00 every single month; there's no variation to average away.
10. **B** — blended CAC is a full-period average (a report card); trailing CAC uses only the most recent period and better estimates what the *next* customer will cost.
11. **B** — naive payback assumes the customer never churns, paying the full margin every month indefinitely, which is why it understates how long payback really takes (or whether it happens at all).
12. **B** — if LTV (the full projected lifetime value) is already below CAC, then no finite amount of time makes cumulative contribution margin catch up — payback is structurally undefined at those numbers, not just "long."
13. **B** — marginal. The standard heuristics: ≥3:1 healthy, 1:1–3:1 marginal, <1:1 unprofitable. 1.26 sits in the marginal band.
14. **B** — revenue growth and unit profitability are independent claims; Lumen Metrics' own `paid_search` channel is a live example of a company growing while one channel is unprofitable.
15. **C** — biased downward (understated). Still-active customers' `months_active` is only their tenure-*so-far*, not their true final tenure — they're going to keep accumulating tenure after the cutoff, so including them at their current, incomplete count pulls the average down, not up.

</details>

**Scoring:** 13+ → start Week 6. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; LTV, CAC, and payback compound directly into every remaining week of this course.
