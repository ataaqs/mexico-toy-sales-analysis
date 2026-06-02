# Mexico Toys Sales & Inventory Performance Analysis

## Project Overview
This project analyzes sales and inventory performance across toy store locations in Mexico using SQL. The analysis focuses on profitability, seasonal sales trends, stock availability, and inventory efficiency to generate actionable business insights.
---

## Objectives
- Identify the most profitable product categories
- Analyze profitability across store locations
- Analyze seasonal sales patterns and trends
- Evaluate inventory coverage and stock efficiency
- Detect products with high stockout risk
---

## Tools Used
- SQL
- Excel
- Github
---

## Dataset
Public retail toy sales dataset containing
- Sales transactions
- Product categories
- Store locations
- Inventory levels
- Revenue and profit data
---

## Business Questions

### 1. Which product categories generate the highest profit?
Analyze product category performance to identify the main profit contributors across the business.

### 2. How does profitability vary across store locations?
Evaluate profit performance by store location and product category to understand regional demand patterns.

### 3. Are there seasonal trends or sales patterns?
Identify monthly revenue and profit trends to detect peak and low sales periods throughout the year.

### 4. How long will current inventory last based on sales demand?
Measure inventory coverage using average daily sales to evaluate stock health and potential overstock or stockout risk.

### 5. Which products are most frequently out of stock across stores?
Analyze stockout frequency to identify products with the highest inventory risk and replenishment priority.

---

## Analysis & Insights

### 1. Product Categories with Highest Profit
#### SQL Query
```sql
SELECT
    p.Product_Category,
    SUM((p.Product_Price - p.Product_Cost) * s.Units) AS Total_Profit
FROM Sales s
JOIN Products p ON s.Product_ID = p.Product_ID
GROUP BY p.Product_Category
ORDER BY Total_Profit DESC;
```

#### Findings
The Toys category generated the highest total profit at approximately **1.08 million**, followed closely by Electronics with around **1.00 million** in total profit.

Meanwhile, Art & Crafts and Games contributed moderate profit levels, while Sports & Outdoors produced the lowest total profit among all categories analyzed.

| Product Category | Total Profit |
|---|---|
| Toys | 1,079,527 |
| Electronics | 1,001,437 |
| Art & Crafts | 753,354 |
| Games | 673,993 |
| Sports & Outdoors | 505,718 |

---

#### Business Insight

The results indicate that Toys and Electronics were the primary profit drivers for the business, suggesting consistently strong customer demand and effective product pricing within these categories.

On the other hand, Sports & Outdoors generated substantially lower profit compared to other categories, which may indicate lower demand, lower margins, or weaker sales performance.

The relatively small gap between Toys and Electronics also suggests that both categories play a critical role in sustaining overall store profitability.

---

#### Recommendation

- Prioritize inventory allocation for Toys and Electronics to avoid stock shortages in high-demand products.
- Increase promotional focus on top-performing categories to maximize revenue opportunities.
- Reevaluate product assortment and pricing strategies for Sports & Outdoors to improve category profitability.
- Conduct store-level category analysis to identify regional differences in customer preferences.

---

### 2. Profitability by Store Location
#### SQL Query
```sql
SELECT 
    st.Store_Location,
    p.Product_Category,
    SUM((p.Product_Price - p.Product_Cost) * s.Units) AS total_profit
FROM Sales s
JOIN Products p ON s.Product_ID = p.Product_ID
JOIN Stores st ON s.Store_ID = st.Store_ID
GROUP BY st.Store_Location, p.Product_Category
ORDER BY total_profit DESC;
```

---

#### Findings

Downtown stores generated the highest overall profitability across nearly all product categories, particularly in Toys and Electronics.

Toys produced the highest profit in Downtown locations with approximately **630,029**, while Electronics dominated in Commercial areas with around **287,574** in total profit.

Airport and Residential locations generated comparatively lower profits across all categories, with Sports & Outdoors consistently showing the weakest performance regardless of store location.

| Store Location | Top Category | Total Profit |
|---|---|---|
| Downtown | Toys | 630,029 |
| Commercial | Electronics | 287,574 |
| Residential | Toys | 136,214 |
| Airport | Electronics | 108,197 |

---

#### Business Insight

Profitability patterns varied significantly by store location, indicating that customer purchasing behavior differs across retail environments.

Downtown stores appeared to benefit from higher customer traffic and stronger demand across multiple categories, making them the primary revenue drivers for the business.

Meanwhile, Commercial locations showed stronger preference toward Electronics products, suggesting that inventory allocation strategies should be adapted based on local demand patterns rather than applying a uniform distribution model across all stores.

The consistently lower profitability of Sports & Outdoors may indicate weaker customer demand or less effective product positioning.

---

#### Recommendation

- Implement location-based inventory strategies tailored to customer demand in each store area.
- Increase stock allocation for Toys in Downtown stores and Electronics in Commercial stores.
- Reevaluate product assortment for low-performing categories such as Sports & Outdoors.
- Prioritize high-performing store locations for promotional campaigns and new product launches.

---

### 3. Seasonal Sales Trends
#### SQL Query
```sql
SELECT 
    strftime('%Y-%m', substr(Date, 7, 4) || '-' || substr(Date, 4, 2) || '-' || substr(Date, 1, 2)
    ) AS month,
    SUM(p.Product_Price * s.Units) AS revenue,
    SUM((p.Product_Price - p.Product_Cost) * s.Units) AS profit
FROM Sales s
JOIN Products p ON s.Product_ID = p.Product_ID
GROUP BY month
ORDER BY month;
```

---

#### Findings

Sales performance showed clear monthly fluctuations throughout the analyzed period.

Revenue and profit generally increased toward the end of the year, with the highest performance recorded in **December 2022**, generating approximately:

- **Revenue:** 877,204
- **Profit:** 246,078

Strong sales performance also appeared in:
- March 2023
- April 2023
- May 2023

Meanwhile, lower sales activity was observed during:
- August 2022
- January–February 2022

| Month | Revenue | Profit |
|---|---|---|
| 2022-12 | 877,204 | 246,078 |
| 2023-03 | 883,516 | 231,909 |
| 2023-04 | 827,691 | 215,096 |
| 2022-08 | 489,423 | 158,931 |

---

#### Business Insight

The data indicates strong seasonality in toy store sales performance, with demand increasing significantly during year-end periods and remaining relatively strong during early 2023.

The spike in December sales likely reflects holiday shopping behavior, seasonal gifting trends, and increased consumer spending during festive periods.

Lower sales periods may represent off-season demand cycles, highlighting the importance of adjusting inventory and promotional strategies throughout the year.

The consistent relationship between revenue growth and profit growth also suggests healthy pricing and margin management during peak demand periods.

---

#### Recommendation

- Prepare additional inventory ahead of peak sales periods, particularly during Q4 and holiday seasons.
- Increase marketing campaigns before high-demand months to maximize revenue opportunities.
- Use historical monthly trends to improve demand forecasting and inventory planning.
- Implement targeted promotions during slower sales periods to stabilize revenue performance throughout the year.

---

### 4. Inventory Coverage Analysis
#### SQL Query
```sql
WITH daily_sales AS (
    SELECT 
        Product_ID,
        Date,
        SUM(Units) AS units_per_day
    FROM Sales
    GROUP BY Product_ID, Date
),
avg_sales AS (
    SELECT 
        Product_ID,
        AVG(units_per_day) AS avg_daily_sales
    FROM daily_sales
    GROUP BY Product_ID
),
total_stock AS (
    SELECT 
        Product_ID,
        SUM(Stock_On_Hand) AS total_stock
    FROM Inventory
    GROUP BY Product_ID
)
SELECT 
    p.Product_Name,
    t.total_stock,
    ROUND(a.avg_daily_sales, 2) AS avg_daily_sales,
    ROUND((t.total_stock * 1.0 / NULLIF(a.avg_daily_sales, 0)), 2) AS coverage_days,
    CASE 
        WHEN (t.total_stock * 1.0 / NULLIF(a.avg_daily_sales, 0)) < 3 THEN 'Stockout Risk'
        WHEN (t.total_stock * 1.0 / NULLIF(a.avg_daily_sales, 0)) BETWEEN 3 AND 30 THEN 'Healthy'
        WHEN (a.avg_daily_sales IS NULL OR a.avg_daily_sales = 0) THEN 'No Sales'
        ELSE 'Overstock'
    END AS stock_status
FROM total_stock t
LEFT JOIN avg_sales a ON t.Product_ID = a.Product_ID
JOIN Products p ON p.Product_ID = t.Product_ID
ORDER BY coverage_days ASC;
```

---

#### Findings

The analysis estimated how long current inventory levels could support average daily sales demand for each product.

Most products were categorized as **Healthy**, meaning inventory levels are generally sufficient to support ongoing sales demand.

Some products with relatively lower inventory coverage include:

| Product Name | Coverage Days | Stock Status |
|---|---|---|
| Playfoam | 3.86 | Healthy |
| Barrel O' Slime | 5.89 | Healthy |
| Jenga | 6.17 | Healthy |
| Action Figure | 6.74 | Healthy |
| Colorbuds | 7.08 | Healthy |

Meanwhile, several products showed significantly higher inventory coverage levels, indicating potential overstock conditions:

| Product Name | Coverage Days | Stock Status |
|---|---|---|
| PlayDoh Playset | 30.39 | Overstock |
| PlayDoh Toolkit | 36.10 | Overstock |
| Dinosaur Figures | 48.32 | Overstock |

---

#### Business Insight

The results suggest that overall inventory management is relatively balanced, as most products fall within the healthy inventory coverage range.

However, products with very high coverage days may indicate slower inventory turnover and excessive stock accumulation. Overstocked items can increase storage costs and tie up working capital that could be allocated more efficiently elsewhere.

On the other hand, products with lower coverage days should still be monitored carefully to prevent future stockout risks if demand increases unexpectedly.

---

#### Recommendation

- Maintain current inventory strategies for products in the healthy coverage range.
- Reduce purchasing volume for overstocked products with slow inventory turnover.
- Monitor low-coverage products more frequently to avoid stock shortages.
- Use inventory coverage metrics regularly to improve replenishment planning and inventory efficiency.

---

### 5. Stockout Risk Analysis
#### SQL Query
```sql
WITH out_of_stock AS (
    SELECT Product_ID, Store_ID
    FROM inventory
    WHERE Stock_On_Hand = 0
),
total_store AS (
    SELECT COUNT(DISTINCT Store_ID) AS total_store
    FROM inventory
),
base AS (
    SELECT
        p.Product_Name,
        COUNT(o.Store_ID) AS out_of_stock_count,
        t.total_store,
        ROUND(COUNT(o.Store_ID) * 1.0 / t.total_store, 2) AS stockout_rate
    FROM out_of_stock o
    INNER JOIN products p ON p.Product_ID = o.Product_ID
    CROSS JOIN total_store t
    GROUP BY p.Product_ID, p.Product_Name, t.total_store
)
SELECT *,
    CASE 
        WHEN stockout_rate >= 0.2 THEN 'Critical'
        WHEN stockout_rate >= 0.1 THEN 'Medium'
        ELSE 'Low Risk'
    END AS risk_level
FROM base
ORDER BY stockout_rate DESC;
```

---

#### Findings

The analysis identified several products that frequently experienced stockout conditions across multiple store locations.

Products with the highest stockout rates include:

| Product Name | Stockout Rate | Risk Level |
|---|---|---|
| Playfoam | 0.26 | Critical |
| Hot Wheels 5-Pack | 0.24 | Critical |
| Etch A Sketch | 0.14 | Medium |
| Dino Egg | 0.12 | Medium |
| Foam Disk Launcher | 0.12 | Medium |
| Mini Ping Pong Set | 0.12 | Medium |
| Action Figure | 0.10 | Medium |

Meanwhile, products such as Animal Figures, Plush Pony, Teddy Bear, and Chutes & Ladders showed relatively low stockout rates and were categorized as low risk.

---

#### Business Insight

The results suggest that certain products consistently experience inventory shortages across multiple stores, particularly Playfoam and Hot Wheels 5-Pack, which were classified as critical risk products.

High stockout rates may indicate strong customer demand, insufficient replenishment planning, or inventory allocation issues across store locations. If these shortages continue, stores could potentially lose sales opportunities and reduce customer satisfaction due to unavailable products.

The analysis also shows that not all products face the same inventory pressure, indicating that demand patterns vary significantly between product categories. Monitoring products with medium and critical stockout risk is important to maintain stable inventory availability and support overall sales performance.

---

#### Recommendation

- Prioritize replenishment for products with critical stockout risk levels.
- Improve demand forecasting for fast-selling products with repeated shortages.
- Increase inventory monitoring frequency for medium and critical risk products.
- Evaluate inventory distribution strategies to ensure balanced stock allocation across stores.
- Use stockout rate metrics as part of regular inventory performance monitoring.

---

## Project Structure

```bash
mexico-toy-sales-analysis/
│
├── dataset/
│   ├── sales.csv
│   ├── products.csv
│   ├── stores.csv
│   └── inventory.csv
│
└── README.md
```

---

## Status
Finished — open for improvement and future dashboard development.
