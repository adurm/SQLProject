

# Adam Moussa SQL Practical Exercise

## Exercise 1 – Northwind Queries (40 marks: 5 for each question)

#### 1.1	Write a query that lists all Customers in either Paris or London. Include Customer ID, Company Name and all address fields.
```SQL
/* 1.1 */
SELECT CustomerID, CompanyName, Address, City, PostalCode, Country -- All address fields
  FROM Customers 
 WHERE City 
    IN ('Paris', 'London')
```

#### 1.2	List all products stored in bottles.
```SQL
/* 1.2 */
SELECT *
  FROM Products
 WHERE QuantityPerUnit 
  LIKE '%bottle%' -- Search for the mention of 'bottle' in QuantityPerUnit
```

#### 1.3	Repeat question above, but add in the Supplier Name and Country.
```SQL
/* 1.3 */
SELECT p.*, s.CompanyName AS 'Supplier Name', s.Country 
  FROM Products AS p
       INNER JOIN Suppliers AS s
       ON p.SupplierID = s.SupplierID
 WHERE QuantityPerUnit 
  LIKE '%bottle%' -- Search for the mention of 'bottle' in QuantityPerUnit
```

#### 1.4	Write an SQL Statement that shows how many products there are in each category. Include Category Name in result set and list the highest number first.
```SQL
/* 1.4 */
SELECT p.CategoryID, c.CategoryName, 
       COUNT(*) AS 'Different Products',  -- Count the number of products that share each CategoryID
       SUM(p.UnitsInStock) AS 'Stock Available' -- Sum the UnitsInStock for each unique CategoryID
  FROM Products AS p
       INNER JOIN Categories AS c
       ON p.categoryID = c.categoryID
 GROUP BY p.CategoryID, c.CategoryName -- Group all unique CategoryIDs
 ORDER BY [Different Products] DESC -- Show the category with the most amount of products first
```

#### 1.5	List all UK employees using concatenation to join their title of courtesy, first name and last name together. Also include their city of residence.
```SQL
/* 1.5 */
SELECT CONCAT(TitleOfCourtesy, ' ', FirstName, ' ', LastName) AS 'Employee',
       City AS 'City of Residence'
  FROM Employees
 WHERE Country = 'UK'
```

#### 1.6	List Sales Totals for all Sales Regions (via the Territories table using 4 joins) with a Sales Total greater than 1,000,000. Use rounding or FORMAT to present the numbers. 
```SQL
/* 1.6 */
SELECT r.RegionID, r.RegionDescription, 
       /* Calculate how much a region has made and format the data to make it easier to read*/
       FORMAT(ROUND(SUM(od.UnitPrice * od.Quantity * (1-od.Discount)), 0), '#,###,###') AS 'Sales Total'
  FROM Territories AS t
       INNER JOIN EmployeeTerritories AS et
       ON t.TerritoryID = et.TerritoryID

       INNER JOIN Region AS r
       ON t.RegionID = r.RegionID
                            
       INNER JOIN Orders AS o
       ON et.EmployeeID = o.EmployeeID
       
       INNER JOIN [Order Details] AS od
       ON o.OrderID = od.OrderID
 GROUP BY r.RegionID, r.RegionDescription 
HAVING ROUND(SUM(od.UnitPrice * od.Quantity * (1-od.Discount)), 0) > 1000000
 ORDER BY [Sales Total] DESC
```

#### 1.7	Count how many Orders have a Freight amount greater than 100.00 and either USA or UK as Ship Country.
```SQL
/* 1.7 */
SELECT COUNT(*) AS 'Orders'
  FROM Orders 
 WHERE Freight > 100.00 
   AND ShipCountry IN ('USA', 'UK')
```

#### 1.8	Write an SQL Statement to identify the Order Number of the Order with the highest amount of discount applied to that order.
```SQL
/* 1.8 */
SELECT OrderID, (Quantity * UnitPrice * Discount) AS 'Discount On Order'
  FROM [Order Details]
 WHERE (Quantity * UnitPrice * Discount) = 
       (SELECT MAX(Quantity * UnitPrice * Discount) 
          FROM [Order Details])
 GROUP BY OrderID, (Quantity * UnitPrice * Discount)
 ORDER BY [Discount On Order] DESC
```
## Exercise 2 – Create Spartans Table (20 marks – 10 each)


## Exercise 3 – Northwind Data Analysis linked to Excel (30 marks)

#### 3.1 List all Employees from the Employees table and who they report to. No Excel required. (5 Marks)
```SQL
/* 3.1 */
SELECT e.EmployeeID, CONCAT(e.FirstName, ' ', e.LastName) AS 'Employee',
       CONCAT(s.FirstName, ' ', s.LastName) AS 'ReportsTo' 
  FROM Employees AS e 
       LEFT JOIN Employees s 
       ON s.EmployeeID = e.ReportsTo
```

#### 3.2 List all Suppliers with total sales over $10,000 in the Order Details table. Include the Company Name from the Suppliers Table and present as a bar chart as below: (5 Marks)
```SQL
/* 3.2 */
SELECT od.UnitPrice * od.Quantity AS 'sales', 
       CompanyName AS 'Supplier name' 
  FROM Suppliers s
       INNER JOIN products p 
       ON s.supplierid = p.supplierid

       INNER JOIN [Order Details] od 
       ON p.ProductID = od.ProductID
 GROUP BY (od.UnitPrice * od.Quantity), CompanyName 
HAVING od.UnitPrice * od.Quantity > 10000
```

#### 3.3 List the Top 10 Customers YTD for the latest year in the Orders file. Based on total value of orders shipped. No Excel required. (10 Marks)
```SQL
/* 3.3 */
SELECT TOP 10 o.CustomerID, c.CompanyName,
       ROUND(SUM(od.UnitPrice*od.Quantity*(1-od.Discount)),0) AS 'Total Value'
  FROM ((Orders o
       INNER JOIN [Order Details] od 
       ON o.OrderID = od.OrderID)
       
       INNER JOIN Customers c 
       ON c.CustomerID=o.CustomerID)
 GROUP BY o.CustomerID, c.CompanyName, YEAR(ShippedDate)
 HAVING YEAR(ShippedDate) = (SELECT YEAR(MAX(ShippedDate)) FROM Orders)
 ORDER BY [Total Value] DESC
```

#### 3.4 Plot the Average Ship Time by month for all data in the Orders Table using a line chart as below. (10 Marks)
```SQL
 /* 3.4 */
SELECT MONTH(ShippedDate) AS 'Month',
       /* Difference in days between data ordered and date shipped */
       AVG(DATEDIFF(day, OrderDate, ShippedDate)) AS 'Avg Ship Time (Days)'
  FROM Orders
 GROUP BY MONTH(ShippedDate) 
HAVING MONTH(ShippedDate) IS NOT NULL -- Don't include orders that havent been shipped
 ORDER BY [Month]
```
 
