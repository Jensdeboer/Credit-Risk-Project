# Credit Risk Prediction — UCI Credit Card Default

## Overview

An end-to-end machine learning project that predicts whether a credit card customer will default on their payment next month. The project covers the full data science workflow: exploratory analysis, data preprocessing, model training, evaluation, and explainability using SHAP.

**Dataset:** UCI Credit Card Default Dataset — 30,000 customers, 25 features  
**Task:** Binary classification (default: yes/no)  
**Best model:** Tuned XGBoost — AUC-ROC: 0.7805

---

## Results

| Model | AUC-ROC | Recall | Precision | F1 |
|-------|---------|--------|-----------|-----|
| Logistic Regression (baseline) | 0.7459 | 0.615 | 0.437 | 0.511 |
| Random Forest | 0.7795 | 0.586 | 0.503 | 0.541 |
| XGBoost (untuned) | 0.7462 | 0.538 | 0.479 | 0.507 |
| **XGBoost (tuned)** | **0.7805** | **0.624** | **0.467** | **0.534** |

> Accuracy was not used as an evaluation metric — the dataset is imbalanced (77.88% no default / 22.12% default), making accuracy misleading.

---

## Project Structure

```
credit-risk-project/
├── data/
│   ├── raw/                    # Original UCI dataset
│   ├── interim/                # Cleaned + log-transformed (before encoding/scaling)
│   └── processed/              # Final train/test splits ready for modelling
├── models/
│   ├── scaler.pkl              # Fitted StandardScaler
│   └── xgb_best_model.pkl     # Tuned XGBoost model
├── notebooks/
│   ├── 01_Problem_understanding_and_EDA.ipynb
│   ├── 02_Data_cleaning_and_preprocessing.ipynb
│   ├── 03_Modeling_and_evaluation.ipynb
│   └── 04_Interpretation_and_conclusion.ipynb
├── references/
│   └── dataset_notes.md        # Dataset documentation and column descriptions
├── reports/
│   └── final_report.md         # Written summary of findings
├── requirements.txt
└── README.md
```

---

## Notebooks

| Notebook | Description |
|----------|-------------|
| `01_Problem_understanding_and_EDA.ipynb` | Problem definition, data loading, sanity checks, univariate/bivariate analysis, correlation analysis, outlier detection |
| `02_Data_cleaning_and_preprocessing.ipynb` | Drop ID, recode EDUCATION/MARRIAGE, log-transform BILL_AMT and PAY_AMT, one-hot encoding, stratified train/test split, StandardScaler |
| `03_Modeling_and_evaluation.ipynb` | Logistic Regression baseline, Random Forest, XGBoost, hyperparameter tuning with RandomizedSearchCV, error analysis, feature importances |
| `04_Interpretation_and_conclusion.ipynb` | SHAP global and individual explanations, business implications, limitations, improvement roadmap, final conclusions |

---

## Setup

**Install dependencies:**
```bash
pip install -r requirements.txt
```

**Run notebooks in order:**
```
01 → 02 → 03 → 04
```

Each notebook saves its outputs (processed data, models) so the next notebook can load them directly.

---

## Key Findings

- **Payment status is the strongest predictor.** PAY_0 (most recent payment status) alone drives 39% of model predictions. Customers delayed 2+ months have a 69%+ default rate.
- **Credit limit is a proxy for creditworthiness.** Lower credit limits correlate with higher default risk (r = −0.15).
- **Bill amounts matter less than payment behaviour.** BILL_AMT columns contribute minimally to predictions — it is not how much you owe but whether you are paying it.
- **Simple models are competitive.** Logistic Regression (AUC 0.7459) comes close to the tuned XGBoost (AUC 0.7805), suggesting the data is the bottleneck rather than model complexity.
- **Missed defaulters are hard to detect.** The 37.6% miss rate consists of customers who appeared financially stable right up until default — unpredictable from payment history alone.

---

## Dataset

**Source:** [UCI Machine Learning Repository — Default of Credit Card Clients](https://archive.ics.uci.edu/ml/datasets/default+of+credit+card+clients)  
**Reference:** Yeh, I. C., & Lien, C. H. (2009). The comparisons of data mining techniques for the predictive accuracy of probability of default of credit card clients. Expert Systems with Applications, 36(2), 2473-2480.
