# Level 2: Advanced SQL Query Writing

## Prerequisites
- Level 1 completion (basic SQL execution and simple queries)
- Understanding of database relationships and normalization
- Familiarity with at least one database system (PostgreSQL, MySQL, SQL Server)
- Basic knowledge of data types and constraints
- Experience with simple JOIN operations

## Problem Statement
As a backend engineer, you need to:
- **Write complex queries** with multiple JOINs and subqueries
- **Optimize query performance** using indexes and query plans
- **Handle advanced data manipulation** with CTEs and window functions
- **Implement business logic** using stored procedures and functions
- **Manage transactions** and ensure data consistency
- **Work with JSON and advanced data types**
- **Create efficient reporting queries** for analytics

These skills are essential for building robust backend applications that handle complex data requirements efficiently.

---

## Key Concepts

### 1. Advanced JOIN Operations

#### Multiple Table JOINs
```sql
-- Complex JOIN with multiple tables
SELECT 
    u.username,
    u.email,
    p.title as profile_title,
    o.order_date,
    o.total_amount,
    oi.quantity,
    pr.name as product_name,
    pr.price,
    c.name as category_name
FROM users u
INNER JOIN profiles p ON u.id = p.user_id
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products pr ON oi.product_id = pr.id
INNER JOIN categories c ON pr.category_id = c.id
WHERE o.order_date >= '2024-01-01'
  AND o.status = 'completed'
ORDER BY o.order_date DESC, u.username;
```

#### Self JOINs and Hierarchical Data
```sql
-- Employee hierarchy with self JOIN
SELECT 
    e.employee_id,
    e.name as employee_name,
    e.position,
    m.name as manager_name,
    m.position as manager_position
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
ORDER BY m.name, e.name;

-- Find all subordinates of a specific manager (recursive)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: direct reports
    SELECT employee_id, name, manager_id, 1 as level
    FROM employees 
    WHERE manager_id = 100
    
    UNION ALL
    
    -- Recursive case: subordinates of subordinates
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy
ORDER BY level, name;
```

#### Advanced JOIN Techniques
```sql
-- LATERAL JOIN (PostgreSQL) for correlated subqueries
SELECT 
    c.customer_name,
    c.email,
    recent_orders.order_count,
    recent_orders.total_spent
FROM customers c
CROSS JOIN LATERAL (
    SELECT 
        COUNT(*) as order_count,
        SUM(total_amount) as total_spent
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.order_date >= CURRENT_DATE - INTERVAL '30 days'
) recent_orders
WHERE recent_orders.order_count > 0;

-- FULL OUTER JOIN for data comparison
SELECT 
    COALESCE(current_month.product_id, last_month.product_id) as product_id,
    COALESCE(current_month.sales, 0) as current_sales,
    COALESCE(last_month.sales, 0) as last_month_sales,
    COALESCE(current_month.sales, 0) - COALESCE(last_month.sales, 0) as sales_difference
FROM (
    SELECT product_id, SUM(quantity * price) as sales
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.id
    WHERE EXTRACT(MONTH FROM o.order_date) = EXTRACT(MONTH FROM CURRENT_DATE)
    GROUP BY product_id
) current_month
FULL OUTER JOIN (
    SELECT product_id, SUM(quantity * price) as sales
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.id
    WHERE EXTRACT(MONTH FROM o.order_date) = EXTRACT(MONTH FROM CURRENT_DATE) - 1
    GROUP BY product_id
) last_month ON current_month.product_id = last_month.product_id;
```

### 2. Subqueries and Common Table Expressions (CTEs)

#### Correlated Subqueries
```sql
-- Find customers who spent more than average in their city
SELECT 
    c.customer_id,
    c.name,
    c.city,
    c.total_spent
FROM (
    SELECT 
        c.customer_id,
        c.name,
        c.city,
        SUM(o.total_amount) as total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.name, c.city
) c
WHERE c.total_spent > (
    SELECT AVG(city_avg.avg_spent)
    FROM (
        SELECT 
            c2.city,
            AVG(SUM(o2.total_amount)) as avg_spent
        FROM customers c2
        JOIN orders o2 ON c2.customer_id = o2.customer_id
        WHERE c2.city = c.city
        GROUP BY c2.customer_id, c2.city
    ) city_avg
);

-- EXISTS vs IN performance comparison
-- Using EXISTS (often more efficient)
SELECT c.customer_id, c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id 
      AND o.order_date >= '2024-01-01'
);

-- Using IN (can be less efficient with large datasets)
SELECT c.customer_id, c.name
FROM customers c
WHERE c.customer_id IN (
    SELECT DISTINCT o.customer_id 
    FROM orders o 
    WHERE o.order_date >= '2024-01-01'
);
```

#### Advanced CTEs
```sql
-- Multiple CTEs for complex analysis
WITH monthly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM o.order_date) as year,
        EXTRACT(MONTH FROM o.order_date) as month,
        SUM(o.total_amount) as total_sales,
        COUNT(DISTINCT o.customer_id) as unique_customers,
        COUNT(*) as order_count
    FROM orders o
    WHERE o.status = 'completed'
    GROUP BY EXTRACT(YEAR FROM o.order_date), EXTRACT(MONTH FROM o.order_date)
),
sales_with_growth AS (
    SELECT 
        year,
        month,
        total_sales,
        unique_customers,
        order_count,
        LAG(total_sales) OVER (ORDER BY year, month) as prev_month_sales,
        total_sales - LAG(total_sales) OVER (ORDER BY year, month) as sales_growth
    FROM monthly_sales
),
performance_metrics AS (
    SELECT 
        year,
        month,
        total_sales,
        unique_customers,
        order_count,
        sales_growth,
        CASE 
            WHEN prev_month_sales > 0 THEN 
                ROUND((sales_growth / prev_month_sales * 100), 2)
            ELSE NULL 
        END as growth_percentage,
        total_sales / order_count as avg_order_value,
        total_sales / unique_customers as avg_customer_value
    FROM sales_with_growth
)
SELECT 
    year,
    month,
    total_sales,
    unique_customers,
    order_count,
    sales_growth,
    growth_percentage,
    avg_order_value,
    avg_customer_value,
    CASE 
        WHEN growth_percentage > 10 THEN 'Excellent'
        WHEN growth_percentage > 5 THEN 'Good'
        WHEN growth_percentage > 0 THEN 'Positive'
        WHEN growth_percentage < 0 THEN 'Declining'
        ELSE 'Stable'
    END as performance_category
FROM performance_metrics
ORDER BY year DESC, month DESC;
```

### 3. Window Functions

#### Ranking and Analytical Functions
```sql
-- Advanced window functions for sales analysis
SELECT 
    p.name as product_name,
    c.name as category_name,
    SUM(oi.quantity * oi.price) as total_revenue,
    
    -- Ranking within category
    ROW_NUMBER() OVER (PARTITION BY c.id ORDER BY SUM(oi.quantity * oi.price) DESC) as rank_in_category,
    RANK() OVER (PARTITION BY c.id ORDER BY SUM(oi.quantity * oi.price) DESC) as dense_rank_in_category,
    
    -- Percentage of category sales
    ROUND(
        SUM(oi.quantity * oi.price) * 100.0 / 
        SUM(SUM(oi.quantity * oi.price)) OVER (PARTITION BY c.id), 
        2
    ) as pct_of_category_sales,
    
    -- Running totals
    SUM(SUM(oi.quantity * oi.price)) OVER (
        PARTITION BY c.id 
        ORDER BY SUM(oi.quantity * oi.price) DESC 
        ROWS UNBOUNDED PRECEDING
    ) as running_total,
    
    -- Moving averages
    AVG(SUM(oi.quantity * oi.price)) OVER (
        PARTITION BY c.id 
        ORDER BY SUM(oi.quantity * oi.price) DESC 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3_products
    
FROM products p
JOIN categories c ON p.category_id = c.id
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'completed'
  AND o.order_date >= '2024-01-01'
GROUP BY p.id, p.name, c.id, c.name
ORDER BY c.name, total_revenue DESC;
```

#### Time Series Analysis
```sql
-- Advanced time series analysis with window functions
WITH daily_metrics AS (
    SELECT 
        DATE(o.order_date) as order_date,
        COUNT(*) as daily_orders,
        SUM(o.total_amount) as daily_revenue,
        COUNT(DISTINCT o.customer_id) as daily_customers
    FROM orders o
    WHERE o.status = 'completed'
      AND o.order_date >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY DATE(o.order_date)
),
time_series_analysis AS (
    SELECT 
        order_date,
        daily_orders,
        daily_revenue,
        daily_customers,
        
        -- Previous day comparison
        LAG(daily_revenue) OVER (ORDER BY order_date) as prev_day_revenue,
        daily_revenue - LAG(daily_revenue) OVER (ORDER BY order_date) as revenue_change,
        
        -- 7-day moving averages
        AVG(daily_revenue) OVER (
            ORDER BY order_date 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) as revenue_7day_avg,
        
        AVG(daily_orders) OVER (
            ORDER BY order_date 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) as orders_7day_avg,
        
        -- Week-over-week comparison
        LAG(daily_revenue, 7) OVER (ORDER BY order_date) as same_day_last_week,
        
        -- Percentile rankings
        PERCENT_RANK() OVER (ORDER BY daily_revenue) as revenue_percentile,
        
        -- Identify outliers (beyond 2 standard deviations)
        ABS(daily_revenue - AVG(daily_revenue) OVER ()) / 
        STDDEV(daily_revenue) OVER () as revenue_z_score
        
    FROM daily_metrics
)
SELECT 
    order_date,
    daily_orders,
    daily_revenue,
    daily_customers,
    revenue_change,
    ROUND(revenue_7day_avg, 2) as revenue_7day_avg,
    ROUND(orders_7day_avg, 2) as orders_7day_avg,
    
    -- Week-over-week growth
    CASE 
        WHEN same_day_last_week > 0 THEN 
            ROUND((daily_revenue - same_day_last_week) / same_day_last_week * 100, 2)
        ELSE NULL 
    END as wow_growth_pct,
    
    ROUND(revenue_percentile * 100, 1) as revenue_percentile,
    
    -- Flag outliers
    CASE 
        WHEN ABS(revenue_z_score) > 2 THEN 'Outlier'
        WHEN ABS(revenue_z_score) > 1.5 THEN 'Unusual'
        ELSE 'Normal'
    END as revenue_pattern
    
FROM time_series_analysis
ORDER BY order_date DESC;
```

### 4. Advanced Data Manipulation

#### MERGE/UPSERT Operations
```sql
-- PostgreSQL UPSERT with ON CONFLICT
INSERT INTO product_inventory (product_id, warehouse_id, quantity, last_updated)
VALUES 
    (1, 1, 100, CURRENT_TIMESTAMP),
    (2, 1, 50, CURRENT_TIMESTAMP),
    (3, 1, 75, CURRENT_TIMESTAMP)
ON CONFLICT (product_id, warehouse_id) 
DO UPDATE SET 
    quantity = EXCLUDED.quantity + product_inventory.quantity,
    last_updated = EXCLUDED.last_updated
RETURNING product_id, warehouse_id, quantity;

-- SQL Server MERGE statement
MERGE customer_summary AS target
USING (
    SELECT 
        customer_id,
        COUNT(*) as total_orders,
        SUM(total_amount) as total_spent,
        MAX(order_date) as last_order_date
    FROM orders
    WHERE status = 'completed'
    GROUP BY customer_id
) AS source ON target.customer_id = source.customer_id
WHEN MATCHED THEN
    UPDATE SET 
        total_orders = source.total_orders,
        total_spent = source.total_spent,
        last_order_date = source.last_order_date,
        updated_at = GETDATE()
WHEN NOT MATCHED THEN
    INSERT (customer_id, total_orders, total_spent, last_order_date, created_at)
    VALUES (source.customer_id, source.total_orders, source.total_spent, 
            source.last_order_date, GETDATE());
```

#### Bulk Data Operations
```sql
-- Efficient bulk updates with CTEs
WITH price_updates AS (
    SELECT 
        p.id,
        p.current_price,
        CASE 
            WHEN c.name = 'Electronics' THEN p.current_price * 1.1
            WHEN c.name = 'Clothing' THEN p.current_price * 1.05
            WHEN c.name = 'Books' THEN p.current_price * 1.02
            ELSE p.current_price
        END as new_price
    FROM products p
    JOIN categories c ON p.category_id = c.id
    WHERE p.last_price_update < CURRENT_DATE - INTERVAL '30 days'
)
UPDATE products 
SET 
    current_price = price_updates.new_price,
    last_price_update = CURRENT_DATE,
    price_history = COALESCE(price_history, '[]'::jsonb) || 
                   jsonb_build_object(
                       'date', CURRENT_DATE,
                       'old_price', price_updates.current_price,
                       'new_price', price_updates.new_price
                   )
FROM price_updates
WHERE products.id = price_updates.id;

-- Conditional bulk inserts
INSERT INTO audit_log (table_name, operation, record_id, old_values, new_values, changed_by, changed_at)
SELECT 
    'products' as table_name,
    'UPDATE' as operation,
    p.id as record_id,
    jsonb_build_object('price', p.current_price) as old_values,
    jsonb_build_object('price', pu.new_price) as new_values,
    'system' as changed_by,
    CURRENT_TIMESTAMP as changed_at
FROM products p
JOIN price_updates pu ON p.id = pu.id
WHERE p.current_price != pu.new_price;
```

### 5. JSON and Advanced Data Types

#### JSON Operations
```sql
-- PostgreSQL JSON operations
-- Query JSON data
SELECT 
    id,
    name,
    metadata->>'category' as category,
    metadata->>'brand' as brand,
    (metadata->>'price')::numeric as price,
    metadata->'specifications'->>'color' as color,
    jsonb_array_length(metadata->'tags') as tag_count
FROM products
WHERE metadata ? 'category'
  AND metadata->>'category' = 'Electronics'
  AND (metadata->>'price')::numeric > 100;

-- Update JSON fields
UPDATE products 
SET metadata = metadata || jsonb_build_object(
    'last_updated', CURRENT_TIMESTAMP,
    'discount_eligible', true,
    'specifications', metadata->'specifications' || jsonb_build_object(
        'warranty_months', 12
    )
)
WHERE metadata->>'category' = 'Electronics';

-- Aggregate JSON data
SELECT 
    metadata->>'category' as category,
    COUNT(*) as product_count,
    AVG((metadata->>'price')::numeric) as avg_price,
    jsonb_agg(DISTINCT metadata->>'brand') as brands,
    jsonb_object_agg(name, metadata->>'price') as product_prices
FROM products
WHERE metadata ? 'category'
GROUP BY metadata->>'category';

-- JSON path queries (PostgreSQL 12+)
SELECT 
    id,
    name,
    jsonb_path_query_array(metadata, '$.specifications.dimensions.*') as all_dimensions,
    jsonb_path_query_first(metadata, '$.tags[*] ? (@ like_regex "smart" flag "i")') as smart_tag
FROM products
WHERE jsonb_path_exists(metadata, '$.specifications.dimensions');
```

#### Array Operations
```sql
-- PostgreSQL array operations
SELECT 
    id,
    name,
    tags,
    array_length(tags, 1) as tag_count,
    'electronics' = ANY(tags) as is_electronics,
    tags && ARRAY['smart', 'wireless'] as has_smart_or_wireless
FROM products
WHERE tags IS NOT NULL;

-- Array aggregation and manipulation
SELECT 
    category_id,
    array_agg(DISTINCT name ORDER BY name) as product_names,
    array_agg(price ORDER BY price DESC) as prices_desc,
    array_remove(array_agg(tags), NULL) as all_tags_nested,
    array_agg(DISTINCT unnest(tags)) as unique_tags
FROM products
GROUP BY category_id;

-- Unnest arrays for analysis
SELECT 
    unnest(tags) as tag,
    COUNT(*) as usage_count,
    AVG(price) as avg_price_for_tag
FROM products
WHERE tags IS NOT NULL
GROUP BY unnest(tags)
ORDER BY usage_count DESC;
```

### 6. Stored Procedures and Functions

#### PostgreSQL Functions
```sql
-- Function for complex business logic
CREATE OR REPLACE FUNCTION calculate_customer_tier(
    customer_id_param INTEGER,
    evaluation_period_months INTEGER DEFAULT 12
) RETURNS TEXT AS $$
DECLARE
    total_spent NUMERIC;
    order_count INTEGER;
    avg_order_value NUMERIC;
    tier TEXT;
BEGIN
    -- Calculate customer metrics
    SELECT 
        COALESCE(SUM(total_amount), 0),
        COUNT(*),
        COALESCE(AVG(total_amount), 0)
    INTO total_spent, order_count, avg_order_value
    FROM orders
    WHERE customer_id = customer_id_param
      AND status = 'completed'
      AND order_date >= CURRENT_DATE - (evaluation_period_months || ' months')::INTERVAL;
    
    -- Determine tier based on business rules
    IF total_spent >= 10000 AND order_count >= 20 THEN
        tier := 'Platinum';
    ELSIF total_spent >= 5000 AND order_count >= 10 THEN
        tier := 'Gold';
    ELSIF total_spent >= 1000 AND order_count >= 5 THEN
        tier := 'Silver';
    ELSIF order_count > 0 THEN
        tier := 'Bronze';
    ELSE
        tier := 'New';
    END IF;
    
    -- Log the calculation
    INSERT INTO customer_tier_history (customer_id, tier, total_spent, order_count, calculated_at)
    VALUES (customer_id_param, tier, total_spent, order_count, CURRENT_TIMESTAMP);
    
    RETURN tier;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT 
    c.id,
    c.name,
    calculate_customer_tier(c.id) as current_tier
FROM customers c
WHERE c.status = 'active';
```

#### Table-Valued Functions
```sql
-- Function returning table for reporting
CREATE OR REPLACE FUNCTION get_sales_report(
    start_date DATE,
    end_date DATE,
    category_filter TEXT DEFAULT NULL
) RETURNS TABLE (
    product_id INTEGER,
    product_name TEXT,
    category_name TEXT,
    total_quantity INTEGER,
    total_revenue NUMERIC,
    avg_price NUMERIC,
    order_count INTEGER
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        p.id,
        p.name,
        c.name,
        SUM(oi.quantity)::INTEGER,
        SUM(oi.quantity * oi.price),
        AVG(oi.price),
        COUNT(DISTINCT oi.order_id)::INTEGER
    FROM products p
    JOIN categories c ON p.category_id = c.id
    JOIN order_items oi ON p.id = oi.product_id
    JOIN orders o ON oi.order_id = o.id
    WHERE o.order_date BETWEEN start_date AND end_date
      AND o.status = 'completed'
      AND (category_filter IS NULL OR c.name = category_filter)
    GROUP BY p.id, p.name, c.name
    ORDER BY SUM(oi.quantity * oi.price) DESC;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM get_sales_report('2024-01-01', '2024-12-31', 'Electronics');
```

### 7. Transaction Management

#### Advanced Transaction Control
```sql
-- Complex transaction with savepoints
BEGIN;

-- Create savepoint before risky operation
SAVEPOINT before_inventory_update;

-- Update inventory
UPDATE products 
SET stock_quantity = stock_quantity - 10
WHERE id = 1;

-- Check if update was successful
DO $$
DECLARE
    current_stock INTEGER;
BEGIN
    SELECT stock_quantity INTO current_stock
    FROM products WHERE id = 1;
    
    IF current_stock < 0 THEN
        RAISE EXCEPTION 'Insufficient stock for product 1';
    END IF;
END;
$$;

-- If we reach here, the update was successful
-- Create the order
INSERT INTO orders (customer_id, total_amount, status, order_date)
VALUES (1, 99.99, 'pending', CURRENT_TIMESTAMP);

-- Insert order items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (currval('orders_id_seq'), 1, 10, 9.99);

-- Update customer statistics
UPDATE customer_summary 
SET 
    total_orders = total_orders + 1,
    total_spent = total_spent + 99.99,
    last_order_date = CURRENT_TIMESTAMP
WHERE customer_id = 1;

COMMIT;

-- Error handling with rollback to savepoint
-- ROLLBACK TO SAVEPOINT before_inventory_update;
-- ROLLBACK;
```

#### Isolation Levels and Locking
```sql
-- Set transaction isolation level
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Explicit locking for critical sections
SELECT stock_quantity 
FROM products 
WHERE id = 1 
FOR UPDATE NOWAIT;

-- Perform operations...

COMMIT;

-- Advisory locks for application-level coordination
SELECT pg_advisory_lock(12345);
-- Perform critical operations
SELECT pg_advisory_unlock(12345);
```

---

## Performance Optimization

### 1. Index Strategies

```sql
-- Composite indexes for complex queries
CREATE INDEX idx_orders_customer_date_status 
ON orders (customer_id, order_date DESC, status)
WHERE status IN ('pending', 'completed');

-- Partial indexes for specific conditions
CREATE INDEX idx_products_active_category 
ON products (category_id, price) 
WHERE status = 'active' AND stock_quantity > 0;

-- Expression indexes
CREATE INDEX idx_customers_email_lower 
ON customers (LOWER(email));

-- JSON indexes (PostgreSQL)
CREATE INDEX idx_products_metadata_gin 
ON products USING GIN (metadata);

CREATE INDEX idx_products_category_btree 
ON products USING BTREE ((metadata->>'category'));

-- Analyze index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

### 2. Query Optimization Techniques

```sql
-- Use EXPLAIN ANALYZE to understand query performance
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT 
    c.name,
    COUNT(o.id) as order_count,
    SUM(o.total_amount) as total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id 
    AND o.order_date >= '2024-01-01'
    AND o.status = 'completed'
GROUP BY c.id, c.name
HAVING COUNT(o.id) > 5
ORDER BY total_spent DESC
LIMIT 100;

-- Optimize with proper indexing and query structure
-- Before: Inefficient subquery
SELECT *
FROM products p
WHERE p.id IN (
    SELECT oi.product_id
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.id
    WHERE o.order_date >= '2024-01-01'
);

-- After: Efficient EXISTS
SELECT p.*
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.id
    WHERE oi.product_id = p.id
      AND o.order_date >= '2024-01-01'
);
```

---

## Hands-on Examples

### Example 1: E-commerce Analytics Dashboard

```sql
-- Comprehensive e-commerce analytics query
WITH sales_metrics AS (
    SELECT 
        DATE_TRUNC('month', o.order_date) as month,
        c.name as category,
        COUNT(DISTINCT o.id) as orders,
        COUNT(DISTINCT o.customer_id) as customers,
        SUM(oi.quantity) as units_sold,
        SUM(oi.quantity * oi.price) as revenue,
        AVG(o.total_amount) as avg_order_value
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
    JOIN categories c ON p.category_id = c.id
    WHERE o.status = 'completed'
      AND o.order_date >= CURRENT_DATE - INTERVAL '12 months'
    GROUP BY DATE_TRUNC('month', o.order_date), c.name
),
monthly_totals AS (
    SELECT 
        month,
        SUM(revenue) as total_monthly_revenue,
        SUM(orders) as total_monthly_orders
    FROM sales_metrics
    GROUP BY month
),
category_performance AS (
    SELECT 
        sm.*,
        mt.total_monthly_revenue,
        ROUND(sm.revenue * 100.0 / mt.total_monthly_revenue, 2) as revenue_share_pct,
        LAG(sm.revenue) OVER (PARTITION BY sm.category ORDER BY sm.month) as prev_month_revenue,
        RANK() OVER (PARTITION BY sm.month ORDER BY sm.revenue DESC) as category_rank
    FROM sales_metrics sm
    JOIN monthly_totals mt ON sm.month = mt.month
)
SELECT 
    month,
    category,
    orders,
    customers,
    units_sold,
    revenue,
    avg_order_value,
    revenue_share_pct,
    category_rank,
    CASE 
        WHEN prev_month_revenue > 0 THEN 
            ROUND((revenue - prev_month_revenue) / prev_month_revenue * 100, 2)
        ELSE NULL 
    END as mom_growth_pct,
    CASE 
        WHEN category_rank <= 3 THEN 'Top Performer'
        WHEN category_rank <= 6 THEN 'Good Performer'
        ELSE 'Needs Attention'
    END as performance_tier
FROM category_performance
ORDER BY month DESC, revenue DESC;
```

### Example 2: Customer Segmentation Analysis

```sql
-- RFM (Recency, Frequency, Monetary) Analysis
WITH customer_metrics AS (
    SELECT 
        c.id as customer_id,
        c.name,
        c.email,
        c.registration_date,
        
        -- Recency: Days since last order
        COALESCE(
            EXTRACT(DAY FROM CURRENT_DATE - MAX(o.order_date)), 
            999
        ) as recency_days,
        
        -- Frequency: Number of orders
        COUNT(o.id) as frequency,
        
        -- Monetary: Total spent
        COALESCE(SUM(o.total_amount), 0) as monetary_value,
        
        -- Additional metrics
        COALESCE(AVG(o.total_amount), 0) as avg_order_value,
        MAX(o.order_date) as last_order_date,
        MIN(o.order_date) as first_order_date
        
    FROM customers c
    LEFT JOIN orders o ON c.id = o.customer_id AND o.status = 'completed'
    WHERE c.registration_date <= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY c.id, c.name, c.email, c.registration_date
),
rfm_scores AS (
    SELECT 
        *,
        -- Recency score (1-5, where 5 is most recent)
        CASE 
            WHEN recency_days <= 30 THEN 5
            WHEN recency_days <= 60 THEN 4
            WHEN recency_days <= 90 THEN 3
            WHEN recency_days <= 180 THEN 2
            ELSE 1
        END as recency_score,
        
        -- Frequency score (1-5, where 5 is most frequent)
        CASE 
            WHEN frequency >= 10 THEN 5
            WHEN frequency >= 5 THEN 4
            WHEN frequency >= 3 THEN 3
            WHEN frequency >= 1 THEN 2
            ELSE 1
        END as frequency_score,
        
        -- Monetary score (1-5, where 5 is highest value)
        NTILE(5) OVER (ORDER BY monetary_value) as monetary_score
        
    FROM customer_metrics
),
customer_segments AS (
    SELECT 
        *,
        CONCAT(recency_score, frequency_score, monetary_score) as rfm_segment,
        
        -- Define customer segments based on RFM scores
        CASE 
            WHEN recency_score >= 4 AND frequency_score >= 4 AND monetary_score >= 4 THEN 'Champions'
            WHEN recency_score >= 3 AND frequency_score >= 3 AND monetary_score >= 3 THEN 'Loyal Customers'
            WHEN recency_score >= 4 AND frequency_score <= 2 THEN 'New Customers'
            WHEN recency_score >= 3 AND frequency_score >= 2 AND monetary_score <= 2 THEN 'Potential Loyalists'
            WHEN recency_score >= 3 AND frequency_score <= 2 AND monetary_score >= 3 THEN 'Big Spenders'
            WHEN recency_score <= 2 AND frequency_score >= 3 AND monetary_score >= 3 THEN 'At Risk'
            WHEN recency_score <= 2 AND frequency_score >= 2 AND monetary_score <= 2 THEN 'Cannot Lose Them'
            WHEN recency_score <= 2 AND frequency_score <= 2 AND monetary_score >= 3 THEN 'Hibernating High Value'
            WHEN recency_score <= 2 AND frequency_score <= 2 AND monetary_score <= 2 THEN 'Lost Customers'
            ELSE 'Others'
        END as customer_segment
        
    FROM rfm_scores
)
SELECT 
    customer_segment,
    COUNT(*) as customer_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) as segment_percentage,
    ROUND(AVG(recency_days), 1) as avg_recency_days,
    ROUND(AVG(frequency), 1) as avg_frequency,
    ROUND(AVG(monetary_value), 2) as avg_monetary_value,
    ROUND(SUM(monetary_value), 2) as total_segment_value,
    ROUND(SUM(monetary_value) * 100.0 / SUM(SUM(monetary_value)) OVER (), 2) as value_percentage
FROM customer_segments
GROUP BY customer_segment
ORDER BY total_segment_value DESC;
```

---

## Best Practices

### Query Writing
1. **Use appropriate JOIN types** based on data relationships
2. **Leverage CTEs** for complex logic and readability
3. **Optimize WHERE clauses** with proper indexing
4. **Use window functions** instead of self-joins when possible
5. **Avoid SELECT *** in production queries
6. **Use LIMIT** for large result sets
7. **Consider query execution plans** during development

### Performance
1. **Create indexes** on frequently queried columns
2. **Use EXPLAIN ANALYZE** to understand query performance
3. **Avoid functions in WHERE clauses** on large tables
4. **Use EXISTS instead of IN** for subqueries
5. **Partition large tables** when appropriate
6. **Regular statistics updates** with ANALYZE
7. **Monitor slow query logs**

### Data Integrity
1. **Use transactions** for multi-table operations
2. **Implement proper constraints** and validations
3. **Handle NULL values** explicitly
4. **Use appropriate data types** for storage efficiency
5. **Implement audit trails** for critical data
6. **Regular backup strategies**
7. **Test rollback procedures**

---

## Common Mistakes

❌ **Using inefficient subqueries**
```sql
-- Inefficient
SELECT * FROM products 
WHERE id IN (SELECT product_id FROM order_items);
```
✅ **Use EXISTS or JOINs**
```sql
-- Efficient
SELECT DISTINCT p.* FROM products p
JOIN order_items oi ON p.id = oi.product_id;
```

❌ **Missing indexes on JOIN columns**
```sql
-- Without proper indexes, this will be slow
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.order_date >= '2024-01-01';
```
✅ **Create appropriate indexes**
```sql
CREATE INDEX idx_orders_customer_date ON orders (customer_id, order_date);
CREATE INDEX idx_customers_id ON customers (id);
```

❌ **Using functions in WHERE clauses**
```sql
-- This prevents index usage
SELECT * FROM orders 
WHERE EXTRACT(YEAR FROM order_date) = 2024;
```
✅ **Use range conditions**
```sql
-- This can use indexes
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01';
```

---

## Practice Projects

### Project 1: Sales Analytics System
Build comprehensive sales reporting:
- Monthly/quarterly sales reports
- Product performance analysis
- Customer segmentation
- Trend analysis with forecasting
- Real-time dashboard queries

### Project 2: Inventory Management
Create inventory tracking system:
- Stock level monitoring
- Reorder point calculations
- Supplier performance analysis
- Cost analysis and optimization
- Automated alerts and notifications

### Project 3: Customer Behavior Analysis
Develop customer insights:
- Purchase pattern analysis
- Churn prediction queries
- Recommendation engine data
- Lifetime value calculations
- Cohort analysis

---

## Related Levels
- **Level 1**: Basic SQL execution and simple queries
- **Level 3**: Database administration and management
- **Advanced Topics**: Data warehousing, ETL processes, NoSQL integration

---

## Q&A Section

**Q: When should I use CTEs vs subqueries?**
A: Use CTEs for complex logic, recursive queries, or when you need to reference the same subquery multiple times. Use subqueries for simple, one-time filtering.

**Q: How do I optimize slow queries?**
A: Use EXPLAIN ANALYZE to identify bottlenecks, ensure proper indexing, avoid functions in WHERE clauses, and consider query restructuring.

**Q: What's the difference between RANK() and ROW_NUMBER()?**
A: ROW_NUMBER() assigns unique sequential numbers, while RANK() assigns the same rank to tied values and skips subsequent ranks.

**Q: When should I use window functions vs GROUP BY?**
A: Use window functions when you need row-level details with aggregate information. Use GROUP BY when you only need summarized results.

**Q: How do I handle large result sets efficiently?**
A: Use LIMIT/OFFSET for pagination, implement proper indexing, consider result set streaming, and use appropriate data types.

**Q: What's the best way to handle JSON data in SQL?**
A: Use native JSON functions, create appropriate indexes (GIN for PostgreSQL), validate JSON structure, and consider normalization for frequently queried fields.

**Q: How do I ensure data consistency in complex transactions?**
A: Use appropriate isolation levels, implement proper locking strategies, use savepoints for partial rollbacks, and handle deadlocks gracefully.

**Q: When should I create stored procedures vs application logic?**
A: Use stored procedures for complex business logic that requires multiple database operations, data validation, or when performance is critical.

---

*Continue to Level 3 to learn about database administration, security, backup strategies, and advanced database management techniques.*