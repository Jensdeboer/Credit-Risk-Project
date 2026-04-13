# Dataset Notes — UCI Credit Card Default

## Source

- **Name:** Default of Credit Card Clients Dataset
- **Repository:** UCI Machine Learning Repository
- **URL:** https://archive.ics.uci.edu/ml/datasets/default+of+credit+card+clients
- **Reference:** Yeh, I. C., & Lien, C. H. (2009). The comparisons of data mining techniques for the predictive accuracy of probability of default of credit card clients. *Expert Systems with Applications*, 36(2), 2473–2480.

---

## Overview

| Property | Value |
|----------|-------|
| Customers | 30,000 |
| Features | 25 |
| Time period | April 2005 – September 2005 (6 months) |
| Geography | Taiwan |
| Target | Whether customer defaults next month (October 2005) |

---

## Column Descriptions

### Demographic & Credit

| Column | Type | Description |
|--------|------|-------------|
| ID | Integer | Row identifier — dropped in preprocessing |
| LIMIT_BAL | Float | Credit limit (NT dollar) |
| SEX | Integer | 1 = male, 2 = female |
| EDUCATION | Integer | 1 = graduate school, 2 = university, 3 = high school, 0/4/5/6 = other (undocumented) |
| MARRIAGE | Integer | 1 = married, 2 = single, 3 = other, 0 = undocumented |
| AGE | Integer | Age in years |

### Payment Status (PAY_0 to PAY_6)

Monthly repayment status from September 2005 (PAY_0) back to April 2005 (PAY_6).

| Value | Meaning |
|-------|---------|
| -2 | No consumption or paid in full (undocumented in original paper) |
| -1 | Payment made on time |
| 0 | Use of revolving credit (minimum payment) |
| 1 | Payment delayed by 1 month |
| 2 | Payment delayed by 2 months |
| ... | ... |
| 8 | Payment delayed by 8 months |

> **Note:** PAY_1 does not exist in the dataset — the columns go PAY_0, PAY_2, PAY_3, ... This appears to be a labelling issue in the original data.

### Bill Amounts (BILL_AMT1 to BILL_AMT6)

Amount of bill statement (NT dollar) from September 2005 (BILL_AMT1) back to April 2005 (BILL_AMT6).  
**Can be negative** — a negative value means the customer overpaid their bill (credit balance).

### Payment Amounts (PAY_AMT1 to PAY_AMT6)

Amount of previous payment (NT dollar) from September 2005 (PAY_AMT1) back to April 2005 (PAY_AMT6).  
Always >= 0.

### Target

| Column | Description |
|--------|-------------|
| default.payment.next.month | 1 = customer defaulted in October 2005, 0 = did not default |

---

## Known Issues

| Issue | Detail | How handled |
|-------|--------|-------------|
| Undocumented EDUCATION values | Values 0, 4, 5, 6 not in official documentation | Recoded to 3 (other) |
| Undocumented MARRIAGE value | Value 0 not in official documentation | Recoded to 3 (other) |
| Missing PAY_1 column | Dataset jumps from PAY_0 to PAY_2 | Kept as-is — consistent with source |
| Undocumented PAY value -2 | Not defined in original paper | Treated as meaningful (no consumption / paid in full) |
| Negative BILL_AMT values | Occur when customer overpays (credit balance) | Handled with signed log transformation |

---

## Class Distribution

| Class | Count | Proportion |
|-------|-------|------------|
| No default (0) | 23,364 | 77.88% |
| Default (1) | 6,636 | 22.12% |

The dataset is moderately imbalanced. Accuracy is not a suitable evaluation metric.
