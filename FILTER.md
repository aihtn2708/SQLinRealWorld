

# 1. Finding Products with Low Stock
Identify products with current stock below a certain threshold (e.g., 50 units).

```sql
SELECT p.ProductID, p.ProductName, 
       SUM(CASE WHEN sm.MovementType = 'IN' THEN sm.Quantity ELSE -sm.Quantity END) AS CurrentStock
FROM Products p
JOIN StockMovements sm ON p.ProductID = sm.ProductID
GROUP BY p.ProductID, p.ProductName
HAVING CurrentStock < 50;
```

# 2. Checking Stock Levels by Category
Get the total stock for each product category.

```sql
SELECT c.CategoryID, c.CategoryName, 
       SUM(CASE WHEN sm.MovementType = 'IN' THEN sm.Quantity ELSE -sm.Quantity END) AS TotalStock
FROM Categories c
JOIN Products p ON c.CategoryID = p.CategoryID
JOIN StockMovements sm ON p.ProductID = sm.ProductID
GROUP BY c.CategoryID, c.CategoryName;
```
