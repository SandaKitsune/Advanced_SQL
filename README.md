# Advanced_SQL
Here you can find spreadsheet with advanced SQL code and extracted data
https://docs.google.com/spreadsheets/d/1vSUijkJ312t_cnIqJxZdS0DGb3qp1oQeZi3-6p6-2d4/edit#gid=0

Examples:

1. 
WITH
customer_order_365_ago AS
(SELECT DATE_SUB(MAX(sal_ord_he.OrderDate),INTERVAL 365 DAY) AS validation_date
FROM `adwentureworks_db.salesorderheader` AS sal_ord_he)

SELECT ind.CustomerID,con.Firstname AS First_Name, con.LastName AS Last_Name, CONCAT(con.Firstname, ' ' , con.LastName) AS FullName, CONCAT(COALESCE(con.Title, 'Dear'), ' ', con.LastName) AS addressing_title, con.Emailaddress AS Email, con.Phone, cust.AccountNumber,cust.CustomerType, add.City, add.AddressLine1, 
SUBSTR(add.AddressLine1, 1,4) AS address_no, SUBSTR(add.AddressLine1, 6,30) AS address_st, COALESCE(add.AddressLine2, '') AS AddressLine2, sta_pro.Name AS State,
sale_terr.Name AS Country, COUNT(DISTINCT sal_ord_he.SalesOrderID) AS NumberOfOrders, ROUND(SUM(sal_ord_he.TotalDue),3) AS TotalAmount, MAX(sal_ord_he.OrderDate) AS date_last_order,
CASE WHEN MAX(sal_ord_he.OrderDate) >= (SELECT validation_date FROM customer_order_365_ago) THEN 'active'
ELSE 'inactive'
END AS customer_status
FROM customer_order_365_ago, `adwentureworks_db.individual` ind
JOIN `adwentureworks_db.contact` con ON ind.ContactID = con.ContactID
JOIN `adwentureworks_db.customer` cust ON cust.CustomerID = ind.CustomerID
JOIN `adwentureworks_db.salesorderheader` AS sal_ord_he ON sal_ord_he.CustomerID = cust.CustomerID
JOIN `adwentureworks_db.address` AS add ON sal_ord_he.ShipToAddressID = add.AddressID
JOIN `adwentureworks_db.stateprovince` AS sta_pro ON add.StateProvinceID = sta_pro.StateProvinceID
JOIN `adwentureworks_db.countryregion` AS cou_reg ON sta_pro.CountryRegionCode = cou_reg.CountryRegionCode
JOIN `adwentureworks_db.salesterritory` sale_terr ON cust.TerritoryID = sale_terr.TerritoryID
WHERE cust.CustomerType = 'I' AND sale_terr.Group = 'North America'
GROUP BY ind.CustomerID, First_Name, Last_Name, addressing_title, Email, con.Phone, cust.AccountNumber, cust.CustomerType, add.City, add.AddressLine1, AddressLine2, state, Country
HAVING customer_status = 'active' AND (TotalAmount >= 2500 OR NumberOfOrders >= 5)
ORDER BY Country, State, date_last_order;
