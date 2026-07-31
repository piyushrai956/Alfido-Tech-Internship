# Loan Approval Prediction

A supervised classification model predicting loan approval from borrower features, with a focus on preprocessing, class-imbalance handling, and business-oriented evaluation.

**Notebook:** [`Loan_Approval_Prediction.ipynb`](./Loan_Approval_Prediction.ipynb)

## Dataset

- Source: [Kaggle — bhanupratapbiswas/loan-approval-prediction-case-study](https://www.kaggle.com/datasets/bhanupratapbiswas/loan-approval-prediction-case-study)
- Size: 614 rows x 13 columns
- Target: `Loan_Status` (Y = approved, N = rejected)
- Class balance: 68.7% approved vs. 31.3% rejected — moderate imbalance

## Data Quality Notes

- Missing values in 7 columns: `Gender`, `Married`, `Dependents`, `Self_Employed`, `LoanAmount`, `Loan_Amount_Term`, `Credit_History` (50 missing — the most of any column).
- `Dependents` contains a non-numeric category (`'3+'`) cleaned to `3` before modeling.
- Income and loan amount are right-skewed with high-end outliers, which informed the choice to scale numeric features for the linear model.

## Methodology

1. **Preprocessing** — mode imputation for categoricals, median imputation for numerics (robust to income/loan-amount outliers), one-hot encoding for categorical variables, `StandardScaler` for numeric features.
2. **Class imbalance** — handled two ways for comparison: `class_weight='balanced'` on Logistic Regression, and SMOTE oversampling (fit on the training split only, to avoid test-set leakage) for the other models.
3. **Models compared** — Logistic Regression (class_weight), Logistic Regression (SMOTE), Decision Tree (SMOTE), Random Forest (SMOTE).
4. **Evaluation** — precision, recall, F1 (both classes), and ROC-AUC on a held-out 20% stratified test split; ROC curves and a confusion matrix for the best model; a precision-recall threshold sweep to select a deployment threshold.

## Model Comparison

| Model | Precision (approved) | Recall (approved) | F1 (approved) | Precision (rejected) | Recall (rejected) | F1 (rejected) | ROC-AUC |
|---|---|---|---|---|---|---|---|
| **Logistic Regression (class_weight)** | 0.862 | 0.882 | 0.872 | 0.722 | 0.684 | 0.703 | **0.853** |
| Logistic Regression (SMOTE) | 0.840 | 0.929 | 0.883 | 0.793 | 0.605 | 0.687 | 0.816 |
| Decision Tree (SMOTE) | 0.806 | 0.682 | 0.739 | 0.471 | 0.632 | 0.539 | 0.706 |
| Random Forest (SMOTE) | 0.852 | 0.882 | 0.867 | 0.714 | 0.658 | 0.685 | 0.783 |

**Best model: Logistic Regression (class_weight='balanced')** — highest ROC-AUC (0.853) among all four.

## Key Findings

- `Credit_History` is a dominant, explainable signal on its own: applicants **with** a credit history are approved **79.6%** of the time vs. only **7.9%** for applicants **without** one.
- In the Random Forest's full feature-importance ranking, `ApplicantIncome` (0.212), `Credit_History` (0.205) and `LoanAmount` (0.197) are closely bunched at the top — credit history is one of several comparably important drivers, not an outright dominant one, once income and loan amount are also in the model.
- `class_weight='balanced'` outperformed SMOTE on this dataset — likely because SMOTE's synthetic points are more prone to overfitting at only 614 rows.
- The F1-maximizing decision threshold on the held-out test set is **0.341**, notably below the default 0.5. At the default threshold, precision is 0.864 and recall is 0.894.

## Business Interpretation & Deployment Threshold

- A **false positive** (approving a loan that should be rejected) carries direct credit/default risk. A **false negative** (rejecting a loan that should be approved) carries opportunity cost but no direct financial loss.
- Because bad-approval downside is typically larger than good-rejection downside, a risk-conscious deployment should generally set the threshold **above**, not at, the F1-optimal point of 0.341 — the F1-optimal threshold is reported here as a statistical reference point, not a recommended cutoff.
- Given the small sample size (614 rows), validate any chosen threshold against more data and monitor real-world default rates before finalizing it for production.

## Visualizations

- EDA overview: loan status distribution, approval rate by credit history, income/loan amount by status, approval rate by property area, correlation heatmap (6-panel)
- ROC curves for all four models + confusion matrix for the best model
- Precision-recall vs. threshold curve with the F1-optimal threshold marked
- Feature importance — top 10 drivers of approval (Random Forest)

## Tech Stack

Python 3.x · Pandas / NumPy · scikit-learn · imbalanced-learn (SMOTE) · Matplotlib / Seaborn · Jupyter / Colab
