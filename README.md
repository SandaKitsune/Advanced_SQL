# Advanced_SQL
Here you can find spreadsheet with advanced SQL code and extracted data
https://docs.google.com/spreadsheets/d/1vSUijkJ312t_cnIqJxZdS0DGb3qp1oQeZi3-6p6-2d4/edit#gid=0

Examples:

1. 
WITH sales_data AS (

SELECT LAST_DAY((CAST(sal_ord_he.OrderDate as date)),month) AS order_month,sal_ter.CountryRegionCode AS CountryRegionCode, sal_ter.Name AS Region, COUNT(DISTINCT sal_ord_he.SalesOrderID) AS number_orders, COUNT(DISTINCT sal_ord_he.CustomerID) AS number_customers, COUNT(DISTINCT sal_ord_he.SalesPersonID) AS no_salesPersons, 

ROUND(SUM(sal_ord_he.TotalDue)) AS Total_w_tax

FROM `adwentureworks_db.salesorderheader` sal_ord_he

JOIN `adwentureworks_db.salesterritory` sal_ter ON sal_ter.TerritoryID = sal_ord_he.TerritoryID

GROUP BY order_month, sal_ter.CountryRegionCode,Region
)
SELECT order_month, CountryRegionCode, Region, number_orders, number_customers, no_salesPersons, Total_w_tax, SUM(Total_w_tax) OVER (PARTITION BY CountryRegionCode, Region 

ORDER BY order_month

) AS Cumulative_Total_w_tax

FROM sales_data
ORDER BY CountryRegionCode,Region, order_month;
