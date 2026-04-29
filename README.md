# Customer Behaviour Analysis
### End-to-End Data Analytics Portfolio Project &nbsp;|&nbsp; Python · PostgreSQL · Power BI

---

## Table of Contents
- [Overview](#overview)
- [Business Problem](#business-problem)
- [Dataset](#dataset)
- [Tools & Technologies](#tools--technologies)
- [Project Structure](#project-structure)
- [Workflow](#workflow)
  - [1. Data Preparation & EDA — Python](#1-data-preparation--eda--python)
  - [2. Data Analysis — PostgreSQL](#2-data-analysis--postgresql)
  - [3. Dashboard — Power BI](#3-dashboard--power-bi)
- [Dashboard](#dashboard)
- [Key Findings](#key-findings)
- [Business Recommendations](#business-recommendations)
- [Acknowledgements](#acknowledgements)
- [How to Run](#how-to-run)
- [Author](#author)

---

## Overview

This project walks through a complete, end-to-end data analytics pipeline, the kind that mirrors what an analyst would actually do inside a retail business. The dataset represents a hypothetical US-based fashion and apparel firm, and the goal is straightforward: take raw customer transaction data, clean it, interrogate it with SQL, and turn it into a dashboard that tells a clear story to decision-makers.

What started as a structured learning exercise became something more. Beyond the foundational workflow, this project independently extends into original analysis — an additional layer of SQL logic, a full second dashboard page, and ten DAX measures built from scratch — to surface insights that a business could realistically act on. The result is not just a technical demonstration but an analytical argument: here is what the data says, here is why it matters, and here is what the company should do about it.

---

## Business Problem

Retail businesses sit on enormous amounts of customer data and often do very little with it. This company is no different. Management has observed changes in how customers shop — shifts in what they buy, when they buy, and how they respond to promotions — but has no structured view of what is actually driving those patterns.

The core question this project sets out to answer is:

> *How can the company use its customer shopping data to identify behavioural trends, strengthen loyalty, and make smarter decisions about marketing, promotions, and product strategy?*

To answer that, the analysis examines customer demographics, seasonal demand, product category performance, shipping preferences, discount behaviour, payment habits, and the gap between who the company's most loyal customers are versus who is actually enrolled in the subscription programme.

---

## Dataset

| Property | Detail |
|---|---|
| **File** | `customer_shopping_behaviour.csv` |
| **Domain** | Hypothetical US-based fashion and apparel retail |
| **Source** | Publicly available consumer behavior dataset (Synthetic) |
| **Records** | 3,900 customer transactions |
| **Original Columns** | 18 |
| **Engineered Columns** | `age_group`, `purchase_frequency_days` |

**Columns used across the analysis:**

`customer_id` · `age` · `gender` · `item_purchased` · `category` · `purchase_amount` · `location` · `season` · `review_rating` · `subscription_status` · `shipping_type` · `discount_applied` · `previous_purchases` · `payment_method` · `frequency_of_purchases` · `age_group` · `purchase_frequency_days`

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **Python** — pandas, SQLAlchemy, psycopg2 | Data loading, EDA, cleaning, feature engineering, database export |
| **PostgreSQL & pgAdmin 4** | Relational storage, business-logic querying, customer segmentation |
| **Power BI Desktop** | Two-page interactive dashboard, DAX development |
| **GitHub** | Version control and portfolio showcase |

---

## Project Structure

```
Customer-Behaviour-Analysis/
│
├── data/
│   └── customer_shopping_behaviour.csv
│
├── python/
│   └── Customer_Shopping_Behaviour_Analysis.ipynb
│
├── sql/
│   └── customer_shopping_behaviour_analysis.sql
│
├── powerbi/
│   ├── customer_behaviour_dashboard.pdf
│   └── customer_behaviour_dashboard.pbix
│
├── images/
│   ├── powerbi_dashboard_pg1.png
│   ├── powerbi_dashboard_pg2.png
│   ├── python_eda_output1.png
│   ├── python_eda_output2.png
│   ├── python_eda_output3.png
│   ├── python_cleaning1.png
│   ├── python_cleaning2.png
│   ├── python_cleaning3.png
│   ├── python_cleaning4.png
│   └── sql_query_result.png
│
└── README.md
```

---

## Workflow

### 1. Data Preparation & EDA — Python

**Notebook:** `python/Customer_Shopping_Behaviour_Analysis.ipynb`

The first step was understanding what the data actually contains before working with it. `df.info()` confirmed 3,900 rows across 18 columns, with `Review Rating` being the only column carrying null values — 37 of them.

![df.info() output](images/python_eda_output1.png)

Running `df.describe(include='all')` gave a full statistical picture. Through its results, we are able to infer details like the average customer is 44 years old, spends $59.76 per transaction, and rates their purchase 3.75 out of 5. Also, purchase amounts range from $20 to $100, and the most common item is a Blouse, the most common category is Clothing, and the most common shipping method is Free Shipping, etc.

![df.describe() output](images/python_eda_output2.png)

The 37 missing review ratings were imputed using the **median rating of each product category** rather than a simple global median. This is a deliberate choice: a global median flattens category-level differences in satisfaction, whereas a group-aware imputation respects the fact that Outerwear customers and Footwear customers may rate their purchases differently on average.

![Null check and imputation](images/python_eda_output3.png)

**Data Cleaning**

All column names were standardised to snake_case for clean PostgreSQL compatibility, and `purchase_amount_(usd)` was manually corrected to `purchase_amount`.

![Column renaming](images/python_cleaning1.png)

Two new features were engineered. `age_group` was created using `pd.qcut()` with four quantile-based bins — `Young Adult`, `Adult`, `Middle-aged`, `Senior` — distributing customers evenly across life-stage segments to enable meaningful demographic comparison.

![age_group feature engineering](images/python_cleaning2.png)

`purchase_frequency_days` converted the text-based frequency column into a numeric interval (Weekly → 7, Fortnightly → 14, Monthly → 30, Quarterly → 90, Annually → 365), making purchase tempo usable as a quantitative measure in both SQL and Power BI.

![purchase_frequency_days engineering](images/python_cleaning3.png)

An additional column audit revealed that `discount_applied` and `promo_code_used` were **perfectly identical across all 3,900 rows**, a redundant column that would silently inflate any model treating them as separate signals.

![Duplicate column audit](images/python_cleaning4.png)

---

### 2. Data Analysis — PostgreSQL

**Script:** `sql/customer_shopping_behaviour_analysis.sql`

The cleaned DataFrame was loaded into PostgreSQL as `public.customer` via SQLAlchemy. Every query in the script is paired with an inline `/* RESULT: */` block that interprets the output as a business finding, not just a number, but what that number means.

| # | Query | Finding |
|---|---|---|
| 1 | Revenue by gender | Male customers generated $157,890 vs. $75,191 from female customers |
| 2 | High spenders who still used a discount | 839 customers exceeded the $59.76 average spend despite applying a discount |
| 3 | Top 5 products by average review rating | Gloves (3.86), Sandals (3.84), Boots (3.82), Hats (3.80), Skirts (3.78) |
| 4 | Avg spend: Standard vs. Express shipping | Express ($60.48) outpaces Standard ($58.46) |
| 5 | Subscribers vs. non-subscribers | 1,053 subscribers generate $62K vs. $170K from 2,847 non-subscribers |
| 6 | Products with highest discount application rates | Hats (50%), Sneakers (49%), Coats (49%), Sweaters (48%), Pants (47%) |
| 7 | Customer segmentation by loyalty tier | Loyal: 3,116 · Returning: 701 · New: 83 |
| 8 | Top 3 products per category *(Window Function)* | Jewelry leads Accessories; Blouse leads Clothing; Sandals lead Footwear; Jacket leads Outerwear |
| 9 | Repeat buyer and subscription overlap | Only 958 of 3,476 repeat buyers (28%) have subscribed |
| 10 | Revenue by age group | Young Adults lead at $62,143; Seniors trail at $55,763 |

The window function query below is one of the most structurally advanced queries in the script, using `ROW_NUMBER() OVER (PARTITION BY category)` inside a CTE to rank products within each category by order volume — a pattern common in real retail analytics work.

![SQL Window Function — pgAdmin output](images/sql_query_result.png)

---

### 3. Dashboard — Power BI

**File:** `powerbi/customer_behaviour_dashboard.pbix`

The dashboard was built directly on top of the PostgreSQL connection and organised across two pages with distinct analytical purposes. All measures exist in a dedicated `_Measures` table in the model view.

**DAX Measures:**

```dax
-- Page 1: Core KPIs
Number of Customers     = COUNT('public customer'[customer_id])
Average Purchase Amount = AVERAGE('public customer'[purchase_amount])
Average Review Rating   = AVERAGE('public customer'[review_rating])
Total Revenue           = SUM('public customer'[purchase_amount])

-- Page 2: Behavioural Intelligence
Discount Rate % =
    DIVIDE(
        COUNTROWS(FILTER('public customer', 'public customer'[discount_applied] = "Yes")),
        COUNTROWS('public customer'), 0)

Loyal Customer Rate % =
    DIVIDE(
        COUNTROWS(FILTER('public customer', 'public customer'[previous_purchases] > 10)),
        COUNTROWS('public customer'), 0)

Subscriber Revenue Share % =
    DIVIDE(
        CALCULATE(SUM('public customer'[purchase_amount]),
                  'public customer'[subscription_status] = "Yes"),
        SUM('public customer'[purchase_amount]), 0)

Avg Spend With Discount =
    CALCULATE(AVERAGE('public customer'[purchase_amount]),
              'public customer'[discount_applied] = "Yes")

Avg Spend Without Discount =
    CALCULATE(AVERAGE('public customer'[purchase_amount]),
              'public customer'[discount_applied] = "No")

Discount Spend Lift = [Avg Spend With Discount] - [Avg Spend Without Discount]

Revenue per Customer =
    DIVIDE(SUM('public customer'[purchase_amount]),
           COUNT('public customer'[customer_id]), 0)
```

---

## Dashboard

### Page 1 — Customer Shopping Analysis
*Interactive slicers: Subscription Status · Gender · Category · Shipping Type*

![Page 1](images/powerbi_dashboard_pg1.png)

The first page establishes the business baseline. Three headline KPI cards anchor the top — 3,900 customers, $59.76 average spend, 3.75 average review rating. Below, the subscription donut (27% Yes, 73% No), paired revenue and sales-count charts by category, and mirrored age-group bars together build a clear picture of who the customer base is, what they buy, and how much they spend.

---

### Page 2 — Behavioural Drivers & Seasonal Patterns *(original extension)*
*Interactive slicers: Gender · Season*

![Page 2](images/powerbi_dashboard_pg2.png)

The second page was developed independently and goes beyond what most entry-level projects attempt. Rather than simply describing the data, it asks sharper questions: Does discounting actually increase spend, or does it just attract bargain hunters? Do subscribers behave differently when choosing shipping? Which season drives the most revenue, and is that driven by volume or by spend per visit? Each visual is an answer to a specific business question rather than a chart chosen for variety.

---

## Key Findings

### Demographics & Revenue

- **Male customers generate 68% of total revenue** — $157,890 compared to $75,191 from female customers. Given that male customers also account for the majority of the customer base (2,652 of 3,900), it is unclear whether this revenue gap reflects a genuine spending difference or simply an acquisition imbalance. Either way, female customers represent a significantly underdeveloped revenue segment.

- **Young Adults are the highest-spending age group** at $62,143 in total revenue and 1,028 transactions, followed by Middle-aged customers at $59,197. Seniors and Adults generate almost identical revenue at $55,763 and $55,978 respectively. The gap between Young Adults and every other group is consistent across both revenue and transaction count, making this the company's most commercially valuable demographic.

### Loyalty & Subscription

- **80% of customers qualify as loyal** — 3,116 of 3,900 have made more than 10 previous purchases. Only 83 customers are genuinely new. This is a deeply repeat-oriented customer base, which makes the next finding all the more striking.

- **The subscription programme captures just 26.9% of total revenue** despite having access to a customer base that is overwhelmingly loyal. Only 1,053 customers have subscribed, generating $62,645 — while 2,847 non-subscribers account for $170,436. The programme is not failing to attract good customers; it is failing to convert them.

- **72% of repeat buyers have never subscribed** — 2,518 of the 3,476 customers with more than 5 previous purchases remain outside the programme. These customers are already demonstrating long-term commitment through their purchase behaviour; the subscription offer is simply not compelling enough to formalise that relationship.

### Seasonal Demand & Category Patterns

- **Fall is the highest-revenue season at $60,018**, despite Spring recording the most transactions (999). Summer is the weakest season at $55,777. The fact that Fall generates more revenue from fewer transactions than Spring points to higher average spend per visit in Fall — almost certainly driven by Outerwear, which is the lowest-volume but highest-ticket category.

- **Clothing dominates across all seasons**, generating $104K in total revenue and 1,737 transactions — more than double any other category. Its seasonal distribution is notably even across all four seasons, meaning Clothing's dominance is structural rather than demand-driven. The same uniformity holds across Accessories, Footwear, and Outerwear, where transaction counts vary by fewer than 20 units season to season. This likely reflects the synthetic nature of the dataset rather than genuine seasonal consumer behaviour, a limitation worth acknowledging when drawing seasonal conclusions.

- **Spring produces the highest customer satisfaction**, with an average review rating of 3.79, compared to 3.75 in Winter and 3.73 in both Fall and Summer. While these differences are small in absolute terms, a consistent satisfaction dip in the two highest-revenue seasons (Fall and Winter) is worth monitoring; it may reflect fulfilment pressure during peak periods.

### Discount Behaviour

- **43% of all purchases involve a discount**, which is a high promotional intensity for a business where average spend is only $59.76. The more important question is whether discounts are actually driving value, and the answer depends heavily on the category.

- **Discounted buyers spend less across most categories, but more in Footwear.** In Clothing, Accessories, and Outerwear, customers who applied a discount spent less on average than those who did not — $59.75 vs. $60.22, $58.49 vs. $60.89, and $55.32 vs. $58.66 respectively. This is consistent with discounts attracting more price-sensitive buyers in these categories. Footwear is the exception: discounted buyers spent $61.80 compared to $59.08 without a discount, which may reflect that discount codes in this category are disproportionately used by customers purchasing higher-priced items such as boots or premium sneakers rather than budget options. Across three of four categories, the data suggests discounts are not driving higher-value purchases.

- **839 customers spent above the average of $59.76 even while using a discount**, confirming that a meaningful subset of the discounted base was already willing to spend. These customers are effectively receiving a margin concession they did not need to convert.

- **Hats, Sneakers, and Coats carry the highest discount application rates** at 50%, 49%, and 49% respectively. These are not the highest-revenue products, which raises the question of whether heavy discounting on these items is building volume or simply eroding margin without proportionate demand.

### Shipping & Payment

- **Free Shipping generates the most total revenue at $41K**, more than any premium shipping option. However, this reflects customer volume choosing that option rather than any per-transaction premium; average spend across shipping types is relatively consistent, sitting between $57 and $61 for most options.

- **Subscribers spend more than non-subscribers when using Standard and Next Day Air** ($60.44 vs. $57.74 and $60.26 vs. $58.06 respectively), but the pattern reverses for other shipping types, where non-subscribers spend more. Subscription status is not a reliable predictor of per-transaction spend across all shipping contexts, but subscriber behaviour on Standard Shipping is a meaningful signal worth acting on.

- **Payment method is almost perfectly distributed** across all six options: Credit Card leads narrowly at 17.29%, followed by PayPal at 17.21%, Cash at 17.16%, Debit Card at 16.62%, Venmo at 16.03%, and Bank Transfer at 15.68%. No single method dominates. This is unusual and may reflect deliberate platform design or the synthetic nature of the dataset, but operationally it means there is no meaningful payment preference to optimise around.

### Purchase Frequency

- **Bi-Weekly buyers record the highest average spend at $60.69**, while Weekly buyers record the lowest at $58.97. The counterintuitive result here is that the most frequent buyers, those purchasing every week, are spending the least per visit. The highest per-transaction spenders buy every two weeks or less frequently. This is a classic high-frequency, low-basket pattern: habitual weekly buyers make smaller, routine purchases, while less frequent buyers are more likely making considered, higher-value ones.

---

## Business Recommendations

**1. Rebuild the subscription programme around the loyal majority.**
The data makes clear that the loyalty problem is not retention, it is conversion. The company already has 3,116 loyal customers; it just has not given them a compelling reason to subscribe. The programme needs tangible benefits tied to what these customers actually do: priority access to new season releases, complimentary Standard Shipping (where subscribers demonstrably spend more), or a points system that compounds with purchase history. The target conversion audience is the 2,518 repeat buyers who have never subscribed — a warm, high-frequency base that no cold acquisition campaign can replicate.

**2. Audit discount effectiveness by category before scaling promotional spend.**
In Clothing, Accessories, and Outerwear, discounted transactions are associated with lower average spend — suggesting discounts in these categories are reaching price-sensitive buyers rather than converting hesitant high-value ones. Footwear runs counter to this pattern, where discounted buyers spend more, warranting closer investigation into which specific products are driving that difference. A structured review of discount allocation by category, followed by targeted A/B testing, would determine whether current promotional spend is generating genuine incremental revenue or simply subsidising purchases that would have occurred regardless.

**3. Invest marketing spend behind high-rated, under-purchased products.**
Gloves, Sandals, and Boots hold the top three average review ratings in the entire catalogue yet are not among the top-selling products in their respective categories. High satisfaction with low volume is a product marketing problem, not a product quality problem. These items deserve increased placement in campaign creative, particularly in the seasons where they are most relevant — Sandals in Spring, Boots and Gloves in Fall and Winter (based on conventional retail seasonality rather than this dataset, which shows near-uniform category distribution across seasons). Directing customers toward products they are most likely to rate highly improves review scores, repeat purchase intent, and word-of-mouth simultaneously.

**4. Target bi-weekly buyers as the highest-priority segment for subscription conversion and upsell.**
Bi-weekly buyers record the highest average spend across all frequency groups at $60.69, and their purchase tempo, regular but not habitual, suggests considered, intentional buying behaviour rather than routine low-basket shopping. This makes them the segment most likely to respond to a subscription offer with tangible benefits. Combined with the fact that 72% of repeat buyers have never subscribed, a conversion campaign targeting specifically bi-weekly frequency customers represents the most commercially efficient use of retention marketing spend.

**5. Treat the female customer gap as two separate problems, not one.**
First, the spend gap: female customers generate $75,191 compared to $157,890 from male customers, but with only 1,248 female customers versus 2,652 male, the gap may reflect an acquisition imbalance rather than a behavioural one. Second, and more critically: there are no female subscribers recorded in the dataset at all. Every one of the 1,053 subscribers is male. Whether this reflects a data quality issue or a genuine product-market misalignment, it means the subscription programme is currently serving only half the customer population by gender. Any subscription growth strategy that does not explicitly address female customer conversion is structurally capped from the outset.

**6. Protect margin on top-discounted products in high-volume categories.**
Hats, Sneakers, and Coats see discounts applied at nearly half of all purchases. These are not low-volume niche items, they sit within the most-purchased categories. A 50% discount rate on Hats, for instance, means the company is effectively pricing at a promotional rate as the new baseline. Gradually testing discount frequency reductions on these specific items, while monitoring volume sensitivity, would help establish whether the current discount rate is genuinely driving demand or has simply become the expected price.

---

## Acknowledgements

This project was inspired by the YouTube tutorial **"COMPLETE Data Analytics Portfolio Project in 6 EASY Steps | Python + SQL + Power BI"** by [Amlan Mohanty](https://www.youtube.com/@amlanmohanty1), which provided the foundational workflow structure and dataset as a learning framework.

**Independent extensions developed in this version:**

- Inline `/* RESULT: */` business interpretations added to every SQL query
- Ten additional DAX measures including `Discount Rate %`, `Loyal Customer Rate %`, `Subscriber Revenue Share %`, `Discount Spend Lift`, and `Revenue per Customer`
- Fully original **Page 2 dashboard** covering seasonal demand, category-level discount impact, shipping type behaviour, payment method distribution, subscription-adjusted spending, and purchase frequency analysis

---

## How to Run

**Prerequisites:** Python 3.8+, PostgreSQL (local), Power BI Desktop (Windows, free)

```bash
# 1. Clone the repository
git clone https://github.com/your-username/Customer-Behaviour-Analysis.git
cd Customer-Behaviour-Analysis

# 2. Install Python dependencies
pip install pandas sqlalchemy psycopg2-binary jupyter

# 3. Update the database connection string in the notebook
# engine = create_engine('postgresql://username:password@localhost:5432/your_database')

# 4. Run the notebook — cleans the data and loads PostgreSQL
jupyter notebook python/customer_behaviour_analysis.ipynb
```

Once the notebook has run, open `sql/customer_shopping_behaviour_analysis.sql` in pgAdmin and execute queries sequentially. Then open `powerbi/customer_behaviour_dashboard.pbix` in Power BI Desktop — if prompted, update the PostgreSQL connection credentials under **Transform Data → Data Source Settings**.

---

## Author

**Ayman Zahid**
Data Analyst | Townsville, Queensland, Australia

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://www.linkedin.com/in/ayman-zahid)
[![GitHub](https://img.shields.io/badge/GitHub-Portfolio-black)](https://github.com/ayman-zahid)
