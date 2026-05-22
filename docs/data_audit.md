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

---

# Week 2 — Advanced Customer Analytics

## Loyalty Validation Analysis

Two separate loyalty frameworks were validated using both Pearson and Spearman correlation analysis against estimated customer lifetime value (`customer_lifetime_value_proxy`).

### Correlation Results

| Metric               | loyalty_score_v1 | loyalty_score_v2 |
| -------------------- | ---------------- | ---------------- |
| Pearson Correlation  | 0.411            | 0.815            |
| Spearman Correlation | 0.446            | 0.784            |

### Interpretation

- `loyalty_score_v1` captures behavioral loyalty and demonstrates moderate correlation with customer lifetime value.
- `loyalty_score_v2` captures commercial loyalty and demonstrates strong correlation with long-term customer value.

This confirms that:

- behavioral loyalty and commercial value are related
- but not equivalent concepts

The dual-loyalty framework was intentionally preserved to support more realistic customer segmentation.

---

# Loyalty Segment Validation

Customers were segmented into three loyalty groups using quartile-based segmentation on `loyalty_score_v2`.

| Segment | Logic      |
| ------- | ---------- |
| Loyal   | Top 25%    |
| Growth  | Middle 50% |
| At-Risk | Bottom 25% |

### Segment Distribution

| Segment | Customer Count |
| ------- | -------------- |
| Loyal   | 975            |
| Growth  | 1950           |
| At-Risk | 975            |

---

# Chi-Square Validation

A chi-square test was performed between loyalty segments and promotional dependency behavior.

### Result

- Chi-Square Statistic: 75.11
- P-Value: 4.91e-17

### Interpretation

The extremely small p-value indicates that promotional dependency differs significantly across loyalty segments.

This statistically validates the segmentation framework and confirms that loyalty segmentation captures meaningful behavioral differences.

---

# Customer Segmentation Modeling

## Clustering Method

K-Means clustering was applied using the following normalized behavioral and commercial features:

- customer_lifetime_value_proxy
- customer_engagement_score
- retention_risk_score
- promo_dependency_score
- normalized_frequency_score

### Cluster Optimization

Silhouette analysis was performed for:

- K = 2 to K = 6

| K   | Silhouette Score |
| --- | ---------------- |
| 2   | 0.3909           |
| 3   | 0.3644           |
| 4   | 0.3893           |
| 5   | 0.3670           |
| 6   | 0.3533           |

Although K=2 produced the highest silhouette score, K=4 was selected because it provided significantly richer business interpretability with only a negligible reduction in clustering quality.

---

# Final Customer Segments

## 1. Loyal High-Value

Characteristics:

- high customer lifetime value
- low retention risk
- high behavioral loyalty
- no promotional dependency

Business Meaning:
Represents the brand’s healthiest and most sustainable customer base.

---

## 2. High-Value Promo Dependent

Characteristics:

- very high customer lifetime value
- high engagement
- complete promotional dependency
- moderate retention risk

Business Meaning:
Represents high-revenue customers whose purchasing behavior relies heavily on discounts and promotions.

---

## 3. At-Risk Promo Customers

Characteristics:

- low customer value
- high retention risk
- strong promotional dependency
- weak engagement

Business Meaning:
Represents low-quality, discount-driven customers with weak long-term retention potential.

---

## 4. Dormant Organic

Characteristics:

- low engagement
- low promotional dependency
- moderate retention risk
- stable but inactive purchasing behavior

Business Meaning:
Represents organically acquired customers with potential for reactivation and growth.

---

# Strategic Findings

## Key Observation 1

The analysis identified two fundamentally different types of high-value customers:

- organically loyal high-value customers
- promotion-dependent high-value customers

This indicates that high customer value does not necessarily imply healthy long-term loyalty.

---

## Key Observation 2

Average purchase amounts were relatively similar across all customer segments.

This suggests that long-term customer value is driven more strongly by:

- purchasing frequency
- engagement behavior
- retention quality
- promotional dependency

rather than by single-purchase spending differences.

---

## Key Observation 3

A substantial portion of commercially valuable customers remain highly promotion-dependent.

This suggests the business may currently be generating revenue growth through discount-driven purchasing behavior rather than through fully organic customer loyalty.

---

# Final Analytical Deliverables

The following analytical artifacts were generated:

| Artifact                    | Purpose                                       |
| --------------------------- | --------------------------------------------- |
| final_segmented_dataset.csv | customer-level dataset with segment labels    |
| segment_profiles.csv        | segment-level business intelligence table     |
| loyalty validation analysis | statistical validation of loyalty framework   |
| customer segmentation model | behavioral and commercial customer clustering |
| strategic segment insights  | business recommendation foundation            |
