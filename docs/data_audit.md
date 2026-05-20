# Data Audit Report

## Dataset Information

- Dataset Name: Customer Shopping Trends Dataset
- Total Rows: 3900
- Total Columns: 18
- Duplicate Customer IDs: 0
- Duplicate Rows: 0
- Dataset Granularity: One row represents one customer

---

# Missing Value Analysis

| Column            | Missing Count | Missing % | Planned Action     |
| ----------------- | ------------- | --------- | ------------------ |
| Review Rating     | 37            | ~0.95%    | Median imputation  |
| All Other Columns | 0             | 0%        | No action required |

### Observation

The dataset is nearly complete with missing values only present in the `Review Rating` column. Since the missing percentage is below 1%, median imputation is appropriate and unlikely to distort the overall distribution.

---

# Duplicate Analysis

| Check                  | Result |
| ---------------------- | ------ |
| Duplicate Rows         | 0      |
| Duplicate Customer IDs | 0      |

### Observation

Each row represents a unique customer profile. The dataset is customer-level rather than transaction-level.

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

No major datatype mismatches were detected.

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

No major inconsistencies were found in categorical values. However, text standardization using lowercase conversion and whitespace stripping was still applied to ensure consistency for grouping and SQL analysis.

---

# Distribution & Outlier Analysis

## Purchase Amount (USD)

- Purchase amounts are distributed approximately between 20–100 USD.
- No strong skewness or extreme concentration was observed.
- Distribution appears relatively balanced.

## Previous Purchases

- Range: 1–50 purchases
- Serves as a strong behavioral proxy for customer retention and loyalty.

## Age

- Distribution is relatively uniform between 18–70.
- No dominant customer age cluster was identified.

### Planned Outlier Treatment

IQR-based outlier detection was applied on:

- Purchase Amount (USD)
- Previous Purchases

Outliers, if detected, were capped rather than removed to preserve customer-level information.

---

# Business-Relevant Initial Findings

1. Clothing is the dominant product category and may represent the primary acquisition category.

2. Geographic distribution is highly balanced across states, meaning opportunity analysis should focus on customer value and loyalty rather than customer count alone.

3. Frequency of Purchases is well distributed and serves as a strong behavioral signal for loyalty modeling.

4. Previous Purchases appears to be one of the strongest retention-related variables available in the dataset.

5. Since the dataset is customer-level rather than transactional, advanced behavioral metrics such as loyalty and promotional dependency must be inferred using proxy features.

---

# Promotional Dependency Observation

The columns:

- `Discount Applied`
- `Promo Code Used`

were found to be perfectly correlated across the dataset.

This means:

- every customer with a discount also used a promo code
- no intermediate promotional behavior exists

As a result:

- `Discount Applied` was retained as the primary feature for promotional dependency analysis
- `Promo Code Used` was retained only for traceability and downstream analysis

This causes promotional dependency to behave as a binary feature rather than a continuous behavioral measure.

---

# Dataset Limitations

The dataset does not contain:

- transaction-level purchase history
- timestamps
- sequential purchasing behavior
- actual churn labels

Therefore:

- retention behavior must be inferred indirectly
- customer lifetime value must be approximated using proxy metrics
- promotional dependency cannot be measured dynamically over time

---

# Planned Cleaning Steps

- Handle missing review ratings using median imputation
- Standardize categorical text columns
- Verify numeric datatypes
- Detect and cap outliers using IQR method
- Save cleaned dataset separately as:
  `cleaned_dataset.csv`

---

# Feature Engineering Strategy

The following business-focused features were engineered to construct customer intelligence from available variables:

| Feature                       | Purpose                                            |
| ----------------------------- | -------------------------------------------------- |
| frequency_score               | Convert purchase cadence into behavioral intensity |
| subscription_flag             | Capture subscription engagement behavior           |
| repeat_buyer_flag             | Identify repeat purchasing behavior                |
| promo_dependency_score        | Measure discount reliance                          |
| discount_sensitivity_segment  | Segment customers by promotional dependency        |
| avg_spend_per_visit           | Estimate spending efficiency                       |
| spending_velocity_score       | Measure spending intensity                         |
| customer_lifetime_value_proxy | Approximate long-term customer value               |
| value_tier                    | Segment customers by economic value                |
| customer_engagement_score     | Estimate overall engagement strength               |
| retention_risk_score          | Estimate retention vulnerability                   |
| loyalty_score_v1              | Behavior-based loyalty definition                  |
| loyalty_score_v2              | Revenue-based loyalty definition                   |

---

# Repeat Buyer Threshold Refinement

The initial repeat buyer definition:

- `Previous Purchases > 2`

produced a highly imbalanced feature with approximately 96% positive cases.

The threshold was refined to:

- `Previous Purchases > 6`

to improve customer differentiation and increase segmentation quality.

---

# Loyalty Score Design

Two separate loyalty definitions were intentionally created to distinguish between behavioral loyalty and commercial value.

## loyalty_score_v1

Behavior-focused loyalty based on:

- purchase frequency behavior
- low promotional dependency
- customer satisfaction
- engagement behavior

Purpose:
Estimate intrinsic customer loyalty independent of spending.

---

## loyalty_score_v2

Revenue-focused loyalty based on:

- estimated customer lifetime value
- engagement behavior
- subscription participation

Purpose:
Estimate commercially valuable customers with strong long-term revenue potential.

---

# Analytical Assumption

High customer value and high customer loyalty are not assumed to be equivalent.

The project intentionally separates:

- behavioral loyalty
- commercial value

to support more realistic customer segmentation and retention strategy development.
