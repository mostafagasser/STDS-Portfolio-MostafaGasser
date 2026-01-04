# Sales Analysis – SQL Queries (AdventureWorks)

## Database Used

**AdventureWorks**

---

## 1️⃣ General KPIs

```sql
-- Total Orders, Revenue, Avg Order Value, Total Customers
SELECT
    COUNT(DISTINCT SalesOrderID) AS TotalOrders,
    SUM(TotalDue) AS TotalRevenue,
    AVG(TotalDue) AS AvgOrderValue,
    COUNT(DISTINCT CustomerID) AS TotalCustomers
FROM Sales.SalesOrderHeader;
```

---

## 2️⃣ Time-Based Analysis

### Daily Summary

```sql
SELECT 
    CONVERT(date, OrderDate) AS OrderDay, 
    COUNT(*) AS TotalOrders, 
    SUM(TotalDue) AS TotalRevenue
FROM Sales.SalesOrderHeader
GROUP BY CONVERT(date, OrderDate)
ORDER BY OrderDay;
```

### Monthly Summary

```sql
SELECT 
    YEAR(OrderDate) AS [Year], 
    MONTH(OrderDate) AS [Month],
    COUNT(*) AS Orders, 
    SUM(TotalDue) AS Revenue
FROM Sales.SalesOrderHeader
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
ORDER BY [Year],[Month];
```

### Yearly Summary

```sql
SELECT 
    YEAR(OrderDate) AS [Year], 
    COUNT(*) AS Orders, 
    SUM(TotalDue) AS Revenue
FROM Sales.SalesOrderHeader
GROUP BY YEAR(OrderDate)
ORDER BY [Year];
```

---

## 3️⃣ Product Performance

### Top 20 Products by Revenue

```sql
SELECT TOP 20
    p.ProductID, 
    p.Name AS Product,
    SUM(sd.LineTotal) AS Revenue,
    SUM(sd.OrderQty) AS QuantitySold
FROM Sales.SalesOrderDetail sd
JOIN Production.Product p ON sd.ProductID = p.ProductID
GROUP BY p.ProductID, p.Name
ORDER BY Revenue DESC;
```

### Top 50 Products by Estimated Profit

```sql
SELECT TOP 50
    p.ProductID, 
    p.Name AS Product,
    SUM(sd.LineTotal) AS Revenue,
    SUM(p.StandardCost * sd.OrderQty) AS EstimatedCost,
    SUM(sd.LineTotal) - SUM(p.StandardCost * sd.OrderQty) AS EstimatedProfit
FROM Sales.SalesOrderDetail sd
JOIN Production.Product p ON sd.ProductID = p.ProductID
GROUP BY p.ProductID, p.Name
ORDER BY EstimatedProfit DESC;
```

---

## 4️⃣ Category & Subcategory Analysis

```sql
SELECT 
    pc.Name AS Category,
    psc.Name AS Subcategory,
    SUM(sd.LineTotal) AS Revenue,
    SUM(sd.OrderQty) AS QuantitySold
FROM Sales.SalesOrderDetail sd
JOIN Production.Product p ON sd.ProductID = p.ProductID
LEFT JOIN Production.ProductSubcategory psc ON p.ProductSubcategoryID = psc.ProductSubcategoryID
LEFT JOIN Production.ProductCategory pc ON psc.ProductCategoryID = pc.ProductCategoryID
GROUP BY pc.Name, psc.Name
ORDER BY Revenue DESC;
```

---

## 5️⃣ Discount & Pricing Analysis

```sql
SELECT 
    p.Name AS Product,
    AVG(sd.UnitPriceDiscount) AS AvgDiscount,
    SUM(sd.LineTotal) AS RevenueAfterDiscount,
    SUM(sd.OrderQty * p.ListPrice) AS GrossRevenue,
    SUM(sd.OrderQty * p.ListPrice) - SUM(sd.LineTotal) AS DiscountLoss
FROM Sales.SalesOrderDetail sd
JOIN Production.Product p ON sd.ProductID = p.ProductID
GROUP BY p.Name
ORDER BY AvgDiscount DESC;
```

---

## 6️⃣ Low Selling Products

```sql
SELECT 
    p.ProductID, 
    p.Name AS Product,
    ISNULL(SUM(sd.OrderQty), 0) AS TotalQtySold,
    COUNT(DISTINCT sd.SalesOrderID) AS OrdersCount
FROM Production.Product p
LEFT JOIN Sales.SalesOrderDetail sd ON p.ProductID = sd.ProductID
GROUP BY p.ProductID, p.Name
HAVING ISNULL(SUM(sd.OrderQty), 0) <= 5
ORDER BY TotalQtySold ASC;
```

---

## 7️⃣ Territory Performance

```sql
SELECT 
    st.Name AS Territory,
    SUM(soh.TotalDue) AS TotalRevenue,
    COUNT(soh.SalesOrderID) AS OrdersCount,
    COUNT(DISTINCT soh.CustomerID) AS UniqueCustomers
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesTerritory st ON soh.TerritoryID = st.TerritoryID
GROUP BY st.Name
ORDER BY TotalRevenue DESC;
```

---

## 8️⃣ Salesperson Performance

```sql
SELECT 
    sp.BusinessEntityID AS SalesPersonID,
    p.FirstName + ' ' + p.LastName AS SalesPersonName,
    SUM(soh.TotalDue) AS TotalSales,
    COUNT(soh.SalesOrderID) AS OrdersHandled
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesPerson sp ON soh.SalesPersonID = sp.BusinessEntityID
JOIN Person.Person p ON sp.BusinessEntityID = p.BusinessEntityID
GROUP BY sp.BusinessEntityID, p.FirstName, p.LastName
ORDER BY TotalSales DESC;
```

---

## 9️⃣ Profitability by Region

```sql
SELECT 
    st.Name AS Territory,
    SUM(sd.LineTotal) AS Revenue,
    SUM(p.StandardCost * sd.OrderQty) AS Cost,
    SUM(sd.LineTotal) - SUM(p.StandardCost * sd.OrderQty) AS Profit
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesOrderDetail sd ON soh.SalesOrderID = sd.SalesOrderID
JOIN Production.Product p ON sd.ProductID = p.ProductID
JOIN Sales.SalesTerritory st ON soh.TerritoryID = st.TerritoryID
GROUP BY st.Name
ORDER BY Profit DESC;
```

---

## 10️⃣ Customer Insights

```sql
SELECT 
    c.CustomerID,
    p.FirstName + ' ' + p.LastName AS CustomerName,
    COUNT(soh.SalesOrderID) AS OrdersCount,
    SUM(soh.TotalDue) AS TotalSpent,
    AVG(soh.TotalDue) AS AvgOrderValue
FROM Sales.Customer c
JOIN Sales.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
JOIN Person.Person p ON c.PersonID = p.BusinessEntityID
GROUP BY c.CustomerID, p.FirstName, p.LastName
ORDER BY TotalSpent DESC;
```

---

## 11️⃣ Product Sales Trend

```sql
SELECT 
    p.Name AS Product,
    FORMAT(soh.OrderDate, 'yyyy-MM') AS SalesMonth,
    SUM(sd.LineTotal) AS MonthlyRevenue
FROM Sales.SalesOrderHeader soh
JOIN Sales.SalesOrderDetail sd ON soh.SalesOrderID = sd.SalesOrderID
JOIN Production.Product p ON sd.ProductID = p.ProductID
GROUP BY p.Name, FORMAT(soh.OrderDate, 'yyyy-MM')
ORDER BY p.Name, SalesMonth;
```

---

## 12️⃣ View for Dashboard

```sql
-- DROP VIEW IF EXISTS vw_SalesAnalysis;
-- CREATE VIEW SalesAnalysisView AS
SELECT 
    soh.SalesOrderID,
    soh.OrderDate,
    soh.DueDate,
    soh.ShipDate,
    soh.SubTotal,
    soh.TaxAmt,
    soh.Freight,
    soh.TotalDue,
    st.Name AS Territory,
    p.ProductID,
    p.Name AS ProductName,
    p.ProductNumber,
    p.ListPrice,
    p.StandardCost,
    sd.OrderQty,
    sd.UnitPrice,
    sd.UnitPriceDiscount,
    sd.LineTotal,
    c.CustomerID,
    per.FirstName + ' ' + per.LastName AS CustomerName
FROM Sales.SalesOrderHeader AS soh
JOIN Sales.SalesOrderDetail AS sd 
    ON soh.SalesOrderID = sd.SalesOrderID
JOIN Production.Product AS p 
    ON sd.ProductID = p.ProductID
JOIN Sales.Customer AS c 
    ON soh.CustomerID = c.CustomerID
```
