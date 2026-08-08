# customerpulse1-consumer-behavior-analysis
CustomerPulse is an interactive Consumer Behavior Analytics dashboard designed to uncover customer purchasing patterns, subscription trends, category-wise revenue contribution, and demographic insights. The dashboard transforms customer data into actionable business insights to support data-driven decisions around customer segmentation

# 📊 CustomerPulse: Consumer Behavior Intelligence Dashboard

> **Turning 3,900 customer transactions into actionable business insights using Python, PostgreSQL & Power BI.**

## 🚀 Project Overview

CustomerPulse is an end-to-end **Consumer Behavior Analytics** project designed to understand how customers spend, what products they prefer, how discounts influence purchasing, and how subscription behavior varies across customer segments.

The project combines **Python for data cleaning & feature engineering, PostgreSQL for business analysis, and Power BI for interactive visualization** to transform raw transactional data into decision-ready insights.

### 🎯 Business Objective

The analysis focuses on answering key business questions around:

- Customer spending patterns
- Product and category performance
- Subscription behavior
- Discount dependency
- Customer loyalty and segmentation
- Age-group revenue contribution
- Shipping preferences
- Product ratings and purchasing behavior

---

## 🛠️ Tech Stack

| Technology | Purpose |
|------------|---------|
| 🐍 **Python (Pandas)** | Data cleaning, EDA & feature engineering |
| 🗄️ **PostgreSQL** | Business analysis & SQL queries |
| 📊 **Power BI** | Interactive dashboard & visualization |
| 📓 **Jupyter Notebook** | Exploratory data analysis |
| 🔗 **GitHub** | Project documentation & portfolio |

---

## 🔄 Project Workflow

**Raw Customer Data**  
⬇  
**Python Data Cleaning & EDA**  
⬇  
**Feature Engineering**  
⬇  
**PostgreSQL Database Integration**  
⬇  
**SQL Business Analysis**  
⬇  
**Power BI Dashboard**  
⬇  
**Business Insights & Recommendations**

---

## 🧹 Data Preparation with Python

The dataset contains **3,900 customer transactions and 18 columns**, covering customer demographics, purchasing behavior, product information, discounts, subscriptions, ratings and shipping preferences. 

Key data preparation steps:

- Loaded and explored the dataset using Pandas
- Performed structural and statistical analysis
- Handled missing Review Rating values using category-level median imputation

  ```python
  df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(lambda x: x.fillna(x.median()))
  ```
  
- Standardized column names using `snake_case`

```python
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(' ','_')
df = df.rename(columns={'purchase_amount_(usd)':'purchase_amount'})
```
  
- Created meaningful customer **Age Groups**

```python
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels = labels)
```

- Engineered **Purchase Frequency (Days)**

```python
frequency_mapping = {
    'Fortnightly': 14,
    'Weekly': 7,
    'Monthly': 30,
    'Quarterly': 90,
    'Bi-Weekly': 14,
    'Annually': 365,
    'Every 3 Months': 90
}

df['purchase_frequency_days'] = df['frequency_of_purchases'].map(frequency_mapping)
```

- Removed redundant `promo_code_used` information
- Connected Python with PostgreSQL
- Loaded the cleaned dataset into PostgreSQL for SQL analysis


---

## 🧠 SQL Business Analysis

PostgreSQL was used to answer practical business questions rather than performing only basic aggregations.

### Key analyses include:

1. **Revenue by Gender**
   - Compared revenue contribution across male and female customers.
  ```sql
select gender,sum(purchase_amount) as total_revenue
from customer1 
group by gender
 ```

2. **High-Spending Discount Users**
   - Identified customers who used discounts but still spent above the average purchase amount.
```sql
select customer_id,purchase_amount
from customer1
where discount_applied='Yes' and purchase_amount>=(select avg(purchase_amount) from customer1)
```

3. **Top-Rated Products**
   - Identified the top 5 products based on average customer ratings.
```sql
select item_purchased,round(avg(review_rating::numeric),2) as avg_product_rating
from customer1
group by item_purchased
order by avg_product_rating desc
limit 5
```
     

4. **Shipping Type Analysis**
   - Compared average purchase amounts between Standard and Express shipping.
```sql
select shipping_type,avg(purchase_amount) as avg_purchase_amt
from customer1
where shipping_type in ('Standard','Express')
group by shipping_type
```


5. **Subscribers vs Non-Subscribers**
   - Compared spending and revenue contribution across subscription groups.
```sql
select subscription_status,count(customer_id),round(avg(purchase_amount),2) as avg_spend,sum(purchase_amount) as total_revenue
from customer1
group by subscription_status
order by avg_spend desc,total_revenue desc
limit 10
```

6. **Discount-Dependent Products**
   - Identified products with the highest percentage of discounted purchases.
```sql
select item_purchased,round(100*sum(case when discount_applied='Yes' then 1 else 0 end)/count(*),2) as discount_rate
from customer1
group by item_purchased
order by discount_rate desc
limit 5
```

7. **Customer Segmentation**
   - Classified customers into **New, Returning and Loyal** segments using purchase history.
```sql
with customer_type as (
SELECT customer_id, previous_purchases,
CASE 
    WHEN previous_purchases = 1 THEN 'New'
    WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
    ELSE 'Loyal'
    END AS customer_segment
FROM customer1)

select customer_segment,count(*) AS "Number of Customers" 
from customer_type 
group by customer_segment;
```


8. **Top Products by Category**
   - Identified the top 3 most-purchased products within each category.
```sql
with item_counts as (
select category,item_purchased,
count(customer_id) as total_orders,
row_number() over(partition by category order by count(customer_id) desc) as item_rnk
from customer1
group by 1,2
)
select item_rnk,category,item_purchased,total_orders
from item_counts
where item_rnk<=3
```

9. **Repeat Buyers & Subscription Behavior**
   - Investigated whether customers with more than 5 purchases are more likely to subscribe.
```sql
SELECT subscription_status,
       COUNT(customer_id) AS repeat_buyers
FROM customer1
WHERE previous_purchases > 5
GROUP BY subscription_status;
```

10. **Revenue by Age Group**
   - Analyzed revenue contribution across different customer age groups.
```sql
select age_group,sum(purchase_amount) as total_revenue
from customer1
group by age_group
order by total_revenue desc
```

---     

## 📊 Power BI Dashboard

The Power BI dashboard provides an interactive view of customer behavior and business performance.

### 📌 Dashboard KPIs

- **3.9K** Customers
- **$59.76** Average Purchase Amount
- **3.75** Average Review Rating
- **27%** Subscription Rate

### 📈 Key Visualizations

- Customer Subscription Distribution
- Revenue Contribution by Product Category
- Customer Distribution by Product Category
- Revenue by Age Group
- Customer Distribution by Age Group
- Interactive filters for:
  - Subscription Status
  - Gender
  - Category
  - Shipping Type
 
 The dashboard enables stakeholders to quickly identify customer segments, high-performing categories and purchasing patterns.

--- 

## 💡 Key Business Insights

### 🔹 Subscription Opportunity
Only **27% of customers are subscribers**, while **73% are non-subscribers**, highlighting a significant opportunity to improve subscription conversion.

### 🔹 Category Performance
**Clothing contributes the highest revenue** among the analyzed product categories, making it an important category for revenue-focused campaigns.

### 🔹 Customer Concentration
Clothing also has the **largest customer base**, indicating strong customer demand and an opportunity for cross-selling and upselling.

### 🔹 Age-Based Revenue
The **Young Adult segment contributes the highest revenue**, followed by the Middle-aged segment, making these groups valuable targets for targeted marketing campaigns.

### 🔹 Customer Experience
The overall average review rating of **3.75/5** indicates an opportunity to identify low-rated products and improve customer experience.

---

## 🎯 Business Recommendations

### 1. 🚀 Increase Subscription Conversion
Introduce exclusive subscriber benefits, personalized offers and loyalty rewards to convert more non-subscribers.

### 2. ❤️ Strengthen Customer Loyalty
Reward repeat customers and develop loyalty programs to move customers from **Returning → Loyal** segments.

### 3. 💰 Optimize Discount Strategy
Analyze discount-dependent products carefully to ensure promotional campaigns increase sales without unnecessarily reducing margins.

### 4. 🛍️ Promote High-Performing Products
Use top-rated and best-selling products in marketing campaigns, bundles and cross-selling strategies.

### 5. 🎯 Target High-Value Customer Segments
Prioritize marketing campaigns toward high-revenue age groups and customers demonstrating strong purchase activity.

These recommendations align with the project's business recommendations around subscriptions, loyalty, discounts, product positioning and targeted marketing.

---

## 📷 Dashboard Preview

![CustomerPulse Dashboard](https://github.com/polonimmo115a/customerpulse1-consumer-behavior-analysis/blob/main/Consumer%20Behavior%20Analysis%20dashboard.png)

---

