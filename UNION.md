# Background

In the logistics industry, managing and monitoring the success rate of deliveries is crucial for maintaining customer satisfaction and operational efficiency. One key metric to track is the ratio of failed deliveries over different periods, such as weekly and monthly. This helps in identifying patterns, potential issues, and areas for improvement in the delivery process.

The objective is to analyze the failed delivery ratio for different services (e.g., bike, truck) over weekly and monthly periods. The failed delivery ratio is calculated as the number of failed deliveries divided by the total number of orders created in the same period. By summarizing this information, we can gain insights into the performance and reliability of different delivery services over time.

# Table Structure

The analysis is based on a table named failed_delivery which has the following structure:

| creation_date | service_id | number_create_order | number_failed_deli |
|---------------|------------|---------------------|--------------------|
| 2023-05-01    | bike       | 100                 | 10                 |
| 2023-05-02    | truck      | 150                 | 5                  |
| 2023-05-08    | bike       | 120                 | 12                 |
| 2023-05-09    | truck      | 160                 | 8                  |
| 2023-05-15    | bike       | 110                 | 11                 |
| 2023-05-16    | truck      | 140                 | 7                  |
| 2023-06-01    | bike       | 130                 | 9                  |
| 2023-06-02    | truck      | 170                 | 6                  |


# SQL Query
To achieve the desired summary, the SQL query uses a combination of Common Table Expressions (CTEs) and the UNION operator to calculate the failed delivery ratio for both weekly and monthly views. Here is the query:

```sql
WITH WeeklySummary AS (
    SELECT
        DATE_TRUNC('week', creation_date) AS date,
        'week' AS view,
        service_id,
        SUM(number_failed_deli) * 1.0 / SUM(number_create_order) AS failed_rate
    FROM failed_delivery
    GROUP BY DATE_TRUNC('week', creation_date), service_id
),
MonthlySummary AS (
    SELECT
        DATE_TRUNC('month', creation_date) AS date,
        'month' AS view,
        service_id,
        SUM(number_failed_deli) * 1.0 / SUM(number_create_order) AS failed_rate
    FROM failed_delivery
    GROUP BY DATE_TRUNC('month', creation_date), service_id
)
SELECT * FROM WeeklySummary
UNION ALL
SELECT * FROM MonthlySummary
ORDER BY date, view, service_id;
```

## 1. Weekly Summary CTE

This part of the query calculates the failed delivery ratio for each week. It truncates the creation_date to the start of the week using DATE_TRUNC('week', creation_date), groups the data by this truncated date and service_id, and calculates the ratio of failed deliveries (SUM(number_failed_deli) / SUM(number_create_order)). The view column is set to 'week' to indicate that these rows represent weekly summaries.
```sql
WITH WeeklySummary AS (
    SELECT
        DATE_TRUNC('week', creation_date) AS date,
        'week' AS view,
        service_id,
        SUM(number_failed_deli) * 1.0 / SUM(number_create_order) AS failed_rate
    FROM failed_delivery
    GROUP BY DATE_TRUNC('week', creation_date), service_id
),
```
This Common Table Expression (CTE) calculates the failed delivery ratio for each week by truncating the creation_date to the start of the week.

| date       | view | service_id | failed_rate |
|------------|------|------------|-------------|
| 2023-05-01 | week | bike       | 0.10        |
| 2023-05-01 | week | truck      | 0.033       |
| 2023-05-08 | week | bike       | 0.10        |
| 2023-05-08 | week | truck      | 0.05        |
| 2023-05-15 | week | bike       | 0.10        |
| 2023-05-15 | week | truck      | 0.05        |
| 2023-05-22 | week | bike       | 0.05        |
| 2023-05-22 | week | truck      | 0.03        |

## 2. Monthly Summary CTE

Similarly, this part of the query calculates the failed delivery ratio for each month. It truncates the creation_date to the start of the month using DATE_TRUNC('month', creation_date), groups the data by this truncated date and service_id, and calculates the ratio of failed deliveries. The view column is set to 'month' to indicate that these rows represent monthly summaries.
```sql
MonthlySummary AS (
    SELECT
        DATE_TRUNC('month', creation_date) AS date,
        'month' AS view,
        service_id,
        SUM(number_failed_deli) * 1.0 / SUM(number_create_order) AS failed_rate
    FROM failed_delivery
    GROUP BY DATE_TRUNC('month', creation_date), service_id
)
```

This CTE calculates the failed delivery ratio for each month by truncating the creation_date to the start of the month.

| date       | view  | service_id | failed_rate |
|------------|-------|------------|-------------|
| 2023-05-01 | month | bike       | 0.10        |
| 2023-05-01 | month | truck      | 0.04        |
| 2023-06-01 | month | bike       | 0.07        |
| 2023-06-01 | month | truck      | 0.035       |

## 3. Final Result
Final Result: The UNION ALL operator combines the results from the weekly and monthly summaries into a single result set. The final result includes the date, view type (week or month), service_id, and failed delivery ratio (failed_rate). The result is ordered by date, view, and service_id for better readability.
```sql
SELECT * FROM WeeklySummary
UNION ALL
SELECT * FROM MonthlySummary
ORDER BY date, view, service_id;
```
The final result combines the weekly and monthly summaries using the UNION ALL operator, providing a comprehensive view of the failed delivery ratios over different periods.

| date       | view  | service_id | failed_rate |
|------------|-------|------------|-------------|
| 2023-05-01 | week  | bike       | 0.10        |
| 2023-05-01 | week  | truck      | 0.033       |
| 2023-05-01 | month | bike       | 0.10        |
| 2023-05-01 | month | truck      | 0.04        |
| 2023-05-08 | week  | bike       | 0.10        |
| 2023-05-08 | week  | truck      | 0.05        |
| 2023-05-15 | week  | bike       | 0.10        |
| 2023-05-15 | week  | truck      | 0.05        |
| 2023-05-22 | week  | bike       | 0.05        |
| 2023-05-22 | week  | truck      | 0.03        |
| 2023-06-01 | month | bike       | 0.07        |
| 2023-06-01 | month | truck      | 0.035       |


# Benefit
Combining weekly and monthly summaries into a single table offers several benefits:

**Comprehensive Overview:** Provides a complete view of failed delivery ratios across different periods, enabling quick comparisons.

**Simplified Analysis:** Makes filtering and aggregating data easier, supporting straightforward queries and data manipulation.

**Enhanced Flexibility:** Allows switching between weekly and monthly views effortlessly, helping identify short-term and long-term trends.

**Improved Data Management:** Reduces redundancy and inconsistencies, ensuring uniform data storage and access.

**Streamlined Reporting:** Facilitates report generation with both weekly and monthly metrics, beneficial for dashboards and reporting tools.

Example Use Case: A logistics manager can quickly demonstrate delivery performance over short and long periods, aiding informed decision-making.

![image](https://github.com/aihtn2708/SQLinRealWorld/assets/17986030/b3efd9e2-f4ea-4a4b-b30a-01657982fde8)
