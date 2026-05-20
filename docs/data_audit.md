# Data Audit Report

## Dataset Information

- Dataset Name: Dataset
- Total Rows: 3900
- Total Columns: 18
- Duplicate Customer IDs: 0
- Duplicate Rows: 0

---

# Missing Value Analysis

| Column            | Missing Count | Missing % | Planned Action     |
| ----------------- | ------------- | --------- | ------------------ |
| Review Rating     | 37            | ~0.95%    | Median imputation  |
| All Other Columns | 0             | 0%        | No action required |

### Observation

The dataset is almost fully complete with missing values only in 'Review Rating' coulmn. The missing value percentage is less than 1% (0.95%), so medium imputation is appropriate and will not cause distortion in distribution.

---

# Duplicate Analysis

| Check                  | Result |
| ---------------------- | ------ |
| Duplicate Rows         | 0      |
| Duplicate Customer IDs | 0      |

### Observation

Each row represents a unique customer.

---

# Data Type Review

| Column                | Current Type | Expected Type | Status  |
| --------------------- | ------------ | ------------- | ------- |
| Customer ID           | int          | int           | Correct |
| Age                   | int          | int           | Correct |
| Purchase Amount (USD) | int          | int           | Correct |
| Review Rating         | float        | float         | Correct |
| Previous Purchases    | int          | int           | Correct |

### Observation

No datatype mismatches.

---

# Categorical Data Review

Columns Reviewed:

- Gender
- Category
- Location
- Season
- Payment Method
- Shipping Type
- Frequency of Purchases

### Observation

No major inconsistencies were found.

---

# Distribution & Outlier Analysis

## Purchase Amount (USD)

- Range appears evenly distributed between approximately 20–100 USD.
- No extreme skewness observed.

## Previous Purchases

- Range: 1–50 purchases
- Serves as a strong base for retention and loyalty behavior.

## Age

- Distribution is relatively uniform between 18–70.
- No dominant customer age cluster detected.

### Planned Outlier Treatment

IQR-based outlier detection method on:

- Purchase Amount (USD)
- Previous Purchases

If outliers are detected then they will be capped rather than removed to preserve customer information.

---

# Business-Relevant Initial Findings

1. Clothing is the dominant product category and may represent the primary acquisition category.

2. Geographic distribution is highly balanced across states, meaning opportunity analysis should focus on spend and loyalty rather than customer volume alone.

3. Frequency of Purchases is well distributed and will likely become one of the strongest behavioral features for loyalty modeling.

4. Previous Purchases appears to be the strongest available retention base in the dataset.

5. Since the dataset is customer-level rather than transactional, advanced metrics such as loyalty and promo dependency must be inferred from available variables.

---

# Planned Cleaning Steps

- Handle missing review ratings using median imputation
- Standardize categorical text columns
- Verify numeric datatypes
- Detect and cap outliers using IQR method
- Save cleaned dataset separately as:
  `cleaned_dataset.csv`

---

# Planned Feature Engineering

The following business-focused features will be created:

| Feature                | Purpose                                    |
| ---------------------- | ------------------------------------------ |
| promo_dependency_score | Measure discount reliance                  |
| value_tier             | Segment customers by revenue potential     |
| repeat_buyer_flag      | Identify repeat purchase behavior          |
| satisfaction_flag      | Capture customer satisfaction              |
| avg_spend_per_visit    | Estimate economic value per purchase cycle |
| loyalty_score_v1       | Behavior-based loyalty                     |
| loyalty_score_v2       | Revenue-based loyalty                      |
