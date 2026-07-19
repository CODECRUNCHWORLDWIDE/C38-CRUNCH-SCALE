# Week 8 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 9. A mix of multiple-choice and short "what would this compute?" — the answer key explains the *why*, not just the letter.

---

**Q1.** Which of these is a properly falsifiable hypothesis, in the if/then/because form from Lecture 1?

- A) "Express Checkout will improve the user experience."
- B) "If we replace the 3-step checkout with Express Checkout, conversion will increase, because removing page transitions removes points where a buyer can bail."
- C) "Let's ship Express Checkout and see what happens."
- D) "Express Checkout is better than the old checkout."

---

**Q2.** Why should a test have exactly **one** primary metric, not several?

- A) SQL can only `GROUP BY` one column at a time
- B) Extra metrics each add their own chance of a false positive from noise alone, inflating the real error rate beyond the stated α
- C) Guardrail metrics are illegal under GDPR
- D) Dashboards can only display one number per test

---

**Q3.** LoopCart randomizes Express Checkout by **visitor**, sticky for the whole test window, rather than by individual page-view. Why?

- A) Visitor-level IDs are easier to generate than session IDs
- B) The unit of randomization must match the unit of analysis — a single visitor's purchase journey needs to see one consistent checkout, not a coin flip on every request
- C) PostgreSQL requires randomization to be visitor-scoped
- D) It has nothing to do with the metric — either unit works identically

---

**Q4.** In the sample-size formula, `α` (significance level) primarily controls:

- A) The risk of missing a real effect (a false negative)
- B) The risk of calling a nonexistent effect "significant" by chance (a false positive)
- C) How many visitors a test needs
- D) How long a test needs to run, regardless of traffic

---

**Q5.** LoopCart wants to detect a **smaller** MDE than originally planned (say, a 5% relative lift instead of 15%), keeping α and power fixed. What happens to the required sample size?

- A) It shrinks — smaller effects are easier to detect
- B) It grows, because a smaller true difference is harder to distinguish from noise, and the formula's denominator is the squared difference between the two rates
- C) It stays exactly the same — MDE doesn't affect sample size
- D) It becomes undefined

---

**Q6.** The pilot's z-test on conversion rate (42% vs. 64%, n=50 each) returned p ≈ 0.0275. The correct interpretation is:

- A) There's a 2.75% chance Express Checkout doesn't actually work
- B) If Express Checkout truly had zero effect, a gap this large or larger between two 50-person samples would occur by chance about 2.75% of the time
- C) Express Checkout is 2.75% better than the old checkout
- D) 97.25% of visitors prefer Express Checkout

---

**Q7.** The pilot's 95% confidence interval for the lift was roughly **[+2.9, +41.1] percentage points**. What does this tell you that the p-value alone does not?

- A) Nothing — a CI and a p-value always carry identical information
- B) The plausible range for the *true* effect size is wide — the true lift could be as small as +2.9 points, not necessarily the dramatic +22-point observed gap
- C) The test needs to be re-run because the interval is invalid
- D) The confidence level should have been 99%, not 95%

---

**Q8.** Why does the CI calculation use **unpooled** standard error (`p1(1-p1)/n1 + p2(1-p2)/n2`) while the significance test uses a **pooled** standard error?

- A) They're interchangeable — either works for either purpose
- B) The significance test assumes the null hypothesis (both rates equal) is true and pools accordingly; the CI is estimating the actual difference and shouldn't assume it's zero
- C) Pooled error is always larger, so it's used only when a bigger p-value is desired
- D) SQLite doesn't support pooled variance calculations

---

**Q9.** A team checks their test's p-value every morning for 6 days and stops the moment it first crosses p < 0.05. Under a true null effect (no real difference), the actual chance of at least one false "significant" reading across those 6 daily checks is closest to:

- A) 5% — checking more often doesn't change the per-check probability
- B) 12%
- C) 26.5% — computed as 1 − (1 − 0.05)⁶
- D) 100% — repeated checks always eventually cross 0.05

---

**Q10.** A test designed for a 50/50 split shows an SRM chi-square of **20.164** (well above the 3.841 critical value at α=0.05). What's the correct next step?

- A) Report the primary metric result anyway, noting the imbalance in a footnote
- B) Stop — do not trust the primary metric result until the assignment mechanism itself is diagnosed and fixed, since SRM means the two groups may no longer be comparable
- C) Reweight the smaller arm's rows to compensate and recompute
- D) Increase α to 0.10 to compensate for the imbalance

---

**Q11.** A test's observed split is **888 control / 112 treatment**. Is this necessarily a sample-ratio mismatch?

- A) Yes — any split other than 50/50 is always an SRM
- B) No — it depends on the *designed* ratio; if the test was deliberately built as a 90/10 split, this observed split is close to expected and healthy
- C) Yes, because 888 and 112 don't sum to a round number
- D) No — SRM checks only apply to tests with more than 2 variants

---

**Q12.** In the NimbleCart example, treatment beat control in **both** the mobile segment (85% vs. 80%) and the desktop segment (25% vs. 20%), but lost overall (31% vs. 74%). This is:

- A) A calculation error — the segments must be wrong if the aggregate disagrees
- B) Simpson's paradox — caused by control's traffic being mostly the easy-to-convert mobile segment while treatment's traffic was mostly the harder desktop segment
- C) Proof that treatment is actually worse and the segment numbers are misleading
- D) A sign that the primary metric should be changed to a segment-weighted average

---

**Q13.** Six metrics are checked in an uncorrected test, and one of them (not the pre-registered primary metric) comes back at p = 0.04. Applying a Bonferroni correction for 6 comparisons, what threshold would that p-value need to clear to be called significant?

- A) 0.05 — Bonferroni doesn't change the threshold
- B) 0.30
- C) approximately 0.0083 (0.05 ÷ 6) — and p = 0.04 does **not** clear it
- D) approximately 0.30 (0.05 × 6)

---

**Q14.** LoopCart's Express Checkout pilot showed AOV falling ~9.9% (a guardrail concern) while conversion rose sharply. Revenue per cart session, however, was **higher** for treatment. What's the right way to read this?

- A) The AOV drop should be ignored entirely since revenue per session went up
- B) The two numbers are consistent: the conversion gain more than offset the smaller average order, so overall revenue improved even though AOV alone looked concerning — both numbers are worth reporting together
- C) This is a contradiction that means the data must contain a bug
- D) AOV and revenue per session always move in the same direction, so one of the two figures must be miscalculated

---

**Q15.** A single-week pilot shows a strong lift for a redesigned page. Before trusting that the lift will hold at full rollout, which concern from Lecture 3 is most directly relevant?

- A) Multiple comparisons — the test only measured one metric
- B) SRM — a one-week test can't have a sample-ratio mismatch
- C) A novelty effect — some of the week-1 lift may be curiosity that fades, or a primacy effect where the true benefit is understated until users adjust; a single short window can't distinguish either from a durable effect
- D) Simpson's paradox — this only applies to tests with more than two variants

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — the if/then/because form names the mechanism and is falsifiable; A, C, and D are all unfalsifiable restatements of "we hope this works."
2. **B** — every additional metric tested is another independent roll of the dice at the stated α, and the *combined* false-positive rate across several untested-for-correction metrics is higher than any single α suggests.
3. **B** — the randomization unit must match the unit of analysis; a visitor whose journey spans multiple requests needs one consistent variant, or the "independent observations" assumption behind every later formula breaks.
4. **B** — α is specifically your tolerance for a false positive (Type I error); power (not α) governs the false-negative risk.
5. **B** — a smaller MDE means a smaller `(p₂ − p₁)²` in the denominator, which makes the whole fraction — and thus the required `n` — larger, not smaller.
6. **B** — a p-value is a statement about how surprising the data would be *under the null hypothesis*, not a probability statement about the hypothesis itself; A and C are the two most common p-value misreadings.
7. **B** — the CI expresses the plausible range for the true effect size, information a bare p-value (which only says "distinguishable from zero or not") doesn't carry.
8. **B** — the significance test's null hypothesis literally assumes the two rates are equal, so pooling is the mathematically consistent choice there; the CI is trying to estimate the (presumably nonzero) true difference, so it shouldn't build in that assumption.
9. **C** — `1 − (1 − 0.05)⁶ ≈ 0.265`, roughly 26.5%, far above the naive 5% a single check would suggest.
10. **B** — an SRM this severe means the two groups likely differ systematically for reasons unrelated to the treatment; the correct move is to diagnose and fix the assignment bug, not to salvage or reweight the tainted result.
11. **B** — SRM checks are always against the *designed* ratio; an 888/112 split is exactly on target for an intentional 90/10 design and is healthy.
12. **B** — Simpson's paradox, driven by an uneven distribution of a confounding variable (device) between the two arms — each arm's traffic mix, not the treatment itself, is what's driving the aggregate reversal.
13. **C** — Bonferroni divides α by the number of comparisons (0.05 ÷ 6 ≈ 0.0083); p = 0.04 is well above that stricter bar and would not be called significant after correction.
14. **B** — both figures are real and both are worth reporting; a guardrail dipping while overall value improves is a legitimate, coherent outcome, not a contradiction to explain away.
15. **C** — a novelty (or primacy) effect is exactly the risk a short single-window test can't rule out; neither SRM nor multiple comparisons nor Simpson's paradox is specifically about effects changing over the life of a test.

</details>

**Scoring:** 13+ → start Week 9. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; Week 9 (pricing) leans on being able to size and read a test correctly without re-deriving it from scratch.
