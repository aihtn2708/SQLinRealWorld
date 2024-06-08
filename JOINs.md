# Tables:

## products

- product_id: Unique identifier for each product (Primary Key).
- product_name: Name of the product.
- sku: SKU of the product.
- price: Price of the product.

## promotions

- promotion_id: Unique identifier for each promotion (Primary Key).
- promotion_name: Name of the promotion.
- start_date: Start date of the promotion.
- end_date: End date of the promotion.

## promotion_products

- promotion_id: Identifier for the promotion (Foreign Key referencing promotions.promotion_id).
- product_id: Identifier for the product (Foreign Key referencing products.product_id).
- discount_percentage: Discount percentage for the promotion.

## sales

- sale_id: Unique identifier for each sale (Primary Key).
- product_id: Identifier for the product (Foreign Key referencing products.product_id).
- sale_date: Date of the sale.
- quantity: Quantity sold.

# Relationships:
- promotion_products.product_id references products.product_id.
- promotion_products.promotion_id references promotions.promotion_id.
- sales.product_id references products.product_id.

# ERD:
![image](https://github.com/aihtn2708/SQLinRealWorld/assets/17986030/96b2bf0b-b4da-4314-95d9-6d91c1b6e7bd)

```dbml
Table products {
  product_id int [pk, increment] // primary key, auto-increment
  product_name varchar
  sku varchar
  price decimal
}

Table promotions {
  promotion_id int [pk, increment] // primary key, auto-increment
  promotion_name varchar
  start_date date
  end_date date
}

Table promotion_products {
  promotion_id int // foreign key
  product_id int // foreign key
  discount_percentage decimal
}

Table sales {
  sale_id int // primary key, auto-increment
  product_id int // foreign key
  sale_date date
  quantity int
}

Ref: promotion_products.product_id > products.product_id
Ref: promotion_products.promotion_id > promotions.promotion_id
Ref: sales.product_id > products.product_id
```

Below is the use case of e-commerce SKU promotion management with examples of different types of SQL JOINs (INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN, and CROSS JOIN).

# 1. INNER JOIN
**Example: Retrieve current promotions for each product

This query retrieves details of products that are currently under promotion. Only products with active promotions will be included in the results.
```sql
SELECT 
    products.product_id, 
    products.product_name, 
    products.sku, 
    promotions.promotion_name, 
    promotion_products.discount_percentage
FROM 
    products


INNER JOIN promotion_products ON products.product_id = promotion_products.product_id
INNER JOIN promotions ON promotion_products.promotion_id = promotions.promotion_id
WHERE 
    CURRENT_DATE BETWEEN promotions.start_date AND promotions.end_date;
```

## 2. LEFT JOIN
Example: List all products with their promotions, including products without any promotions.

This query retrieves all products, including those without any active promotions. Products without promotions will have NULL values for the promotion details.
```sql
SELECT 
    products.product_id, 
    products.product_name, 
    products.sku, 
    promotions.promotion_name, 
    promotion_products.discount_percentage
FROM 
    products
LEFT JOIN promotion_products ON products.product_id = promotion_products.product_id
LEFT JOIN promotions ON promotion_products.promotion_id = promotions.promotion_id
AND CURRENT_DATE BETWEEN promotions.start_date AND promotions.end_date;
```

## 3. RIGHT JOIN
Example: List all promotions and the products under each promotion, including promotions without any products

This query retrieves all promotions, including those without any products associated with them. Promotions without products will have NULL values for the product details.
```sql
SELECT 
    products.product_id, 
    products.product_name, 
    promotions.promotion_name, 
    promotions.start_date, 
    promotions.end_date
FROM 
    products
RIGHT JOIN promotion_products ON products.product_id = promotion_products.product_id
RIGHT JOIN promotions ON promotion_products.promotion_id = promotions.promotion_id
WHERE 
    CURRENT_DATE BETWEEN promotions.start_date AND promotions.end_date;
```

## 4. FULL OUTER JOIN
Example: List all products and all promotions, matching where possible

This query retrieves all products and all promotions, including products without promotions and promotions without products. Both unmatched products and promotions will have NULL values for the missing details.
```sql
SELECT 
    products.product_id, 
    products.product_name, 
    promotions.promotion_name, 
    promotion_products.discount_percentage
FROM 
    products
FULL OUTER JOIN promotion_products ON products.product_id = promotion_products.product_id
FULL OUTER JOIN promotions ON promotion_products.promotion_id = promotions.promotion_id
AND CURRENT_DATE BETWEEN promotions.start_date AND promotions.end_date;
```

## 5. CROSS JOIN
Example: Generate a matrix of all products and all promotions to analyze potential future promotions

This query creates a Cartesian product of all products and all promotions. Each product will be paired with each promotion, useful for planning and analyzing potential future promotions.
```sql
SELECT 
    p.product_id, 
    p.product_name, 
    pr.promotion_id, 
    pr.promotion_name
FROM 
    products p
CROSS JOIN promotions pr;
```


## 6. SELF JOIN
Example: Finding Products with Multiple Promotions

This query identifies products that have been linked to more than one promotion.
```sql
SELECT 
    pp1.product_id, 
    p.product_name, 
    pp1.promotion_id AS promotion1_id, 
    pr1.promotion_name AS promotion1_name, 
    pp2.promotion_id AS promotion2_id, 
    pr2.promotion_name AS promotion2_name
FROM 
    promotion_products pp1
INNER JOIN promotion_products pp2 ON pp1.product_id = pp2.product_id 
    AND pp1.promotion_id != pp2.promotion_id
INNER JOIN products p ON pp1.product_id = p.product_id
INNER JOIN promotions pr1 ON pp1.promotion_id = pr1.promotion_id
INNER JOIN promotions pr2 ON pp2.promotion_id = pr2.promotion_id
ORDER BY 
    pp1.product_id, 
    pp1.promotion_id, 
    pp2.promotion_id;
```
