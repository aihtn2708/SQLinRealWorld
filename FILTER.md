# Tables:

## Products Table:

- ProductID: Primary key, auto-incremented.
- ProductName: Name of the product.
- CategoryID: Foreign key referencing the Categories table.
- Unit: Unit of measure for the product.
- Price: Price of the product.
- DateReceived: Date the product was received.

## StockMovements Table:

- MovementID: Primary key, auto-incremented.
- ProductID: Foreign key referencing the Products table.
- MovementType: Type of movement, either 'IN' or 'OUT'.
- Quantity: Quantity of the product moved.
- MovementDate: Date of the movement.

## Categories Table:

- CategoryID: Primary key, auto-incremented.
- CategoryName: Name of the category.

## Relationships:

The Products table references the Categories table via CategoryID.

The StockMovements table references the Products table via ProductID.

# ERD
![image](https://github.com/aihtn2708/SQLinRealWorld/assets/17986030/f1a6c64f-d2cf-496a-9ce0-572f3867f04d)

```sql
Table Products {
    ProductID int [primary key, increment]
    ProductName varchar(255)
    CategoryID int
    Unit varchar(50)
    Price decimal(10,2)
    DateReceived date
}

Table StockMovements {
    MovementID int [primary key, increment]
    ProductID int [ref: > Products.ProductID]
    MovementType varchar(3) // IN or OUT
    Quantity int
    MovementDate date
}

Table Categories {
    CategoryID int [primary key, increment]
    CategoryName varchar(255)
}

Ref: Products.CategoryID > Categories.CategoryID
```

## Product table:
| ProductID | ProductName  | CategoryID | Unit  | Price | DateReceived |
|-----------|--------------|------------|-------|-------|--------------|
| 1         | Laptop       | 1          | pcs   | 1000  | 2023-01-10   |
| 2         | Smartphone   | 1          | pcs   | 700   | 2023-01-15   |
| 3         | Headphones   | 2          | pcs   | 100   | 2023-01-20   |
| 4         | Monitor      | 1          | pcs   | 300   | 2023-02-01   |

## StockMovements Table
| MovementID | ProductID | MovementType | Quantity | MovementDate |
|------------|-----------|--------------|----------|--------------|
| 1          | 1         | IN           | 50       | 2023-01-12   |
| 2          | 1         | OUT          | 10       | 2023-01-20   |
| 3          | 2         | IN           | 100      | 2023-01-18   |
| 4          | 2         | OUT          | 20       | 2023-01-25   |
| 5          | 3         | IN           | 200      | 2023-01-22   |
| 6          | 3         | OUT          | 50       | 2023-02-10   |
| 7          | 4         | IN           | 75       | 2023-02-05   |
| 8          | 4         | OUT          | 5        | 2023-02-15   |

## Categories Table
| CategoryID | CategoryName |
|------------|--------------|
| 1          | Electronics  |
| 2          | Accessories  |

# WHERE - HAVING - ON
## WHERE Clause
Purpose: Used to filter rows before any grouping or aggregation occurs.
When to Use: When you need to filter records based on specified conditions before performing any aggregation (e.g., SUM, COUNT).
Scope: Applies to individual rows in the table.
> Eg: Let's find products that have a price greater than $300.
```sql
SELECT ProductID, ProductName, Price
FROM Products
WHERE Price > 300;
```
| ProductID | ProductName | Price |
|-----------|-------------|-------|
| 1         | Laptop      | 1000  |
| 2         | Smartphone  | 700   |

## HAVING Clause
Purpose: Used to filter groups after aggregation.
When to Use: When you need to filter aggregated results (e.g., groups created by GROUP BY).
Scope: Applies to groups of rows created by GROUP BY.
> Eg: Let's find categories with a total quantity of stock movements greater than 100.
```sql
SELECT c.CategoryID, c.CategoryName, SUM(sm.Quantity) AS TotalQuantity
FROM Categories c
JOIN Products p ON c.CategoryID = p.CategoryID
JOIN StockMovements sm ON p.ProductID = sm.ProductID
GROUP BY c.CategoryID, c.CategoryName
HAVING SUM(sm.Quantity) > 100;
```
| CategoryID | CategoryName | TotalQuantity |
|------------|--------------|---------------|
| 1          | Electronics  | 160           |
| 2          | Accessories  | 150           |

## ON Clause
Purpose: Used to specify join conditions between tables.
When to Use: When you need to define how two tables should be joined.
Scope: Applies to rows being joined between tables.
> Eg: Let's list all products in category Electronics.
```sql
SELECT p.ProductID, p.ProductName, c.CategoryName
FROM Products p
JOIN Categories c ON p.CategoryID = c.CategoryID;
AND c.CategoryName = 'Electronics'
```
| ProductID | ProductName | CategoryID | CategoryName | Unit | Price | DateReceived |
|-----------|-------------|------------|--------------|------|-------|--------------|
| 1         | Laptop      | 1          | Electronics  | pcs  | 1000  | 2023-01-10   |
| 2         | Smartphone  | 1          | Electronics  | pcs  | 700   | 2023-01-15   |
| 4         | Monitor     | 1          | Electronics  | pcs  | 300   | 2023-02-01   |

# USE CASE
Below is the use case of inventory management with examples of different types of filtering WHERE, HAVING, ON.
## 1. Finding Products with Low Stock
Identify products with current stock below a certain threshold (e.g., 50 units).
>Uses a HAVING clause to filter products with a calculated current stock below 50 units.
```sql
SELECT p.ProductID, p.ProductName, 
       SUM(CASE WHEN sm.MovementType = 'IN' THEN sm.Quantity ELSE -sm.Quantity END) AS CurrentStock
FROM Products p
JOIN StockMovements sm ON p.ProductID = sm.ProductID
GROUP BY p.ProductID, p.ProductName
HAVING CurrentStock < 50;
```

## 2. Checking Stock Levels by Category
Get the total stock for each product category.
>Aggregates total stock for each product category using a GROUP BY clause.
```sql
SELECT c.CategoryID, c.CategoryName, 
       SUM(CASE WHEN sm.MovementType = 'IN' THEN sm.Quantity ELSE -sm.Quantity END) AS TotalStock
FROM Categories c
JOIN Products p ON c.CategoryID = p.CategoryID
JOIN StockMovements sm ON p.ProductID = sm.ProductID
GROUP BY c.CategoryID, c.CategoryName;
```

## 3. Identifying Fast-Moving Products
Find products with the highest quantity sold in the last 30 days.
>Filters and orders products by total quantity sold in the last 30 days.
```sql
SELECT p.ProductID, p.ProductName, 
       SUM(CASE WHEN sm.MovementType = 'OUT' THEN sm.Quantity ELSE 0 END) AS TotalSold
FROM Products p
JOIN StockMovements sm ON p.ProductID = sm.ProductID
WHERE sm.MovementDate >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
GROUP BY p.ProductID, p.ProductName
ORDER BY TotalSold DESC
LIMIT 10;
```

## 4. Analyzing Stock Aging
Identify products that have been in stock for more than 180 days and their remaining quantities.
>Identifies products in stock for more than 180 days, ensuring they have a positive stock.
```sql
SELECT p.ProductID, p.ProductName, p.DateReceived, 
       DATEDIFF(CURDATE(), p.DateReceived) AS DaysInStock,
       SUM(CASE WHEN sm.MovementType = 'IN' THEN sm.Quantity ELSE -sm.Quantity END) AS CurrentStock
FROM Products p
JOIN StockMovements sm ON p.ProductID = sm.ProductID
WHERE DATEDIFF(CURDATE(), p.DateReceived) > 180
GROUP BY p.ProductID, p.ProductName, p.DateReceived
HAVING CurrentStock > 0;
```

## 5. Monitoring Restocking Needs
Identify products that need to be restocked based on minimum stock levels.
>Filters products that need to be restocked based on a minimum stock level threshold.
```sql
SELECT p.ProductID, p.ProductName, 
       SUM(CASE WHEN sm.MovementType = 'IN' THEN sm.Quantity ELSE -sm.Quantity END) AS CurrentStock
FROM Products p
JOIN StockMovements sm ON p.ProductID = sm.ProductID
GROUP BY p.ProductID, p.ProductName
HAVING CurrentStock < 100; -- Assume 100 units as the minimum stock level
```
