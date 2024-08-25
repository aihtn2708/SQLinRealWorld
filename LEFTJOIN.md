```sql
-- Creating a series for all months (1 to 12)
WITH all_months AS (
    SELECT generate_series(1, 12) AS month
),
monthly_transactions AS (
    SELECT
        EXTRACT(MONTH FROM date) AS month,
        SUM(amount) AS total_amount,
        SUM(CASE WHEN amount < 0 THEN amount ELSE 0 END) AS total_payments,
        COUNT(CASE WHEN amount < 0 THEN 1 END) AS payment_count
    FROM
        transactions
    GROUP BY
        EXTRACT(MONTH FROM date)
),
-- Left join to ensure all months are included
full_monthly_transactions AS (
    SELECT
        m.month,
        COALESCE(mt.total_amount, 0) AS total_amount,
        COALESCE(mt.total_payments, 0) AS total_payments,
        COALESCE(mt.payment_count, 0) AS payment_count
    FROM
        all_months m
    LEFT JOIN
        monthly_transactions mt ON m.month = mt.month
),
fees AS (
    SELECT
        month,
        CASE
            WHEN payment_count >= 3 AND total_payments <= -100 THEN 0
            ELSE 5
        END AS fee
    FROM
        full_monthly_transactions
),
total_fees AS (
    SELECT SUM(fee) AS total_fee FROM fees
),
raw_balance AS (
    SELECT SUM(amount) AS total_amount FROM transactions
),
final_balance AS (
    SELECT (total_amount - total_fee) AS balance
    FROM raw_balance, total_fees
)
SELECT balance FROM final_balance;

```
