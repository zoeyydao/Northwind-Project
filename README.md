# Northwind Traders: SQL Data Analysis Case Study


## Project Overview

Northwind Traders is a global specialty food import/export company that purchases gourmet food products from suppliers worldwide and sells them to retailers in various countries. The company has experienced growth over the past few years but faces increasing competition in the specialty food market.

This project involves a comprehensive deep dive into the company’s relational database to uncover actionable insights across five key business domains: Customer Behavior, Product Performance, Sales Trends, Employee Efficiency, and Supplier & Inventory Management.

## Technical Stack

**Language:** SQL (SQLite)

**Key Techniques Used:**
- Advanced Aggregations & Joins  
- Common Table Expressions (CTEs)  
- Window Functions  
- Conditional Logic  

## Dataset

This diagram presents the Entity Relationship Diagram (ERD) for the dataset.

> Insert ERD image here  
> Example: `![Northwind ERD](images/northwind-erd.png)`



## Business Questions

This project explores a series of key business questions designed to extract meaningful insights from the Northwind database and support data-driven decision-making.

The analysis covers multiple business dimensions, including customers, products, sales performance, employees, and supply chain operations. Each section addresses specific business questions, accompanied by SQL queries used to derive the results.



## Section 1: Customer Analysis

**1. Customer Segmentation: Who are Northwind's top 10 customers by total purchase amount?**

SQL Query:

```sql
SELECT c.CompanyName, 
       c.ContactName,
       c.Country,
       SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalPurchaseAmount
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN [Order Details] od ON o.OrderID = od.OrderID
GROUP BY c.CompanyName, c.ContactName, c.Country
ORDER BY TotalPurchaseAmount DESC
LIMIT 10;
``` 

Result:
![Insert Image Here](link-to-your-image.png)

**2. Geographic Distribution: Which countries generate the most revenue for Northwind?**

SQL Query:

```sql
SELECT c.Country,
       SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalSales
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN [Order Details] od ON o.OrderID = od.OrderID
GROUP BY c.Country
ORDER BY TotalSales DESC
LIMIT 10;
``` 

Result:
![Insert Image Here](link-to-your-image.png)

**3. Customer Loyalty: Identify customers who have been ordering consistently over time (orders in at least 3 different quarters). Return their company name and the number of quarters they've placed orders in.**

SQL Query:

```sql
SELECT C.CompanyName, 
       COUNT(DISTINCT (strftime('%Y', OrderDate) || '-' || ((strftime('%m', OrderDate) - 1) / 3))) AS NumberOfQuarters,
       DATE(MIN(OrderDate)) AS FirstOrderDate,
       DATE(MAX(OrderDate)) AS MostRecentOrderDate
FROM Orders O
JOIN Customers C ON O.CustomerID = C.CustomerID
GROUP BY C.CompanyName
HAVING NumberOfQuarters >= 3
ORDER BY NumberOfQuarters DESC;
```
Result:
![Insert Image Here](link-to-your-image.png)

## Section 2: Product Performance Analysis

**4. Product Revenue Analysis: Which 5 products generate the most revenue?**

SQL Query:

```sql
SELECT 
    p.ProductName, 
    p.UnitPrice,
    SUM(od.Quantity) AS TotalQuantitySold,
    SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalRevenue
FROM Products p
JOIN [Order Details] od ON p.ProductID = od.ProductID
GROUP BY p.ProductName, p.UnitPrice
ORDER BY TotalRevenue DESC
LIMIT 5;
```
Result:
![Insert Image Here](link-to-your-image.png)

**5. Seasonal Trends: Do certain products show seasonal sales patterns? Write a query that analyses quarterly sales by product category over the past two years.**

SQL Query:

```sql
SELECT cat.CategoryName, 
    SUM(CASE WHEN strftime('%Y', o.OrderDate) = '2022' 
              AND ((CAST(strftime('%m', o.OrderDate) AS INTEGER)-1)/3 +1) = 1
        THEN od.UnitPrice * od.Quantity * (1 - od.Discount) END) AS Q1_2022,

    SUM(CASE WHEN strftime('%Y', o.OrderDate) = '2022' 
              AND ((CAST(strftime('%m', o.OrderDate) AS INTEGER)-1)/3 +1) = 2
        THEN od.UnitPrice * od.Quantity * (1 - od.Discount) END) AS Q2_2022,

    SUM(CASE WHEN strftime('%Y', o.OrderDate) = '2022' 
              AND ((CAST(strftime('%m', o.OrderDate) AS INTEGER)-1)/3 +1) = 3
        THEN od.UnitPrice * od.Quantity * (1 - od.Discount) END) AS Q3_2022,

    SUM(CASE WHEN strftime('%Y', o.OrderDate) = '2022' 
              AND ((CAST(strftime('%m', o.OrderDate) AS INTEGER)-1)/3 +1) = 4
        THEN od.UnitPrice * od.Quantity * (1 - od.Discount) END) AS Q4_2022,

    SUM(CASE WHEN strftime('%Y', o.OrderDate) = '2023' 
              AND ((CAST(strftime('%m', o.OrderDate) AS INTEGER)-1)/3 +1) = 1
        THEN od.UnitPrice * od.Quantity * (1 - od.Discount) END) AS Q1_2023,

    SUM(CASE WHEN strftime('%Y', o.OrderDate) = '2023' 
              AND ((CAST(strftime('%m', o.OrderDate) AS INTEGER)-1)/3 +1) = 2
        THEN od.UnitPrice * od.Quantity * (1 - od.Discount) END) AS Q2_2023

FROM Orders o
JOIN [Order Details] od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID
JOIN Categories cat ON p.CategoryID = cat.CategoryID
GROUP BY cat.CategoryName
ORDER BY cat.CategoryName;
```
Result:
![Insert Image Here](link-to-your-image.png)

## Section 3: Sales Performance Analysis

**6. Time-based Analysis: How have monthly sales trended over time? Write a query showing total monthly sales for the past two years.**

SQL Query:

```sql
SELECT 
    strftime('%Y-%m', o.OrderDate) AS YearMonth,
    ROUND(SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)), 2) AS TotalSales,
    COUNT(DISTINCT o.OrderID) AS OrderCount,
    ROUND(
        SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) 
        / COUNT(DISTINCT o.OrderID), 
    2) AS AverageOrderValue
FROM Orders o
JOIN [Order Details] od ON o.OrderID = od.OrderID
WHERE o.OrderDate BETWEEN '2022-01-01' AND '2023-12-31'
GROUP BY YearMonth
ORDER BY YearMonth DESC;
```
Result:
![Insert Image Here](link-to-your-image.png)

**7. Discount Impact: How do discounts affect order size? Compare the average order value for orders with and without discounts.**

SQL Query:

```sql
WITH OrderSummary AS (
    SELECT o.OrderID,
           SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS OrderValue,
           AVG(od.Quantity) AS AvgQuantity,
           AVG(od.Discount) AS AvgDiscountRate
    FROM Orders o
    JOIN [Order Details] od ON o.OrderID = od.OrderID
    GROUP BY o.OrderID
)

SELECT 
    CASE 
        WHEN AvgDiscountRate = 0 THEN 'No discount'
        WHEN AvgDiscountRate <= 0.05 THEN 'Small Discount (1-5%)'
        WHEN AvgDiscountRate <= 0.15 THEN 'Medium Discount (6-15%)'
        ELSE 'Large Discount (>15%)'
    END AS DiscountLevel,
    
    COUNT(*) AS OrderCount,
    ROUND(AVG(AvgQuantity), 2) AS AvgQuantityPerOrderLine,
    ROUND(AVG(OrderValue), 2) AS AvgOrderValue,
    ROUND(AVG(AvgDiscountRate), 4) AS AvgDiscountRate

FROM OrderSummary
GROUP BY 1
ORDER BY AvgDiscountRate;
```
Result:
![Insert Image Here](link-to-your-image.png)

**8. Shipping Analysis: Which shipping company delivers orders most quickly? Calculate the average days between shipping date and required date for each shipper.**

SQL Query:

```sql
SELECT 
    S.CompanyName AS ShipperName,
    COUNT(O.OrderID) AS TotalOrders,
    ROUND(AVG(julianday(O.ShippedDate) - julianday(O.OrderDate)), 2) AS AvgDaysToShip,
    ROUND(AVG(julianday(O.RequiredDate) - julianday(O.ShippedDate)), 2) AS AvgDaysToRequired,
    ROUND(
        SUM(CASE WHEN O.ShippedDate <= O.RequiredDate THEN 1 ELSE 0 END) * 100.0 
        / COUNT(O.OrderID), 
    2) || '%' AS OnTimeDeliveryRate
FROM Shippers S
JOIN Orders O ON S.ShipperID = O.ShipVia
WHERE O.ShippedDate IS NOT NULL
GROUP BY S.CompanyName
ORDER BY AvgDaysToRequired DESC;
```
Result:
![Insert Image Here](link-to-your-image.png)

## Section 4: Employee Performance Analysis

**9. Sales by Employee: Who are the top-performing sales representatives based on total revenue generated? Create a ranking of employees by their total sales.**

SQL Query:

```sql
WITH EmployeeSales AS (
    SELECT 
        e.EmployeeID,
        e.FirstName || ' ' || e.LastName AS EmployeeName,
        COUNT(DISTINCT o.OrderID) AS TotalOrders,
        SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalSales,
        AVG(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS AvgSaleValue
    FROM Employees e
    JOIN Orders o 
        ON e.EmployeeID = o.EmployeeID
    JOIN [Order Details] od 
        ON o.OrderID = od.OrderID
    GROUP BY e.EmployeeID, EmployeeName
)

SELECT EmployeeID, EmployeeName, TotalOrders, 
       ROUND(TotalSales, 2) AS TotalSales, 
       ROUND(AvgSaleValue, 2) AS AvgSaleValue,
       RANK() OVER (ORDER BY TotalSales DESC) AS SalesRank
FROM EmployeeSales
ORDER BY SalesRank;
```
Result:
![Insert Image Here](link-to-your-image.png)

**10. Territory Performance: Which sales territories are performing best? For each territory, calculate the total sales, number of orders, and average order value.**

SQL Query:

```sql
SELECT t.TerritoryDescription,
    COUNT(DISTINCT o.OrderID) AS TotalOrders,
    COUNT(DISTINCT o.CustomerID) AS TotalCustomers,
    SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalSales,
    ROUND(
        SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) 
        / COUNT(DISTINCT o.OrderID), 
    2) AS AvgOrderValue
FROM Territories t
JOIN EmployeeTerritories et ON t.TerritoryID = et.TerritoryID
JOIN Employees e ON et.EmployeeID = e.EmployeeID
JOIN Orders o ON e.EmployeeID = o.EmployeeID
JOIN [Order Details] od ON o.OrderID = od.OrderID
GROUP BY t.TerritoryDescription
ORDER BY TotalOrders DESC;
```
Result:
![Insert Image Here](link-to-your-image.png)

**11. Products Needing Reorder: Which products need to be reordered based on their current stock levels?**

SQL Query:

```sql
SELECT 
     ProductID, 
     ProductName, 
     UnitsInStock, 
     ReorderLevel, 
     Discontinued
FROM Products
WHERE UnitsInStock <= ReorderLevel AND Discontinued = 0
ORDER BY UnitsInStock ASC;
```
Result:
![Insert Image Here](link-to-your-image.png)

**12. Suppliers with Most Products: Which suppliers provide the most products to Northwind, and how many discontinued products do they have?**

SQL Query:

```sql
SELECT 
    s.SupplierID, 
    s.CompanyName AS SupplierName, 
    s.Country,
    COUNT(p.ProductID) AS TotalProducts,
    SUM(CASE WHEN p.Discontinued = 1 THEN 1 ELSE 0 END) AS DiscontinuedProducts
FROM Suppliers s
LEFT JOIN Products p 
    ON s.SupplierID = p.SupplierID
GROUP BY s.SupplierID, s.CompanyName, s.Country
ORDER BY TotalProducts DESC;
```
Result:
![Insert Image Here](link-to-your-image.png)

**13. Most Expensive Products by Category: For each category, what is the most expensive product?**

SQL Query:

```sql
WITH RankedProducts AS (
    SELECT 
        C.CategoryID,
        C.CategoryName,
        P.ProductID,
        P.ProductName,
        P.UnitPrice,
        RANK() OVER (PARTITION BY C.CategoryID ORDER BY P.UnitPrice DESC) AS PriceRank
    FROM Categories C
    JOIN Products P ON C.CategoryID = P.CategoryID
)
SELECT 
    CategoryID, 
    CategoryName, 
    ProductID, 
    ProductName, 
    UnitPrice
FROM RankedProducts
WHERE PriceRank = 1;
```
Result:
![Insert Image Here](link-to-your-image.png)

**14. Products with Higher Than Average Price: Which products have a price higher than the average price in their category?**

SQL Query:

```sql
WITH CategoryPriceAnalysis AS (
    SELECT 
        p.ProductID,
        p.ProductName,
        p.UnitPrice,
        c.CategoryName,
        ROUND(AVG(p.UnitPrice) OVER (PARTITION BY c.CategoryName), 2) AS CategoryAvgPrice
    FROM Products p
    JOIN Categories c ON p.CategoryID = c.CategoryID
)
SELECT 
    ProductID,
    ProductName,
    UnitPrice,
    CategoryName,
    CategoryAvgPrice
FROM CategoryPriceAnalysis
WHERE UnitPrice > CategoryAvgPrice
ORDER BY CategoryName, UnitPrice DESC;
```
Result:
![Insert Image Here](link-to-your-image.png)

