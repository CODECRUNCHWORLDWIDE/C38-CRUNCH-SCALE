# Week 10 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 11. A mix of multiple-choice and short "what would you do" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** A churn label needs precisely three things defined before you write a single feature query. Which of these is **not** one of them?

- A) What counts as "churned"
- B) The cutoff date ("as of when")
- C) The label window ("over what future period")
- D) The exact scikit-learn model class you'll use

---

**Q2.** You compute a feature `events_last_30d` using `WHERE event_ts >= cutoff - INTERVAL '30 days'` with **no upper bound** on `event_ts`. What's wrong with this query?

- A) Nothing — it correctly captures the last 30 days
- B) It will include events *after* the cutoff, leaking future information into a feature meant to describe "now"
- C) `INTERVAL` syntax doesn't work in SQLite
- D) It should use `<=` instead of `>=`

---

**Q3.** Why does `stratify=y` matter for this week's 236-row, 36.9%-positive dataset specifically?

- A) It's required syntax for `train_test_split` — it isn't optional
- B) Without it, a small dataset's train/test split can land a meaningfully different positive rate in each split by chance, making the evaluation noisy
- C) It speeds up model training
- D) It removes the need for a random seed

---

**Q4.** On this week's seed, logistic regression (ROC-AUC 0.669) beat the random forest (ROC-AUC 0.575). The right conclusion is:

- A) Random forests are worse than logistic regression in general
- B) On a small tabular dataset, added model complexity isn't automatically an improvement — validate both, ship the one that performs better
- C) The random forest's code has a bug
- D) ROC-AUC isn't a valid metric for random forests

---

**Q5.** A colleague reports "our churn model gets 78% accuracy" on a dataset with an 8% true churn rate. The single best follow-up question is:

- A) "What's the ROC-AUC and precision/recall — a model predicting 'never churns' would get ~92% accuracy here and be useless"
- B) "What programming language did you use?"
- C) "How many rows were in the training set?"
- D) There's no follow-up needed — 78% is clearly good

---

**Q6.** PR-AUC (average precision) matters more than ROC-AUC specifically when:

- A) The dataset is large
- B) The classes are severely imbalanced, and you care about precision holding up as recall increases
- C) You're using logistic regression instead of a random forest
- D) The label window is longer than 3 months

---

**Q7.** In this week's seed, `days_since_last_event` uses a sentinel value of `999` for two customers. What does that sentinel represent, and why is it not simply "wrong data"?

- A) A data-entry error that should be deleted
- B) Those customers signed up very recently and have no pre-cutoff activity at all — `999` flags "no activity on record," distinct from an actually-inactive-for-a-long-time customer
- C) `999` means the customer already churned
- D) It's a placeholder that should always be imputed to `0`

---

**Q8.** Why are risk-band thresholds (`0.60`/`0.35` in this week's materials) best described as:

- A) A pure statistical calculation with one correct answer
- B) A business/capacity decision — set where your intervention capacity runs out, not derived from a formula
- C) Fixed values every churn model should use
- D) Something the model chooses automatically during training

---

**Q9.** The next-best-action decision table in Lecture 2 is implemented as a simple lookup (risk band × value tier → action) rather than a second trained model. Why?

- A) A lookup table is always more accurate than a model
- B) It keeps the business logic legible and editable by a non-technical stakeholder, separate from the (necessarily opaque) statistical scoring step
- C) scikit-learn doesn't support multi-class classification
- D) Lookup tables train faster

---

**Q10.** Only 12 of this week's 70 high-risk customers are high-value; the rest are low-value. What's the direct implication for intervention design?

- A) All 70 should get a CSM call regardless of value
- B) Most of the "urgent-looking" risk is concentrated in accounts too cheap to justify human-touch intervention, which is exactly why an automated action exists for that cell
- C) The model must be wrong, since risk and value should correlate
- D) Low-value customers should be removed from the dataset entirely

---

**Q11.** Why is comparing "customers who opened the win-back email" against "customers who didn't" the wrong way to measure a lifecycle campaign's effect?

- A) Email open rates are unreliable data
- B) Customers who engage with an email are already more engaged in general and would likely churn less regardless of the email — the comparison confounds engagement with campaign effect
- C) It's not wrong — it's the standard method
- D) You need at least 1,000 customers for this comparison to be valid

---

**Q12.** The hash-based holdout split in Lecture 3 uses a fixed salt string and `MD5(user_id)`. What property does this design guarantee?

- A) Every customer gets a unique action
- B) The split is deterministic and reproducible — re-running the assignment produces the identical treatment/holdout groups every time, with no state to store or lose
- C) It guarantees exactly 50/50 regardless of segment size
- D) It encrypts the customer data

---

**Q13.** This week's win-back test showed a 13.8-percentage-point observed lift with **p = 0.293**. The correct interpretation is:

- A) The campaign definitely worked — 13.8 points is a big number
- B) The campaign definitely didn't work — p > 0.05 means no effect
- C) The result is directionally positive but not statistically distinguishable from chance at conventional significance — the test doesn't have enough customers to know yet
- D) p-values don't apply to churn campaigns

---

**Q14.** The power calculation found you'd need **≈205 customers per group** (≈410 total) to reliably detect the observed effect size, against a current segment of 58. What does this number tell you?

- A) The campaign should be canceled immediately
- B) A quantified, honest answer to "how much more data would it take to know for sure" — useful for deciding whether to wait, run continuously, or combine with other evidence
- C) The p-value calculation must be wrong
- D) 205 is the number of customers who will churn

---

**Q15.** Why does the mini-project's rubric explicitly require the model card to state at least one real limitation and one explicit non-use-case, rather than just listing strengths?

- A) It's a formality with no practical value
- B) A model card that only lists strengths gives false confidence to whoever acts on the score downstream — Week 10's own Scale-plan blind spot (zero high-risk flags among 33 Scale customers, 5 of whom actually churned) is exactly the kind of thing a strengths-only document would hide
- C) Limitations are required by scikit-learn's license
- D) It makes the model appear less accurate than it is, for marketing reasons

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **D** — the model class is an implementation choice, not part of the label's definition. What counts as churned, the cutoff, and the window are the three things that must be fixed first.
2. **B** — an unbounded upper filter pulls in post-cutoff activity, which in production you'll never have when you actually need the score. Always bound both ends: `event_ts >= cutoff - N days AND event_ts < cutoff`.
3. **B** — with 236 rows and a 36.9% base rate, an unstratified split can easily land 25% or 48% positive in the test set purely by chance, making metrics noisy for no real reason.
4. **B** — this week's actual result. Complexity is a hyperparameter to validate, not a default win; ship whichever model performs better on held-out data.
5. **A** — with an 8% true rate, predicting "never churns" gets ~92% accuracy while catching zero real churners. Accuracy alone is close to meaningless on imbalanced problems.
6. **B** — ROC-AUC can look reasonable even when precision collapses at higher recall on imbalanced data; PR-AUC is sensitive to exactly that failure mode.
7. **B** — a sentinel like 999 flags "no data," a real and different situation from "inactive for 999 days." Deleting or blindly imputing it would erase a distinct signal (a stalled/very-new onboarding).
8. **B** — thresholds encode how much intervention capacity you have and how much a false positive costs; there's no universal "correct" 0.60.
9. **B** — separating the (opaque) scoring step from the (legible) decisioning step lets a non-technical stakeholder audit and adjust the business logic without touching the model.
10. **B** — the crosstab shows most of the flagged risk sits in low-value accounts, which is precisely why the `winback_email_discount` action (automated, cheap) exists as a distinct cell from `csm_call` (human, expensive).
11. **B** — engagement with the campaign and the outcome it's supposed to cause are both driven by the same underlying trait (customer health), so comparing engaged-vs-not measures the wrong thing; only randomizing before anyone can self-select avoids this.
12. **B** — a deterministic hash of a fixed salt plus `user_id` reproduces the identical split on every run, with nothing to persist or accidentally re-randomize.
13. **C** — p = 0.293 means "not distinguishable from chance at this sample size," not "no effect" and not "proven effect." The honest read names the sample-size limitation explicitly.
14. **B** — a power calculation converts "I have a hunch this test is too small" into an exact, actionable number, which is far more useful than either false confidence or vague doubt.
15. **B** — this week's own model has a documented blind spot (the Scale-plan subgroup); a model card that hid it would actively mislead whoever routes high-value accounts based on the score.

</details>

**Scoring:** 12+ → start Week 11. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; next week's forecasting builds directly on this week's discipline around windows and honest evaluation.
