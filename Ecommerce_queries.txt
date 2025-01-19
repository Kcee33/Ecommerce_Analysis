-- 1.Customer Retention Analysis
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


--2.Seasonality Analysis   
 SELECT
        EXTRACT(Month from transaction_date) as Monthly,
        category,
        sum(total_sale_amount) as Monthly_revenue
        from ecommerce_data
    GROUP BY category,Monthly
    order by Monthly_revenue DESC

--3.Market Basket Analysis
    SELECT e.product_name as Product_1,
        o.product_name as product_2,
            count (*) as Pair_count
    from ecommerce_data as e
    LEFT JOIN ecommerce_data as o ON e.product_id = o.order_id
    and e.order_id < o.product_id
    WHERE o.product_name is NOT NULL

        group by Product_1, product_2
        ORDER BY Pair_count DESC
--4. Customer segmentation
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

--5 Revenue Heatmap by month
SELECT
    continent,
    country,
    sum(total_sale_amount) as Monthly_revenue,
    EXTRACT(MONTH from transaction_date) as Month_id
from ecommerce_data
GROUP BY continent,country,Month_id
ORDER BY Monthly_revenue DESC

--alter table ecommerce_data
--rename column date to transaction_date

--6. Top Revenue drivers by price
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


--7. Payment failure root cause analysis
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

--8. Month by Month Revenue Growth
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


--9. Inventory report by Quarter
SELECT
    product_name,
    category,
    sum(quantity) as stock_level,
     EXTRACT(quarter from transaction_date) as Quarter
from ecommerce_data
GROUP BY category,product_name,quarter
order by Quarter,stock_level

        
        

        




