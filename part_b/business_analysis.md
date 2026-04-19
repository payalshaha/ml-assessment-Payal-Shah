
# Part B: Business Case Analysis

## Promotion Effectiveness at a Fashion Retail Chain

---

# B1. Problem Formulation 

## (a) Machine Learning Problem Formulation

### Target Variable

* **items_sold** (number of items sold per store per month)

---

### Candidate Input Features

#### Store Features

* Store size (square footage)
* Location type (urban / semi-urban / rural)
* Monthly footfall
* Competition density
* Customer demographics (age group, income level)

#### Promotion Features

* Promotion type

  * Flat Discount
  * BOGO (Buy-One-Get-One)
  * Free Gift with Purchase
  * Category-Specific Offer
  * Loyalty Points Bonus
* Promotion duration
* Discount percentage
* Marketing spend

#### Time / Calendar Features

* Month
* Season
* Weekend count
* Festival indicator
* Holiday indicator

#### Historical Performance Features

* Previous month sales
* Previous promotion performance
* Average sales per store

---

### Type of Machine Learning Problem

**Supervised Learning — Regression Problem**

---

### Justification

* The outcome (**items_sold**) is a **continuous numeric value**
* We want to **predict a quantity**
* Historical labeled data is available
* The goal is to **maximize sales volume**

Therefore, this is a:

**Supervised Regression Optimization Problem**

---

## (b) Why Items Sold Is Better Than Total Sales Revenue

Using **total sales revenue** can be misleading because revenue depends on:

* Product price
* Discount levels
* Product mix
* Inflation
* Premium vs budget items

A promotion may:

* Sell many items
* But generate lower revenue due to discounts

---

### Example

| Promotion     | Items Sold | Revenue |
| ------------- | ---------- | ------- |
| Flat Discount | 100        | ₹50,000 |
| BOGO          | 180        | ₹48,000 |

Revenue suggests **Flat Discount** is better.
But business objective is **moving inventory**, so **BOGO** is more effective.

---

### Broader Principle

**The target variable must align with the business objective.**

This illustrates a key real-world ML principle:

> Choose the metric that represents the decision you want to optimize.

---

## (c) Alternative to One Global Model

Instead of one global model:

**Use a Hierarchical / Segmented Modeling Strategy**

---

### Example Segments

* Urban stores
* Semi-urban stores
* Rural stores

OR

* High footfall stores
* Medium footfall stores
* Low footfall stores

---

### Why This Is Better

Customer behavior differs by location.

Example:

* Urban store → responds to Loyalty Points
* Rural store → responds to Flat Discount

A single global model:

* Averages behavior
* Reduces accuracy
* Ignores local patterns

Segmented models:

* Capture local differences
* Improve prediction accuracy
* Improve business decisions

---

# B2. Data and EDA Strategy 

## (a) Joining the Tables

### Raw Tables

1. Transactions
2. Store Attributes
3. Promotion Details
4. Calendar

---

### Step 1 — Join Tables

Join using keys:

* `transactions.store_id`
* `transactions.promo_id`
* `transactions.date`

---

### Join Structure

```
transactions
    JOIN store_attributes
        ON store_id

    JOIN promotion_details
        ON promo_id

    JOIN calendar
        ON date
```

---

### Grain of Final Dataset

**One row = One Store × One Month**

This is called:

**Store-Month Level Dataset**

---

### Required Aggregations

| Metric              | Aggregation |
| ------------------- | ----------- |
| Items sold          | SUM         |
| Revenue             | SUM         |
| Transactions        | COUNT       |
| Footfall            | SUM         |
| Average basket size | MEAN        |
| Promotion usage     | COUNT       |

---

### Example Final Dataset

| store_id | month | promotion | items_sold | footfall | weekend_days | festival_flag |
| -------- | ----- | --------- | ---------- | -------- | ------------ | ------------- |

---

## (b) EDA to Perform Before Modeling

### 1. Promotion Performance Comparison

**Chart:** Bar Chart
**Variables:** Promotion vs Items Sold

**Look for:**

* Best performing promotion
* Worst performing promotion
* Variation across promotions

**Impact:**

* Helps select useful features
* Identifies strong signals

---

### 2. Seasonality Analysis

**Chart:** Line Chart
**Variables:** Month vs Items Sold

**Look for:**

* Seasonal peaks
* Festival spikes
* December / holiday effects

**Impact:**

Create features:

* month
* season
* festival_flag

---

### 3. Store Location Performance

**Chart:** Box Plot
**Variables:** Location Type vs Items Sold

**Look for:**

* Differences between urban and rural stores
* Variability across locations

**Impact:**

Create feature:

* location_type

May justify:

* segmented models

---

### 4. Correlation Heatmap

**Chart:** Correlation Matrix

**Look for:**

* Strong predictors
* Redundant variables
* Multicollinearity

**Impact:**

* Feature selection
* Remove unnecessary variables
* Improve model stability

---

### 5. Promotion Effect by Store

**Chart:** Grouped Bar Chart
**Variables:** Store vs Promotion vs Sales

**Look for:**

* Different response patterns

**Impact:**

Supports:

* store-level modeling

---

## (c) Handling Promotion Imbalance (80% No Promotion)

### Problem

The model may:

* Learn mostly from non-promotion data
* Ignore promotion effects
* Underestimate promotion impact

This is called:

**Class Imbalance**

---

### Steps to Address It

1. Stratified sampling
2. Oversample promotion data
3. Undersample non-promotion data
4. Use weighting in the model
5. Create binary feature:

```
promotion_active (0 / 1)
```

---

# B3. Model Evaluation and Deployment

## (a) Train-Test Split Strategy

### Data Available

* 3 years monthly data
* 50 stores

Total:

```
36 months
```

---

### Correct Split

**Time-Based Split**

Example:

```
Training:
Year 1
Year 2

Testing:
Year 3
```

---

### Why Random Split Is Wrong

Random split:

* Mixes past and future data
* Causes data leakage
* Gives unrealistic accuracy

Real-world prediction:

> We predict future from past.

---

### Evaluation Metrics

#### 1. MAE — Mean Absolute Error

Measures:

Average prediction error.

**Interpretation:**

If:

```
MAE = 120 items
```

Then:

The model is off by **120 items on average**.

---

#### 2. RMSE — Root Mean Squared Error

Measures:

Penalty for large mistakes.

**Interpretation:**

Useful when:

Big prediction errors are costly.

---

#### 3. MAPE — Mean Absolute Percentage Error

Measures:

Percentage error.

**Interpretation:**

If:

```
MAPE = 8%
```

Then:

Prediction error is about **8%**.

---

### Best Business Metric

**MAPE**

Because:

* Easy to interpret
* Easy to communicate to managers

---

## (b) Explaining Different Recommendations Using Feature Importance

The model recommends:

* **December → Loyalty Points Bonus**
* **March → Flat Discount**

We use:

* Feature Importance
  OR
* SHAP values

---

### Investigation Steps

1. Extract feature importance
2. Compare December vs March inputs
3. Identify key drivers

---

### Example Explanation

**December**

Key factors:

* Festival season
* High footfall
* Loyal customers shopping

Important features:

* festival_flag
* footfall
* customer_loyalty

Therefore:

**Loyalty Points perform better**

---

**March**

Key factors:

* Low demand period
* Price-sensitive customers

Important features:

* season
* income_level
* price_sensitivity

Therefore:

**Flat Discount performs better**

---

### Communication to Marketing Team

Use:

**Feature Importance Bar Chart**

Explain:

> December demand is driven by loyalty and festivals.
> March demand is driven by price sensitivity.

---

## (c) End-to-End Deployment Process

### Step 1 — Save the Model

Use:

```
pickle
```

or

```
joblib
```

Example:

```
model.pkl
```

---

### Step 2 — Monthly Data Pipeline

Each month:

1. Collect new data
2. Join tables
3. Aggregate to store-month level
4. Apply preprocessing

   * scaling
   * encoding
5. Feed data into model

---

### Step 3 — Generate Recommendations

Model predicts:

Expected items sold for each promotion.

System selects:

**Best promotion**

---

### Example Output

| Store    | Month    | Recommended Promotion |
| -------- | -------- | --------------------- |
| Store 12 | December | Loyalty Points Bonus  |
| Store 12 | March    | Flat Discount         |

---

### Step 4 — Monitoring

Monitor:

**Prediction Performance**

Key checks:

1. Prediction error (MAE)
2. Sales trend changes
3. Data drift
4. Model drift

---

### Retraining Triggers

Retrain when:

* Error increases significantly
* New promotion introduced
* Customer behavior changes
* Seasonal shift occurs
* Every **6–12 months**
