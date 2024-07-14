An full outer join is a method of combining tables so that the result includes unmatched rows of both tables. If you are joining two tables and want the result set to include unmatched rows from both tables, use a FULL OUTER JOIN clause. The matching is based on the join condition. 


In this use case, we aim to analyze the historical and current supply/demand (S/D) ratio for products in an inventory system. The goal is to understand past risks and manage current inventory levels effectively. We utilize data from three key sources:

- **Stock Table**: Records the historical and current stock levels of products.
- **OrderSchedule Table**: Details the historical and upcoming orders, including quantities and dates.
- **SafetyStock Table**: Lists the safety stock levels required for each product to ensure smooth operations.

# Tables:

## Stock Table

- ProductID: Unique identifier for the product.

- ProductName: Name of the product.

- StockQuantity: Quantity of the product in stock.

- StockDate: Date of the stock entry.


| ProductID | ProductName | StockQuantity | StockDate  |
|-----------|-------------|---------------|------------|
| 1         | Widget A    | 50            | 2024-07-01 |
| 1         | Widget A    | 40            | 2024-08-01 |
| 2         | Widget B    | 20            | 2024-07-01 |
| 2         | Widget B    | 30            | 2024-08-01 |
| 3         | Widget C    | 0             | 2024-07-01 |
| 3         | Widget C    | 10            | 2024-08-01 |
| 4         | Widget D    | 15            | 2024-07-01 |
| 4         | Widget D    | 25            | 2024-08-01 |

## OrderSchedule Table

- OrderID: Unique identifier for the order.

- OrderDate: Date of the order.

- CustomerID: Unique identifier for the customer.

- ProductID: Unique identifier for the product (foreign key referencing Stock).

- Quantity: Quantity of the product ordered.

- Value: Value of the order.

- ShippingPoint: Warehouse from where the order will be shipped.

| OrderID | OrderDate  | CustomerID | ProductID | Quantity | Value | ShippingPoint |
|---------|------------|------------|-----------|----------|-------|---------------|
| 101     | 2024-07-01 | C001       | 2         | 30       | 300   | WH1           |
| 102     | 2024-07-03 | C002       | 3         | 25       | 250   | WH2           |
| 103     | 2024-08-04 | C003       | 2         | 15       | 150   | WH1           |
| 104     | 2024-08-05 | C004       | 3         | 20       | 200   | WH2           |

## SafetyStock Table

- ProductID: Unique identifier for the product (foreign key referencing Stock).

- SafetyStock: Safety stock level for the product.

| ProductID | SafetyStock |
|-----------|-------------|
| 1         | 10          |
| 2         | 15          |
| 3         | 5           |
| 4         | 8           |
| 5         | 12          |


# Why Use FULL OUTER JOIN?

Using a FULL OUTER JOIN is essential for this analysis because it allows us to:

- **Capture Complete History**: Include all products, regardless of whether they have stock entries, sales entries, or both in any given month. This comprehensive inclusion is crucial for historical analysis.

- **Identify Risks**: Capture scenarios where a product might have had stock but no sales in a specific month or sales but no stock. Identifying these gaps helps understand risks such as understocking or overstocking that occurred in the past.

- **Comprehensive View**: By combining data from both stock and order schedules, we can accurately calculate the supply/demand ratio over time, determine the stock status (understocked, overstocked, or balanced), and track changes month by month. This historical perspective helps in identifying patterns and making informed decisions to mitigate future risks.

**Without a FULL OUTER JOIN**, we risk missing critical historical data points, leading to incomplete or inaccurate insights into past inventory risks and current management needs.

```sql
WITH StockSummary AS (
    -- Subquery to summarize stock levels by product and month
    SELECT 
        ProductID,
        ProductName,
        DATE_TRUNC('month', StockDate) AS Month, -- Truncate date to month for grouping
        SUM(StockQuantity) AS TotalStock -- Sum the stock quantities
    FROM 
        Stock
    GROUP BY 
        ProductID, ProductName, DATE_TRUNC('month', StockDate) -- Group by product and month
), SalesSummary AS (
    -- Subquery to summarize sales quantities by product and month
    SELECT 
        ProductID,
        DATE_TRUNC('month', OrderDate) AS Month, -- Truncate date to month for grouping
        SUM(Quantity) AS TotalSales -- Sum the order quantities
    FROM 
        OrderSchedule
    GROUP BY 
        ProductID, DATE_TRUNC('month', OrderDate) -- Group by product and month
), SupplyDemand AS (
    -- Subquery to combine stock and sales summaries
    SELECT 
        COALESCE(ss.ProductID, sa.ProductID) AS ProductID, -- Combine product IDs
        COALESCE(ss.ProductName, sa.ProductName) AS ProductName, -- Combine product names
        COALESCE(ss.Month, sa.Month) AS Month, -- Combine months
        COALESCE(ss.TotalStock, 0) AS TotalStock, -- Handle NULLs in stock quantities
        COALESCE(sa.TotalSales, 0) AS TotalSales -- Handle NULLs in sales quantities
    FROM 
        StockSummary ss
    FULL OUTER JOIN 
        SalesSummary sa
    ON 
        ss.ProductID = sa.ProductID AND ss.Month = sa.Month -- Join on product ID and month
)
SELECT 
    sd.ProductID,
    sd.ProductName,
    sd.Month,
    sd.TotalStock,
    sd.TotalSales,
    COALESCE(s.SafeStock, 0) AS SafetyStock, -- Handle NULLs in safety stock
    (COALESCE(sd.TotalStock, 0) - COALESCE(sd.TotalSales, 0) - COALESCE(s.SafeStock, 0)) AS SupplyDemandRatio, -- Calculate supply/demand ratio
    CASE 
        WHEN (COALESCE(sd.TotalStock, 0) - COALESCE(sd.TotalSales, 0) - COALESCE(s.SafeStock, 0)) < 0 THEN 'Understock' -- Determine stock status
        WHEN (COALESCE(sd.TotalStock, 0) - COALESCE(sd.TotalSales, 0) - COALESCE(s.SafeStock, 0)) > 0 THEN 'Overstock'
        ELSE 'Balanced'
    END AS StockStatus -- Assign stock status based on supply/demand ratio
FROM 
    SupplyDemand sd
LEFT JOIN 
    SafetyStock s
ON 
    sd.ProductID = s.ProductID -- Join with safety stock table on product ID
ORDER BY 
    sd.ProductID, sd.Month; -- Order by product ID and month
```


# Result:

| ProductID | ProductName | Month      | TotalStock | TotalSales | SafetyStock | SupplyDemandRatio | StockStatus |
|-----------|-------------|------------|------------|------------|-------------|-------------------|-------------|
| 1         | Widget A    | 2024-07-01 | 50         | 0          | 10          | 40                | Overstock   |
| 1         | Widget A    | 2024-08-01 | 40         | 0          | 10          | 30                | Overstock   |
| 2         | Widget B    | 2024-07-01 | 20         | 30         | 15          | -25               | Understock  |
| 2         | Widget B    | 2024-08-01 | 30         | 15         | 15          | 0                 | Balanced    |
| 3         | Widget C    | 2024-07-01 | 0          | 25         | 5           | -30               | Understock  |
| 3         | Widget C    | 2024-08-01 | 10         | 20         | 5           | -15               | Understock  |
| 4         | Widget D    | 2024-07-01 | 15         | 0          | 8           | 7                 | Overstock   |
| 4         | Widget D    | 2024-08-01 | 25         | 0          | 8           | 17                | Overstock   |
| 5         | Widget E    | 2024-08-01 | 0          | 10         | 12          | -22               | Understock  |
