# Ecommerce Data Analysis and Business Insights

## Introduction
This project contains SQL scripts aimed at performing detailed analysis on an e-commerce data. The analysis focuses on customer behavior, sales trends, product performance, inventory tracking, and operational bottlenecks. The ultimate goal was to hone my skills on SQL by solving advanced business problems, to uncover key insights for making data-driven decisions.

## Background
The e-commerce industry thrives on data, and businesses need effective ways to analyze customer transactions, preferences, and patterns. This repository explores key business questions, including:

 Who are the most valuable customers, and how can they be retained?

What product categories or regions contribute most to revenue?

How do payment failures impact sales, and what causes them?

What are the growth trends over time?

This project showcases how SQL can be used to answer these questions by transforming raw data into meaningful insights.

## Tools I Used
**SQL**: For data exploration, aggregation, and insights generation.

**PostgreSQL:** The database management system used to store and manipulate the data.

**VS Code** : Used for writing and debugging the SQL queries 

**GitHub:** For version control and documentation.

**Excel/Google**Sheets: For visualizing query results where necessary.

**AI(ChatGPT)**: For generating visualisations in the python model,also used to debugg some queries 

## The Analysis

## 1. Customer Retention Analysis
This analysis focuses on identifying customers who placed multiple orders and contributed the most revenue.It shows customers who have been top spenders based o their frequency orders and monetry value they contribute to the total revenue.The chart was filtered to get the top 10 spenders based on the factors stated.

**Steps:**

Counted the total orders and calculated total revenue for each customer.

Filtered out customers with only one order to focus on retained customers.

Ranked customers by total revenue to highlight top contributors.

**Insights:**

High-value customers can be targeted for loyalty programs or exclusive offers.

Retention strategies can focus on converting single-order customers into repeat buyers.

```sql
WITH Customerorders as (
    select
        customer_id,
        count(order_id) as orders_count,
        Sum(total_sale_amount) as Total_revenu
    from ecommerce_data
    group by customer_id
)

select*
from Customerorders
where orders_count > 1
ORDER BY Total_revenu DESC
```

 ![image](https://github.com/user-attachments/assets/76ec217e-162c-469e-aa90-bbbfe576d07b)

## 2. Seasonality Analysis

This analysis identifies how revenue varies across months and categories.It analyses which category of product are the best performers across the entire month and the least performance across the varoius months.

**Steps:**

Extracted the month from transaction dates.

Grouped sales data by month and category to calculate monthly revenue.

Ranked the revenue by month to detect seasonal demand trends.

**Insights:**

Helps forecast high-demand periods for specific categories.

Aids in planning marketing campaigns around seasonal spikes

```sql
 SELECT
        EXTRACT(Month from transaction_date) as Monthly,
        category,
        sum(total_sale_amount) as Monthly_revenue
        from ecommerce_data
    GROUP BY category,Monthly
    order by Monthly_revenue DESC
```

## 3. Market Basket Analysis
Analyzes product pairings frequently purchased together to identify cross-selling opportunities. From the market basket analysis Smartphones and Headsets are the top selling pair with more 600 sales while Headphones and Laptop rank bottom with less than 600 pairs sold

**Steps:**

Self-joined the dataset to pair products bought in the same transaction.

Counted the number of times each product pair was purchased together.

Sorted product pairs by frequency to reveal the most common combinations.

**Insights:**

Frequently paired items can be bundled or promoted together.

Cross-selling opportunities can improve average order value.

```sql
 SELECT e.product_name as Product_1,
        o.product_name as product_2,
            count (*) as Pair_count
    from ecommerce_data as e
    LEFT JOIN ecommerce_data as o ON e.product_id = o.order_id
    and e.order_id < o.product_id
    WHERE o.product_name is NOT NULL

        group by Product_1, product_2
        ORDER BY Pair_count DESC
```

!![image](https://github.com/user-attachments/assets/e81c13c1-9386-40a0-8560-009b3a829d80)

## 4. Customer Segmentation

Segments customers into tiers based on their spending behavior. This helps to analyse which price range contributes more to the company revenue and from the visualisation the "Medium value" spenders contribute a large percentage of the company revenue,meaning the company thrives more on the spending from the medium valued customers

**Steps:**

Calculated total revenue and order count for each customer.

Used thresholds to classify customers as High Value (≥ $50,000), Medium Value ($20,000–$50,000), or Low Value (< $20,000).

**Insights:**

High-value customers can receive VIP treatment and exclusive offers.

Medium-value customers can be encouraged to increase spending through upselling.

```sql
SELECT
    customer_id,
    sum(total_sale_amount) as Monetry_value,
    count(order_id) as Order_count,
case
    WHEN sum(total_sale_amount) >= 50000 then 'High Value'
   WHEN sum(total_sale_amount) > 20000 and sum(total_sale_amount) < 50000 THEN 'Medium value'
   else 'Low value'
    end as customer_segment
    FROM ecommerce_data
    group by customer_id
   order by sum(total_sale_amount) DESC
```
![image](https://github.com/user-attachments/assets/661522d3-1872-4c99-a15c-edb6fe08a67a)

## 5. Revenue Heatmap by Month

Visualizes revenue contribution by country and month. From the chart South africa are the top spenders in the month of November and France are the least spenders in the month of February. This will help to predict future sales drive. 

**Steps:**

Grouped revenue by country and continent.

Extracted the month from transaction dates.

Summarized revenue and sorted by month for heatmap visualization.

**Insights:**

Identifies regions with consistent revenue performance.

Provides a framework to focus sales efforts on underperforming regions.

Visualizes revenue contribution by country and month.

```sql
SELECT
    continent,
    country,
    sum(total_sale_amount) as Monthly_revenue,
    EXTRACT(MONTH from transaction_date) as Month_id
from ecommerce_data
GROUP BY continent,country,Month_id
ORDER BY Monthly_revenue DESC
```

![image](https://github.com/user-attachments/assets/b54a5655-fbad-491e-857b-615be65abb69)

## 6. Top Revenue Drivers by Price

Examines how pricing impacts product revenue. This shows laptops which is the highest priced commodity contributing a gross revenue of over $16m and Headphones which the least priced item contributing the lowest revenue of over $800k

**Steps:**

Grouped products by price ranges: Low (≤ $100), Mid ($100–$500), and High (> $500).

Calculated revenue for each product and price range.

Ranked revenue by price range to identify key revenue drivers.

**Insights:**

Helps determine the optimal pricing strategy for revenue maximization.

High-range products may need enhanced marketing to improve visibility.

```sql
SELECT 
        product_name,
        price_per_unit,
        sum(total_sale_amount) as total_revenue,
    CASE
        when price_per_unit <= 100 then 'Low price_range'
        when price_per_unit BETWEEN 100 and 500 then 'Mid_range'
        ELSE 'High_range'
        end as Price_Range
from ecommerce_data
GROUP BY  product_name,price_per_unit
ORDER BY total_revenue DESC
```

| **Product Name** | **Price Per Unit** | **Total Revenue** | **Price Range**   |
|-------------------|--------------------|--------------------|--------------------|
| Laptop           | $1,000            | $16,009,000       | High Range        |
| Smartphone       | $700              | $11,337,900       | High Range        |
| Tablet           | $300              | $4,929,600        | Mid Range         |
| Smartwatch       | $200              | $3,258,800        | Mid Range         |
| Headphones       | $50               | $818,000          | Low Price Range   |

## 7. Payment Failure Root Cause Analysis

Analyzes failed payment transactions to identify root causes.This shows South Africa with highest payment failure followed closely by USA and Brazil. Germany ranks lowest when it comes to payment failures. Over to the pie chart it shows credit card to be the most failed payment method followed by Bank transfer and paypal

**Steps:**

Filtered for failed payments and grouped by category, country, and payment method.

Calculated failure percentages by category to understand failure distribution.

**Insights:**

Highlights problematic payment methods or regions with high failure rates.

Enables businesses to address payment gateway issues or offer alternative methods.

```sql
SELECT
    category,
    payment_method,
    country,
    payment_status,
    (count(payment_status) * 100 / sum(count(*)) over(PARTITION BY category))
     ::numeric (10,2) :: text || '%'   as failure_percentage
from ecommerce_data
where payment_status = 'Failed'
GROUP BY category,payment_method,country,payment_status
order by failure_percentage DESC
```

![image](https://github.com/user-attachments/assets/5daf5c9f-8dfd-42f0-af75-85e4f72d02be)

![image](https://github.com/user-attachments/assets/15df0057-bbb7-405a-aab5-8d3244b052bb)


## 8. Month-by-Month Revenue Growth

Tracks revenue growth over consecutive months. Shows the inconsistency in the revenue growth while March peaked the highest DEcember recorded the least sales 

**Steps:**

Calculated monthly revenue using SUM.

Used the LAG function to compare each month’s revenue with the previous month.

Derived growth rate as a percentage of the previous month’s revenue.

**Insights:**

Identifies periods of high or low growth.

Provides a basis for setting realistic growth targets.

```sql
SELECT
 --DATE_PART('month', transaction_date) AS Month_id,
  SUM(Total_Sale_Amount) AS monthly_Revenue, 
  LAG(SUM(Total_Sale_Amount)) OVER (ORDER BY DATE_PART('month', transaction_date))
  AS Previous_month_Revenue, 
 ( SUM(Total_Sale_Amount) - LAG(SUM(Total_Sale_Amount)) 
  OVER (ORDER BY DATE_PART('month', transaction_date))) * 100.0 / LAG(SUM(Total_Sale_Amount)) 
  OVER (ORDER BY DATE_PART('month', transaction_date))  AS Growth_Rate
   FROM ecommerce_data
   GROUP By DATE_PART('month', transaction_date)
   ORDER BY DATE_PART('month', transaction_date)
```

![image](https://github.com/user-attachments/assets/d8aed77c-b1eb-4755-838a-77b2c19483b8)

### 9. Inventory Report by Quarter

Summarizes stock levels for products over time.

**Steps:**

Grouped products by category and quarter.

Summed up the stock quantity for each product.

Ranked inventory levels by quarter to track product availability.

**Insights:**

Ensures optimal stock levels are maintained for each quarter.

Reduces the risk of overstocking or stockouts.

```sql
SELECT
    product_name,
    category,
    sum(quantity) as stock_level,
     EXTRACT(quarter from transaction_date) as Quarter
from ecommerce_data
GROUP BY category,product_name,quarter
order by Quarter,stock_level
```

| **Product Name** | **Category**      | **Stock Level** | **Quarter** |
|-------------------|-------------------|------------------|-------------|
| Tablet           | Electronics       | 3,798           | 1           |
| Laptop           | Electronics       | 4,011           | 1           |
| Smartphone       | Electronics       | 4,011           | 1           |
| Headphones       | Accessories       | 4,027           | 1           |
| Smartwatch       | Accessories       | 4,061           | 1           |
| Smartphone       | Electronics       | 4,056           | 2           |
| Laptop           | Electronics       | 4,088           | 2           |
| Smartwatch       | Accessories       | 4,105           | 2           |
| Tablet           | Electronics       | 4,108           | 2           |
| Headphones       | Accessories       | 4,119           | 2           |
| Laptop           | Electronics       | 4,005           | 3           |
| Smartphone       | Electronics       | 4,039           | 3           |
| Headphones       | Accessories       | 4,054           | 3           |
| Smartwatch       | Accessories       | 4,130           | 3           |
| Tablet           | Electronics       | 4,324           | 3           |
| Laptop           | Electronics       | 3,905           | 4           |
| Smartwatch       | Accessories       | 3,998           | 4           |
| Smartphone       | Electronics       | 4,091           | 4           |
| Headphones       | Accessories       | 4,160           | 4           |
| Tablet           | Electronics       | 4,202           | 4           |

## What I Learned
How to use SQL for advanced data analysis, including window functions like LAG and OVER.

The importance of customer segmentation in tailoring marketing strategies and improving retention.

How market basket analysis can directly improve sales through cross-selling and bundling strategies.

The need for payment optimization to reduce lost revenue from failed transactions.

How seasonality and regional trends influence sales performance.

How to prompt AI (chatgpt) to debug codes and also to generate visuals 

## Conclusions

This project demonstrates how SQL can transform raw data into actionable business insights. The analyses provide frameworks to improve customer retention, optimize inventory, target high-value customers, and address payment issues. These scripts can be adapted for any e-commerce dataset to drive growth and operational efficiency.
It also addressed complex business prolems and provided key insigts on how to analyse them with the data on grond 

Explore the repository to leverage these SQL queries for your own e-commerce data analysis needs
