# KFC-Sales-Analysis-
Restaurant Sales Analysis 


This project represents the intersection of data analytics, business intelligence and automation — turning raw sales data into actionable insights and proactive alerts for management decision making.
The dashboard doesn’t just show data . It tells the story of KFC’s revenue journey and where it’s headed.(2023-2025 datasets).

After weeks of dedicated work, I’ve built a comprehensive Sales Intelligence System for KFC that combines the power of Power BI and Power Automate to deliver real-time revenue insights and automated alerts.

Power Automate Integration:
Built a fully automated KFC Dynamic Revenue Alert System that:
 ∙ Runs daily at 8:10 AM automatically
 ∙ Queries live Power BI dataset
 ∙ Compares Total Net Sales vs Revenue Target
 ∙ Sends dynamic Outlook email alerts with:
 ∙ Formatted currency values
 ∙ Target met/not met status
 ∙ Real-time timestamp
 ∙ True & False branch logic

Key Business Insights Uncovered:
 ∙Revenue grew 66.7% YoY — exceptional growth
 ∙Dine-in dominates at ₦1.99bn — customers prefer the in-store experience
 ∙Weekends drive peak sales — Saturday & Sunday outperform weekdays by 30%+
 ∙Discount impact is minimal at 0.39% — pricing strategy is healthy
 ∙All 6 regions contribute consistently — balanced national performance
 ∙ Forecast suggests continued growth into 2026-2027

Tools & Technologies Used:
 ∙ Microsoft Power BI Desktop
 ∙ DAX (Data Analysis Expressions)
 ∙ Power Automate
 ∙ Microsoft Outlook
 ∙ Power BI Service



SQL CODE 

````sql
-- Drop the incorrect table
DROP TABLE [Sales fact table dataset].dbo.Fact_Sales_2026_Jan

-- Create table matching your actual CSV columns
CREATE TABLE [Sales fact table dataset].dbo.Fact_Sales_2026_Jan (
    sales_order_id VARCHAR(100),
    order_datetime VARCHAR(100),
    order_date VARCHAR(100),
    store_id VARCHAR(100),
    region VARCHAR(100),
    regional_manager_code VARCHAR(100),
    product_id VARCHAR(100),
    channel VARCHAR(100),
    quantity_sold VARCHAR(100),
    unit_price_ngn VARCHAR(100),
    discount_amount_ngn VARCHAR(100),
    gross_sales_ngn VARCHAR(100),
    net_sales_ngn VARCHAR(100),
    year VARCHAR(100)
   )

BULK INSERT [Sales fact table dataset].dbo.Fact_Sales_2026_Jan
FROM 'C:\Data\Fact_Sales_2026_Jan.csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    TABLOCK
)

SELECT *
FROM dbo.Fact_Sales_2026_Jan

SELECT *
FROM dbo.Fact_Sales_2025

SELECT *
FROM dbo.Fact_Sales_2024


SELECT *
FROM dbo.[Fact_Sales_2023 (3)]

----BUSINESS QUESTIONS---
---What are our overall sales trends from 2023 to  2025?
SELECT  
COUNT(DISTINCT sales_order_id) AS Total_Orders,
SUM(quantity_sold) AS Total_Quantity_Sold,
SUM(gross_sales_ngn) AS Total_Gross_Sales,
SUM(discount_amount_ngn) AS Total_Discount_Amount,
SUM(net_sales_ngn) AS Total_Net_Sales,
AVG(unit_price_ngn) AS Average_Unit_Price,
AVG(discount_amount_ngn) AS Average_Discount_Amount,
FORMAT(SUM(discount_amount_ngn) * 100.0 / NULLIF(SUM(gross_sales_ngn), 0),'P2') AS Average_Discount_Percentage
FROM (
SELECT * FROM dbo.[Fact_Sales_2023 (3)]
UNION ALL 
SELECT * FROM dbo.Fact_Sales_2024
UNION ALL
SELECT * FROM dbo.Fact_Sales_2025
) AS CombinedSales


-----How do KPIs compare year-over-year?
SELECT 
year,
COUNT(DISTINCT sales_order_id) AS Total_Orders,
SUM(quantity_sold) AS Total_Quantity_Sold,
SUM(net_sales_ngn) AS Total_Net_Sales,
AVG(net_sales_ngn) AS Average_Net_Sales,
FORMAT(SUM(discount_amount_ngn ) * 100.0 /NULLIF(SUM(net_sales_ngn),0),'P2') AS Average_Net_Sales_Percentage
FROM (
SELECT * FROM dbo.[Fact_Sales_2023 (3)]
UNION ALL 
SELECT * FROM dbo.Fact_Sales_2024
UNION ALL 
SELECT * FROM dbo.Fact_Sales_2025
) AS ConmbinedSales
GROUP BY year
ORDER BY year;

---What is the year-over-year growth rate in net sales excluding 2026 because it only have January data?
WITH yearly_sales AS (
SELECT year,
       SUM(net_sales_ngn) AS Total_net_sales,
       SUM(quantity_sold) AS Total_Quantity_Sold,
       COUNT(DISTINCT sales_order_id) AS Total_Orders
       FROM
       (SELECT * FROM dbo.[Fact_Sales_2023 (3)]
        UNION ALL
        SELECT * FROM dbo.Fact_Sales_2024
        UNION ALL 
        SELECT *
        FROM dbo.Fact_Sales_2025
        ) AS combinedSales 
        GROUP BY year
        )
SELECT 
       year,
       Total_net_sales,
       Total_Quantity_Sold,
       Total_Orders,
       LAG(Total_net_sales) OVER (ORDER BY year) AS Prev_year_sales,
       CONCAT(CAST((Total_net_sales - LAG(Total_net_sales) OVER (ORDER BY year)) * 100.0 / NULLIF(LAG(Total_net_sales) OVER (ORDER BY year),0)  AS DECIMAL(10,2)),'%') AS Sales_growth_pct,
       CONCAT(CAST((Total_Quantity_Sold - LAG(Total_Quantity_Sold) OVER (ORDER BY year)) * 100.0 / NULLIF(LAG(Total_Quantity_Sold) OVER (ORDER BY year),0) AS DECIMAL(10,2)),'%') AS Quantity_growth_pct,
       CONCAT(CAST((Total_Orders - LAG(Total_Orders) OVER (ORDER BY year)) * 100.0 / NULLIF(LAG(Total_Orders) OVER (ORDER BY year),0) AS DECIMAL(10,2)),'%') AS Order_growth_pct,
       CASE WHEN LAG(Total_net_sales) OVER (ORDER BY year)  IS NULL THEN 'Base Year' ELSE 'YoY_Type' END AS Comparison_Type
       FROM yearly_sales
       ORDER BY year;

     ----Monthly sales trends across all years
     SELECT 
     year,
     MONTH(order_date) AS Order_Month,
     DATENAME(MONTH,order_date) AS Month_Name,
     COUNT(DISTINCT sales_order_id) AS Total_Order,
     SUM(quantity_sold) AS Total_Quantity,
     SUM(net_sales_ngn) AS Total_sales_net,
     AVG(net_sales_ngn) AS Avg_order_Value
     FROM (
     SELECT * FROM dbo.[Fact_Sales_2023 (3)]
     UNION ALL 
     SELECT *
     FROM dbo.Fact_Sales_2024
     UNION ALL 
     SELECT *
     FROM dbo.Fact_Sales_2025
    
    ) CombinedSales
     GROUP BY year, MONTH(order_date),DATENAME(MONTH,order_date)
     ORDER BY year,DATENAME(MONTH,order_date) DESC;


     ---What's the Total Sales by Channel 

    SELECT 
     channel,
      SUM(net_sales_ngn) AS Total_Net_Sales 
      FROM 
      (
      SELECT channel,net_sales_ngn
      FROM
      dbo.[Fact_Sales_2023 (3)]
      UNION ALL
      SELECT channel,net_sales_ngn
      FROM dbo.Fact_Sales_2024
      UNION ALL 
      SELECT channel,net_sales_ngn
      FROM dbo.Fact_Sales_2025) AS CombinedSales
      GROUP BY channel
      ORDER BY Total_Net_Sales DESC;

 ---What's the Total Net Sales trend over month
 SELECT 
 MONTH(Order_date) AS Order_Month,
 DATENAME(MONTH, Order_date) AS Month_Name,
 SUM(net_sales_ngn) AS Total_net_sales
 FROM (
 SELECT *
 FROM dbo.[Fact_Sales_2023 (3)]
 UNION ALL
 SELECT *
 FROM dbo.Fact_Sales_2024
 UNION ALL
 SELECT *
 FROM dbo.Fact_Sales_2025 ) AS CombinedSales
 GROUP BY MONTH(Order_date),DATENAME(MONTH, Order_date)
 ORDER BY SUM(net_sales_ngn) ;
 

 ---Are We Hitting our Target ? 3 years of Truth
WITH SalesBase AS (
    SELECT Order_date,net_sales_ngn FROM dbo.[Fact_Sales_2023 (3)]
    UNION ALL
    SELECT order_date, net_sales_ngn FROM dbo.Fact_Sales_2024
    UNION ALL
    SELECT order_date,net_sales_ngn   FROM dbo.Fact_Sales_2025
),
MonthlySales AS (
    SELECT
        YEAR(order_date)     AS SaleYear,
        MONTH(order_date)    AS SaleMonth,
        SUM(net_sales_ngn) AS TotalNetSales
    FROM SalesBase
    GROUP BY YEAR(order_date), MONTH(order_date)
),
WithTarget AS (
    SELECT
        SaleYear,
        SaleMonth,
        TotalNetSales,
        LAG(TotalNetSales) OVER (
            PARTITION BY SaleMonth
            ORDER BY SaleYear
        ) * 1.1  AS RevenueTarget
    FROM MonthlySales
)

SELECT
    LEFT(DATENAME(MONTH, DATEFROMPARTS(SaleYear, SaleMonth, 1)), 3)
    + ' ' + RIGHT(CAST(SaleYear AS VARCHAR), 2)  AS MonthYearLabel,
    SaleYear,
    SaleMonth,
    TotalNetSales,
   CASE WHEN SaleYear = 2023  THEN 0 ELSE RevenueTarget END AS RevenueTarget
FROM WithTarget
WHERE SaleYear BETWEEN 2023 AND 2025
ORDER BY SaleYear, SaleMonth;

---Where Is Revenue Coming From? The Channel breakdown
SELECT 
Channel,
SUM(discount_amount_ngn) AS Total_discount,
SUM(gross_sales_ngn) AS Total_gross
FROM(
SELECT Channel,discount_amount_ngn,gross_sales_ngn
FROM dbo.[Fact_Sales_2023 (3)]
UNION ALL 
SELECT Channel,discount_amount_ngn,gross_sales_ngn
FROM dbo.Fact_Sales_2024
UNION ALL 
SELECT Channel,discount_amount_ngn,gross_sales_ngn
FROM dbo.Fact_Sales_2025) AS CombinedSales
GROUP BY channel
ORDER BY Total_discount, Total_gross DESC;


SELECT 
region,
SUM(net_sales_ngn) AS Total_Sales 
FROM(
SELECT region,net_sales_ngn 
FROM dbo.[Fact_Sales_2023 (3)]
UNION ALL 
SELECT region,net_sales_ngn
FROM dbo.Fact_Sales_2024
UNION ALL 
SELECT region,net_sales_ngn
FROM dbo.Fact_Sales_2025) CombinedSales 
GROUP BY region
ORDER BY Total_Sales DESC;



SELECT 
DATENAME(WEEKDAY,Order_date) AS Weekname, 
SUM(net_sales_ngn) AS Total_Sales
FROM(
SELECT *
FROM dbo.[Fact_Sales_2023 (3)]
UNION ALL 
SELECT *
FROM dbo.Fact_Sales_2024
UNION ALL 
SELECT *
FROM dbo.Fact_Sales_2025) CombinedSales
GROUP BY DATENAME(WEEKDAY,Order_date) 
ORDER BY Total_Sales DESC;



SELECT 
SUM(gross_sales_ngn) AS Total_gross,
SUM(net_sales_ngn) AS Total_sales,
SUM(discount_amount_ngn) AS Total_discount
FROM (
SELECT *
FROM dbo.[Fact_Sales_2023 (3)]
UNION ALL 
SELECT *
FROM dbo.Fact_Sales_2024
UNION ALL 
SELECT *
FROM dbo.Fact_Sales_2025) AS CombinedSales;






 
