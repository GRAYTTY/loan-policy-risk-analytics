# Loan Approval Policy & Borrower Risk Analytics

A data-driven lending decision project combining **policy evaluation, borrower segmentation, and default prediction**.

The project explores a core lending problem:

> How can a lender reduce default risk without unnecessarily sacrificing approval volume and business opportunities?

Rather than evaluating a model in isolation, the analysis connects statistical testing, customer segmentation, and predictive modelling to support more differentiated loan approval decisions.

---

## 1. Project Overview

The analysis uses approximately **50,000 historical LendingClub loans** containing borrower characteristics, loan information, and repayment outcomes.

The project follows three levels of decision analysis:

```text
Portfolio Level
Loan Policy Evaluation
        ↓
Segment Level
Borrower Risk Segmentation
        ↓
Individual Level
Default Prediction
        ↓
Business Recommendation
```

This structure moves from evaluating a single portfolio-wide rule toward more differentiated and individualised lending decisions.

---

## 2. Policy Evaluation

Two rule-based approval policies were simulated using **loan grade** and **Debt-to-Income Ratio (DTI)**.

### Policy A — Control

```text
Approve:
Grade ∈ {A, B, C, D}
AND
DTI ≤ 25%
```

### Policy B — Treatment

```text
Approve:
Grade ∈ {A, B, C}
AND
DTI ≤ 20%
```

Policy A prioritises approval volume, while Policy B applies stricter risk controls.

### Primary Metric

The main policy metric is:

```text
Default Rate among Approved Loans
=
Defaulted Approved Loans
────────────────────────
Total Approved Loans
```

Default rate was selected because it directly reflects portfolio credit quality.

However, it is evaluated together with **approval volume**, because reducing defaults by simply rejecting more borrowers may not produce a better business outcome.

---

## 3. Key Policy Results

| Metric | Policy A | Policy B |
|---|---:|---:|
| Loans Approved | 37,438 | 24,905 |
| Approval Rate | 74.90% | 49.80% |
| Default Rate | 13.51% | 11.33% |

Policy B reduced the approved-loan default rate by:

**2.18 percentage points**

A one-sided two-proportion z-test found the difference statistically significant:

```text
z ≈ 8.0
p < 0.001
```

However, the stricter policy also resulted in:

**12,533 fewer approved loans**

This reveals the main business trade-off:

```text
Lower Credit Risk
        ↕
Lower Approval Volume
& Potential Revenue Loss
```

The analysis therefore does not treat a lower default rate as automatically equivalent to a better business policy.

> **Metric improvement needs to be interpreted together with its business cost.**

---

## 4. Borrower Segmentation

Portfolio-level averages can hide important differences between borrowers.

K-means clustering was therefore used to identify borrower groups based on characteristics available at loan origination, including:

- Debt-to-Income Ratio
- Annual Income
- Revolving Credit Utilisation
- Recent Delinquencies
- Loan Amount
- Loan Term

The default outcome and LendingClub grade were excluded from clustering so that the segments could be identified independently from existing risk labels.

The Elbow Method and Silhouette Score were used to select **3 borrower segments**.

### Segment Profiles

| Segment | Profile | Default Rate |
|---|---|---:|
| Cluster 0 | High Income, High Loan Size | 15.4% |
| Cluster 1 | Financially Stretched | 18.3% |
| Cluster 2 | Conservative Borrowers | 12.3% |

The difference between the highest- and lowest-risk segments exceeds **6 percentage points**, showing that portfolio risk is not evenly distributed.

---

## 5. Policy Performance by Segment

Policy A and Policy B were then evaluated separately within each borrower segment.

Policy B reduced default rates across all three groups, but the largest improvement occurred among **Cluster 1 — Financially Stretched Borrowers**.

This suggests that a single approval threshold may be inefficient.

A more differentiated strategy could apply:

```text
Higher-risk segments
→ stricter approval criteria

Lower-risk segments
→ greater approval flexibility
```

This could preserve more low-risk lending opportunities while concentrating risk controls where they create the greatest value.

---

## 6. Default Prediction

Rule-based policies are transparent but cannot fully capture complex relationships between borrower characteristics.

A neural-network prototype was therefore developed to estimate default risk at the individual level.

Nine origination-time features were used, including:

- Loan Amount
- Loan Term
- Interest Rate
- DTI
- Annual Income
- Revolving Utilisation
- Recent Credit Inquiries
- Recent Delinquencies
- Employment Length

The dataset was split using stratified sampling:

```text
Training    70%
Validation  15%
Test        15%
```

Three neural-network architectures were compared.

| Model | Test AUC | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| Model 1 | 0.668 | 0.235 | 0.646 | 0.345 |
| Model 2 | 0.666 | 0.236 | 0.655 | 0.347 |
| Model 3 | **0.669** | 0.234 | **0.667** | **0.347** |

Model 3 used additional depth, Batch Normalisation and Dropout.

Greater model complexity produced only limited improvement, suggesting that the available origination features place a ceiling on predictive performance.

---

## 7. Metric Selection & Business Interpretation

Model selection was not based on AUC alone.

In lending, failing to identify a borrower who later defaults can carry significant financial cost.

Therefore, **Recall** and **False Negative Rate (FNR)** were considered alongside AUC.

Model 3 achieved the highest recall:

```text
Recall = 0.667
```

However, class weighting also made the model relatively conservative, causing predicted default rates to exceed actual default rates and potentially rejecting borrowers who would have repaid successfully.

This creates another business trade-off:

```text
Higher Recall
→ Fewer risky borrowers missed

but

More False Rejections
→ Potential loss of profitable customers
```

Threshold selection should therefore depend on the relative business cost of false approvals and false rejections rather than using `0.5` automatically.

---

## 8. Segment-Level Model Evaluation

Model performance was also evaluated separately across borrower segments.

The highest-risk segment — **Cluster 1** — was also the most difficult to predict accurately.

For Model 3:

```text
Cluster 0 AUC = 0.687
Cluster 1 AUC = 0.643
Cluster 2 AUC = 0.666
```

This suggests that borrowers with high DTI, lower income, and elevated utilisation may involve more complex risk relationships.

Instead of simply increasing model depth, potential next steps include:

- richer feature engineering
- segment-specific thresholds
- dedicated modelling for high-risk borrowers
- additional behavioural data

---

## 9. Key Business Insight

The central finding of the project is:

> **No single approval threshold works equally well for every borrower.**

The three analytical stages provide complementary information:

```text
Policy Evaluation
→ Does a stricter policy reduce portfolio risk?

Borrower Segmentation
→ Which customers benefit most from the policy change?

Default Prediction
→ Can risk be assessed at the individual level?
```

Together, they support a more differentiated lending strategy:

- maintain stricter rules for higher-risk borrower segments;
- preserve greater flexibility for lower-risk borrowers;
- use individual risk scores to complement rule-based decisions;
- monitor both risk metrics and approval / profitability metrics.

---

## 10. Methodological Limitation

The policy comparison is a **simulated policy evaluation using historical observational data**, not a real randomised A/B experiment.

Therefore, the results show how the policies would have behaved on observed historical loans, but they do not establish causal effects.

A production implementation should validate proposed policy changes through a controlled live experiment on new applications and track:

- approval rate
- default rate
- false negative rate
- profitability
- customer acceptance
- portfolio performance

---

## 11. Tech Stack

`Python` · `Pandas` · `NumPy` · `scikit-learn` · `PyTorch` · `K-Means` · `Statistical Testing` · `Deep Learning`

Key methods:

- Data Cleaning & Feature Engineering
- Mutual Information
- Policy Simulation
- Two-Proportion Z-Test
- K-Means Clustering
- Elbow Method
- Silhouette Analysis
- Neural Networks
- AUC / Precision / Recall / F1 / FNR
- Segment-Level Model Evaluation

---

## 12. Repository Structure

```text
loan-policy-risk-analytics/
├── README.md
├── loan_policy_analysis.ipynb
└── .gitignore
```

## Data

This project uses a LendingClub loan dataset provided for academic coursework.

The original dataset is not redistributed in this repository due to data access
and redistribution considerations.

The repository focuses on the analytical workflow, methodology, and results.

---

## 13. Project Takeaway

This project demonstrates that analytical models should not be evaluated independently from the decisions they support.

A lower default rate may come at the cost of losing viable borrowers.  
A higher recall may come at the cost of excessive false rejection.  
A model with greater complexity may not deliver meaningful business improvement.

The key challenge is therefore not simply:

> **"Which model has the best metric?"**

but:

> **"Which combination of policy, segmentation and prediction creates the most appropriate risk–business trade-off?"**
