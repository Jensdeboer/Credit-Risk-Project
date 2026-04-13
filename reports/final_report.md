# Final Report — Credit Risk Prediction

## 1. Problem Statement

Credit card companies continuously lend money and face the risk of customers failing to repay. The objective of this project was to build a machine learning model that predicts whether a customer will default on their credit card payment in the next month, based on 6 months of historical financial and behavioural data.

- **Input:** Customer demographic and payment history data
- **Output:** Binary classification — default (1) or no default (0)
- **Dataset:** UCI Credit Card Default — 30,000 customers, 25 features

---

## 2. Data Overview

| Property | Value |
|----------|-------|
| Rows | 30,000 |
| Features | 25 (24 after dropping ID) |
| Missing values | None |
| Duplicates | None |
| Target imbalance | 77.88% no default / 22.12% default |

**Feature groups:**
- **Demographic:** SEX, EDUCATION, MARRIAGE, AGE
- **Credit:** LIMIT_BAL (credit limit)
- **Payment status:** PAY_0, PAY_2–PAY_6 (monthly payment status, most recent to oldest)
- **Bill amounts:** BILL_AMT1–BILL_AMT6 (monthly statement balances)
- **Payment amounts:** PAY_AMT1–PAY_AMT6 (monthly payments made)

---

## 3. Key EDA Findings

- **Target imbalance:** 22.12% default rate — accuracy is not a valid metric
- **Strongest predictor:** PAY_0 (most recent payment status), correlation r = 0.32 with default
- **Strongest negative predictor:** LIMIT_BAL, r = −0.15
- **Multicollinearity:** BILL_AMT columns are highly correlated with each other (r > 0.9 between consecutive months) — logical, as balances carry over
- **Skewness:** BILL_AMT and PAY_AMT columns are heavily right-skewed, requiring log transformation
- **Undocumented categories:** EDUCATION (values 0, 4, 5, 6) and MARRIAGE (value 0) required recoding

---

## 4. Preprocessing Steps

| Step | Action | Reason |
|------|--------|--------|
| Drop ID | Removed row index | Not a predictive feature |
| Rename target | `default.payment.next.month` → `default` | Cleaner code |
| Recode EDUCATION | 0, 4, 5, 6 → 3 (other) | Remove undocumented categories |
| Recode MARRIAGE | 0 → 3 (other) | Remove undocumented category |
| Log transform BILL_AMT | Signed log1p | Handle skewness + negative overpayment values |
| Log transform PAY_AMT | log1p | Handle extreme skewness and zeros |
| One-hot encoding | SEX, EDUCATION, MARRIAGE (drop_first=True) | Proper categorical treatment |
| Stratified split | 80/20, stratify=y, random_state=0 | Preserve 22% default rate in both sets |
| StandardScaler | Fit on train only | Prevent data leakage |

---

## 5. Modelling

### Models trained:

| Model | Class imbalance handling |
|-------|--------------------------|
| Logistic Regression (baseline) | class_weight='balanced' |
| Random Forest | class_weight='balanced', max_depth=10, min_samples_leaf=10 |
| XGBoost | scale_pos_weight=3.52 |
| XGBoost (tuned) | scale_pos_weight + RandomizedSearchCV (50 iterations, 5-fold CV) |

### Best hyperparameters (Tuned XGBoost):

| Parameter | Value |
|-----------|-------|
| n_estimators | 300 |
| max_depth | 5 |
| learning_rate | 0.01 |
| subsample | 0.6 |
| colsample_bytree | 0.6 |
| min_child_weight | 1 |

---

## 6. Results

### Final model comparison (test set):

| Model | AUC-ROC | Recall | Precision | F1 |
|-------|---------|--------|-----------|-----|
| XGBoost (tuned) | **0.7805** | **0.624** | 0.467 | 0.534 |
| Random Forest | 0.7795 | 0.586 | **0.503** | **0.541** |
| XGBoost (untuned) | 0.7462 | 0.538 | 0.479 | 0.507 |
| Logistic Regression | 0.7459 | 0.615 | 0.437 | 0.511 |

### Error analysis:

- Total defaulters in test set: 1,327
- Correctly caught (TP): 828 (62.4%)
- Missed (FN): 499 (37.6%)
- Missed defaulters profile: lower PAY_0 (appeared on-time recently), higher PAY_AMT1 (larger payments), closer to average LIMIT_BAL — harder to distinguish from reliable customers

---

## 7. Interpretation (SHAP)

Top features by SHAP importance:

| Rank | Feature | Importance | Direction |
|------|---------|------------|-----------|
| 1 | PAY_0 | 38.6% | High PAY_0 → higher default risk |
| 2 | PAY_2 | 15.7% | High PAY_2 → higher default risk |
| 3 | PAY_3 | 5.6% | High PAY_3 → higher default risk |
| 4 | PAY_5 | 4.9% | High PAY_5 → higher default risk |
| 5 | PAY_4 | 4.4% | High PAY_4 → higher default risk |
| 6 | PAY_AMT3 | 3.0% | Higher payment → lower default risk |
| 7 | LIMIT_BAL | 2.7% | Higher limit → lower default risk |

---

## 8. Business Implications

1. **Early intervention trigger:** A customer delaying payment by 2+ months should immediately trigger outreach — default rate rises to 69%+ at this point.
2. **Threshold tuning:** The default 0.5 classification threshold should be lowered to ~0.35 for credit risk use cases to improve recall at an acceptable precision cost.
3. **Regulatory explainability:** SHAP waterfall plots provide per-customer explanations suitable for regulatory requirements in credit decision-making.
4. **Model monitoring:** The model should be retrained periodically as customer behaviour and economic conditions change.

---

## 9. Limitations

- Only 6 months of payment history — no long-term behavioural patterns
- No credit bureau data (external credit scores, total debt across lenders)
- No transactional data (spending categories, cash withdrawals)
- Precision of 0.47 means roughly half of flagged customers are false alarms
- 37.6% of defaulters are missed — sudden life events are not predictable from payment history

---

## 10. How to Improve

| Approach | Expected AUC gain |
|----------|------------------|
| Feature engineering (utilisation ratio, payment trend, pay ratio) | +0.02 to +0.04 |
| Model stacking (LR + RF + XGBoost meta-model) | +0.01 to +0.02 |
| Bayesian hyperparameter optimisation (optuna) | +0.00 to +0.01 |
| Threshold optimisation | No AUC gain — better recall/precision balance |

The estimated ceiling of this dataset with standard ML approaches is **AUC ~0.82–0.84**. Beyond that, external data (bureau, transactional) would be required.
