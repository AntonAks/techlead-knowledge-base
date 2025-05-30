# Query Optimization

## Overview

**Query Optimization** is the process of improving database query performance by selecting the most efficient execution plan and writing queries that minimize resource usage. It involves both automatic database optimizations and manual query tuning techniques.

## Core Principles

### **Cost-Based Optimization**
Database optimizers evaluate multiple execution plans and choose the one with the lowest estimated cost:
- **CPU Cost**: Processing operations (sorting, filtering, calculations)
- **I/O Cost**: Disk reads and writes
- **Memory Cost**: RAM usage for buffers and temporary storage
- **Network Cost**: Data transfer in distributed systems

### **Statistics-Based Decisions**
Optimizers rely on table and index statistics:
- **Row Counts**: Number of rows in tables
- **Data Distribution**: Histogram of value frequencies
- **Index Selectivity**: How unique index values are
- **Column Cardinality**: Number of distinct values

## Index Optimization

### **Index Selection Strategy**
Choose indexes based on query patterns:

```sql
-- Query needing optimization
SELECT customer_name, order_total 
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date BETWEEN '2024-01-01' AND '2024-12-31'
AND o.status = 'completed'
ORDER BY o.order_date DESC;

-- Optimal indexes
CREATE INDEX idx_orders_date_status ON orders(order_date, status);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_customers_pk ON customers(customer_id); -- Usually exists as PK
```

### **Composite Index Design**
Order columns by selectivity (most selective first):

```sql
-- Good: High selectivity column first
CREATE INDEX idx_orders_optimal ON orders(customer_id, order_date, status);

-- Less optimal: Low selectivity column first
CREATE INDEX idx_orders_suboptimal ON orders(status, customer_id, order_date);

-- Query can use optimal index efficiently
SELECT * FROM orders 
WHERE customer_id = 123 
AND order_date >= '2024-01-01'
AND status = 'pending';
```

### **Covering Indexes**
Include all needed columns in the index to avoid table lookups:

```sql
-- Query requiring multiple columns
SELECT customer_id, order_date, total_amount
FROM orders 
WHERE status = 'shipped' 
AND order_date >= '2024-01-01';

-- Covering index includes all SELECT columns
CREATE INDEX idx_orders_covering 
ON orders(status, order_date) 
INCLUDE (customer_id, total_amount);
```

## Query Structure Optimization

### **WHERE Clause Optimization**

#### **Use Selective Conditions First**
```sql
-- Good: Most selective condition first
SELECT * FROM products 
WHERE category_id = 5  -- High selectivity
AND price > 100        -- Lower selectivity
AND in_stock = true;   -- Lowest selectivity

-- Index should match this order
CREATE INDEX idx_products_optimal ON products(category_id, price, in_stock);
```

#### **Avoid Function Calls on Indexed Columns**
```sql
-- Bad: Function prevents index usage
SELECT * FROM orders 
WHERE YEAR(order_date) = 2024;

-- Good: Range query uses index
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' 
AND order_date < '2025-01-01';
```

#### **Use EXISTS Instead of IN with Subqueries**
```sql
-- Less efficient: IN with subquery
SELECT * FROM customers c
WHERE c.customer_id IN (
    SELECT order_customer_id FROM orders WHERE order_date >= '2024-01-01'
);

-- More efficient: EXISTS
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.customer_id = c.customer_id 
    AND o.order_date >= '2024-01-01'
);
```

### **JOIN Optimization**

#### **JOIN Order Matters**
```sql
-- Query optimizer will reorder, but help with proper structure
SELECT c.customer_name, o.order_total, p.product_name
FROM customers c                    -- Smallest table first
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE c.customer_segment = 'premium'  -- Most selective filter
AND o.order_date >= '2024-01-01';
```

#### **Use Appropriate JOIN Types**
```sql
-- INNER JOIN when you need matching rows only
SELECT c.customer_name, COUNT(o.order_id) as order_count
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

-- LEFT JOIN when you need all left table rows
SELECT c.customer_name, COALESCE(COUNT(o.order_id), 0) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

### **Subquery Optimization**

#### **Correlated vs Non-Correlated Subqueries**
```sql
-- Correlated subquery (runs for each outer row)
SELECT customer_name 
FROM customers c
WHERE (
    SELECT COUNT(*) FROM orders o 
    WHERE o.customer_id = c.customer_id
) > 5;

-- Better: JOIN approach
SELECT DISTINCT c.customer_name
FROM customers c
JOIN (
    SELECT customer_id, COUNT(*) as order_count
    FROM orders 
    GROUP BY customer_id
    HAVING COUNT(*) > 5
) o ON c.customer_id = o.customer_id;
```

#### **Window Functions vs Subqueries**
```sql
-- Subquery approach (less efficient)
SELECT customer_id, order_total,
    (SELECT AVG(order_total) FROM orders o2 
     WHERE o2.customer_id = o1.customer_id) as avg_order
FROM orders o1;

-- Window function approach (more efficient)
SELECT customer_id, order_total,
    AVG(order_total) OVER (PARTITION BY customer_id) as avg_order
FROM orders;
```

## Execution Plan Analysis

### **Common Plan Operations**

#### **Table Scan vs Index Scan**
```sql
-- Table scan (avoid for large tables)
EXPLAIN SELECT * FROM orders WHERE order_notes LIKE '%urgent%';
-- Result: Seq Scan on orders (cost=0.00..1500.00)

-- Index scan (preferred)
CREATE INDEX idx_orders_notes ON orders USING gin(to_tsvector('english', order_notes));
EXPLAIN SELECT * FROM orders WHERE to_tsvector('english', order_notes) @@ 'urgent';
-- Result: Bitmap Index Scan using idx_orders_notes
```

#### **Nested Loop vs Hash Join vs Merge Join**
```sql
-- Small tables: Nested Loop Join
SELECT * FROM customers c 
JOIN orders o ON c.customer_id = o.customer_id 
WHERE c.customer_id = 123;

-- Medium tables: Hash Join
SELECT c.customer_name, COUNT(*)
FROM customers c 
JOIN orders o ON c.customer_id = o.customer_id 
GROUP BY c.customer_id, c.customer_name;

-- Large sorted tables: Merge Join
SELECT c.customer_name, o.order_date
FROM customers c 
JOIN orders o ON c.customer_id = o.customer_id 
ORDER BY c.customer_id, o.order_date;
```

## Performance Patterns

### **Pagination Optimization**

#### **OFFSET/LIMIT Problems**
```sql
-- Bad: OFFSET becomes slow for large offsets
SELECT * FROM products 
ORDER BY product_id 
LIMIT 20 OFFSET 100000;  -- Scans and discards 100k rows

-- Better: Cursor-based pagination
SELECT * FROM products 
WHERE product_id > 100020  -- Last ID from previous page
ORDER BY product_id 
LIMIT 20;
```

#### **Keyset Pagination**
```sql
-- Most efficient for large datasets
SELECT * FROM orders 
WHERE (order_date, order_id) > ('2024-06-15', 12345)
ORDER BY order_date, order_id 
LIMIT 20;

-- Requires composite index
CREATE INDEX idx_orders_pagination ON orders(order_date, order_id);
```

### **Aggregation Optimization**

#### **Partial Aggregation**
```sql
-- Slow: Aggregating large dataset
SELECT customer_id, SUM(order_total) 
FROM orders 
WHERE order_date >= '2024-01-01'
GROUP BY customer_id;

-- Faster: Pre-aggregated summary tables
SELECT customer_id, SUM(monthly_total)
FROM monthly_customer_summary 
WHERE summary_month >= '2024-01'
GROUP BY customer_id;
```

#### **Materialized Views for Complex Aggregations**
```sql
-- Create materialized view for expensive calculations
CREATE MATERIALIZED VIEW customer_metrics AS
SELECT 
    customer_id,
    COUNT(*) as total_orders,
    SUM(order_total) as total_spent,
    AVG(order_total) as avg_order_value,
    MAX(order_date) as last_order_date
FROM orders 
GROUP BY customer_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW customer_metrics;

-- Fast queries using materialized view
SELECT * FROM customer_metrics 
WHERE total_spent > 10000;
```

## Query Rewriting Techniques

### **Predicate Pushdown**
```sql
-- Before: Filter after JOIN
SELECT c.customer_name, o.order_total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2024-01-01';

-- Optimized: Filter in subquery
SELECT c.customer_name, recent_orders.order_total
FROM customers c
JOIN (
    SELECT customer_id, order_total 
    FROM orders 
    WHERE order_date >= '2024-01-01'  -- Filter early
) recent_orders ON c.customer_id = recent_orders.customer_id;
```

### **Union vs Union All**
```sql
-- UNION removes duplicates (expensive)
SELECT customer_id FROM orders WHERE order_date = '2024-01-01'
UNION
SELECT customer_id FROM orders WHERE order_date = '2024-01-02';

-- UNION ALL keeps duplicates (faster when duplicates are acceptable)
SELECT customer_id FROM orders WHERE order_date = '2024-01-01'
UNION ALL
SELECT customer_id FROM orders WHERE order_date = '2024-01-02';
```

### **Conditional Aggregation**
```sql
-- Multiple queries approach
SELECT COUNT(*) FROM orders WHERE status = 'pending';
SELECT COUNT(*) FROM orders WHERE status = 'shipped';
SELECT COUNT(*) FROM orders WHERE status = 'delivered';

-- Single query with conditional aggregation
SELECT 
    COUNT(CASE WHEN status = 'pending' THEN 1 END) as pending_count,
    COUNT(CASE WHEN status = 'shipped' THEN 1 END) as shipped_count,
    COUNT(CASE WHEN status = 'delivered' THEN 1 END) as delivered_count
FROM orders;
```

## Database-Specific Optimizations

### **PostgreSQL**
```sql
-- Partial indexes for filtered queries
CREATE INDEX idx_orders_pending ON orders(customer_id) 
WHERE status = 'pending';

-- Expression indexes for computed values
CREATE INDEX idx_customers_email_lower ON customers(LOWER(email));

-- GIN indexes for array and JSON operations
CREATE INDEX idx_products_tags ON products USING gin(tags);
```

### **MySQL**
```sql
-- Prefix indexes for long strings
CREATE INDEX idx_customers_email_prefix ON customers(email(10));

-- Covering indexes with INCLUDE equivalent
CREATE INDEX idx_orders_covering ON orders(customer_id, order_date, status, order_total);

-- Query cache optimization (MySQL 5.7 and earlier)
SELECT SQL_CACHE customer_name FROM customers WHERE customer_id = 123;
```

### **SQL Server**
```sql
-- Columnstore indexes for analytics
CREATE NONCLUSTERED COLUMNSTORE INDEX idx_orders_columnstore
ON orders (customer_id, order_date, order_total, status);

-- Filtered indexes
CREATE INDEX idx_orders_active ON orders(customer_id)
WHERE status IN ('pending', 'processing');

-- Include columns in indexes
CREATE INDEX idx_orders_covering ON orders(customer_id, order_date)
INCLUDE (order_total, status);
```

## Monitoring and Diagnostics

### **Query Performance Metrics**
```sql
-- PostgreSQL: pg_stat_statements
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;

-- MySQL: Performance Schema
SELECT 
    sql_text,
    exec_count,
    total_latency,
    avg_latency
FROM performance_schema.events_statements_summary_by_digest
ORDER BY total_latency DESC 
LIMIT 10;
```

### **Index Usage Analysis**
```sql
-- PostgreSQL: Check index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes 
ORDER BY idx_scan ASC;  -- Low usage indexes

-- MySQL: Index usage
SELECT 
    object_schema,
    object_name,
    index_name,
    count_star as usage_count
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema = 'your_database'
ORDER BY count_star ASC;
```

## Best Practices

### **Query Design**
- **Limit Result Sets**: Use LIMIT/TOP for large datasets
- **Select Specific Columns**: Avoid SELECT * in production queries
- **Use Appropriate Data Types**: Smaller types reduce memory usage
- **Batch Operations**: Group multiple operations when possible

### **Index Strategy**
- **Monitor Index Usage**: Remove unused indexes
- **Composite Index Order**: Most selective columns first
- **Covering Indexes**: Include frequently accessed columns
- **Regular Maintenance**: Update statistics and rebuild indexes

### **Testing and Monitoring**
- **Performance Baselines**: Establish performance benchmarks
- **Regular Analysis**: Monitor slow query logs
- **Load Testing**: Test queries under realistic load
- **Continuous Optimization**: Regular review and tuning

## Common Anti-Patterns

### **Query Anti-Patterns**
```sql
-- Anti-pattern: SELECT *
SELECT * FROM orders o 
JOIN customers c ON o.customer_id = c.customer_id;

-- Better: Select only needed columns
SELECT o.order_id, o.order_total, c.customer_name
FROM orders o 
JOIN customers c ON o.customer_id = c.customer_id;

-- Anti-pattern: OR conditions
SELECT * FROM products 
WHERE category_id = 1 OR category_id = 2 OR category_id = 3;

-- Better: IN clause
SELECT * FROM products 
WHERE category_id IN (1, 2, 3);

-- Anti-pattern: Functions in WHERE clause
SELECT * FROM orders 
WHERE UPPER(status) = 'PENDING';

-- Better: Store data in consistent case
SELECT * FROM orders 
WHERE status = 'PENDING';
```

### **Index Anti-Patterns**
- **Too Many Indexes**: Slows down INSERT/UPDATE operations
- **Redundant Indexes**: Multiple indexes covering same columns
- **Unused Indexes**: Indexes that are never used by queries
- **Missing Statistics**: Outdated statistics leading to poor plans

## Advanced Optimization Techniques

### **Partitioning**
```sql
-- Range partitioning by date
CREATE TABLE orders_2024 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Hash partitioning for even distribution
CREATE TABLE customers_0 PARTITION OF customers
FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```

### **Query Hints (Use Sparingly)**
```sql
-- PostgreSQL: Force specific join order
SELECT /*+ Leading(c o) */ c.customer_name, o.order_total
FROM customers c 
JOIN orders o ON c.customer_id = o.customer_id;

-- SQL Server: Force index usage
SELECT customer_name 
FROM customers WITH (INDEX(idx_customers_name))
WHERE customer_name LIKE 'John%';
```

### **Parallel Query Execution**
```sql
-- PostgreSQL: Enable parallel execution
SET max_parallel_workers_per_gather = 4;

-- SQL Server: Parallel query hint
SELECT customer_id, SUM(order_total)
FROM orders
GROUP BY customer_id
OPTION (MAXDOP 4);
```

## Summary

Query optimization is an iterative process that combines understanding of database internals, proper indexing strategies, and careful query design. Success requires continuous monitoring, testing, and refinement based on actual usage patterns.

**Key Principles:**
- **Measure First**: Use EXPLAIN plans and performance metrics
- **Index Strategically**: Create indexes based on query patterns
- **Write Efficient Queries**: Use proper JOIN techniques and filtering
- **Monitor Continuously**: Track performance over time
- **Test Thoroughly**: Validate optimizations under realistic load

**Optimization Process:**
1. **Identify Slow Queries**: Use monitoring tools and logs
2. **Analyze Execution Plans**: Understand how queries execute
3. **Apply Optimizations**: Indexes, query rewrites, schema changes
4. **Measure Improvements**: Verify performance gains
5. **Monitor Ongoing**: Watch for performance regressions

**Success Factors:**
- **Understanding Workload**: Know your read vs write patterns
- **Proper Indexing**: Right indexes for your query patterns
- **Database Knowledge**: Understand your database's optimizer
- **Regular Maintenance**: Keep statistics updated, rebuild indexes
- **Performance Culture**: Make optimization part of development process

Query optimization is both an art and a science, requiring technical knowledge, analytical thinking, and practical experience to achieve optimal database performance.