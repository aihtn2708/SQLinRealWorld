Using CROSS JOIN, we can transform product status intervals into daily records. This approach is crucial for preparing data for analytics with systems like ERP, and Distribution/Sales Management System, which store data such as product availability or material prices with valid-from and valid-to dates, ensuring accurate daily records without storing daily transactions. 

Eg:
![image](https://github.com/aihtn2708/SQLinRealWorld/assets/17986030/b551809e-46f0-4893-b0b8-a8aa08a6f376)
This approach leverages a CROSS JOIN to generate all possible date combinations and then filters them according to the status periods defined in the ProductStatus table.


1. Create the DailyProductStatus Table: This table will store the daily statuses.

2. Use CTE to generate a range of dates
 - DateRange CTE: Finds the minimum and maximum dates in the ProductStatus table.
 - AllDates CTE: Generates all dates between the minimum and maximum dates using generate_series.

3. CROSS JOIN and Filtering:
 - The CROSS JOIN between ProductStatus and AllDates generates a combination of all products and all dates.
 - The WHERE clause filters these combinations to include only the relevant dates for each product status period.

4. Insert into DailyProductStatus: The final selection inserts the filtered daily statuses into the DailyProductStatus table, ordered by SKU and date.

```sql
--create the DailyProductStatus table
 CREATE TABLE DailyProductStatus (
    SKU VARCHAR(10),
    Date DATE,
    Status VARCHAR(10)
);

-- Create a Date Range using a CTE
WITH DateRange AS (
    SELECT 
        MIN(Date) AS StartDate,
        MAX(Date) AS EndDate
    FROM 
        ProductStatus
),
AllDates AS (
    SELECT
        d.StartDate + INTERVAL '1' DAY * n AS Date
    FROM
        DateRange d,
        generate_series(0, (SELECT EXTRACT(DAY FROM (d.EndDate - d.StartDate)))::int) AS t(n)
)
-- Insert data into DailyProductStatus
INSERT INTO DailyProductStatus (SKU, Date, Status)
SELECT 
    p.SKU, 
    ad.Date, 
    p.Status
FROM 
    ProductStatus p
CROSS JOIN 
    AllDates ad
WHERE 
    ad.Date BETWEEN p.Date AND COALESCE(LEAD(p.Date) OVER (PARTITION BY p.SKU ORDER BY p.Date) - INTERVAL '1' DAY, ad.Date)
ORDER BY 
    p.SKU, ad.Date;
```
