# Database Views

## What are Views?

A **view** is a virtual table based on the result of a SQL query. Views contain rows and columns just like regular tables, but they don't store data physically—they dynamically generate results from underlying tables when queried.

## Types of Views

### **Simple Views**
Based on a single table with basic SELECT operations.

```sql
-- Customer contact information view
CREATE VIEW customer_contacts AS
SELECT 
    customer_id,
    first_name,
    last_name,
    email,
    phone
FROM customers
WHERE status = 'active';

-- Usage
SELECT * FROM customer_contacts WHERE last_name LIKE 'Smith%';
```

### **Complex Views**
Involve multiple tables, joins, aggregations, or complex logic.

```sql
-- Order summary view with joins and calculations
CREATE VIEW order_summary AS
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email as customer_email,
    COUNT(oi.order_item_id) as item_count,
    SUM(oi.quantity * oi.unit_price) as subtotal,
    SUM(oi.quantity * oi.unit_price) * 0.08 as tax_amount,
    SUM(oi.quantity * oi.unit_price) * 1.08 as total_amount,
    o.status
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_date, c.customer_name, c.email, o.status;

-- Usage
SELECT * FROM order_summary 
WHERE order_date >= '2024-01-01' 
AND total_amount > 100
ORDER BY total_amount DESC;
```

### **Materialized Views**
Physically store query results for improved performance.

```sql
-- PostgreSQL materialized view
CREATE MATERIALIZED VIEW monthly_sales_report AS
SELECT 
    DATE_TRUNC('month', order_date) as month,
    COUNT(*) as order_count,
    SUM(total_amount) as total_revenue,
    AVG(total_amount) as avg_order_value,
    COUNT(DISTINCT customer_id) as unique_customers
FROM orders 
WHERE order_date >= '2023-01-01'
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- Refresh when needed
REFRESH MATERIALIZED VIEW monthly_sales_report;

-- SQL Server indexed view (similar concept)
CREATE VIEW dbo.product_sales_summary
WITH SCHEMABINDING AS
SELECT 
    p.product_id,
    p.product_name,
    SUM(oi.quantity) as total_quantity_sold,
    COUNT_BIG(*) as order_count
FROM dbo.products p
JOIN dbo.order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name;

-- Create clustered index to make it materialized
CREATE UNIQUE CLUSTERED INDEX idx_product_sales 
ON dbo.product_sales_summary(product_id);
```

## View Benefits

### **Data Security and Access Control**
```sql
-- Hide sensitive columns from general users
CREATE VIEW employee_directory AS
SELECT 
    employee_id,
    first_name,
    last_name,
    department,
    job_title,
    work_email,
    work_phone
FROM employees
-- Excludes: salary, ssn, personal_email, address

-- Grant access to view instead of base table
GRANT SELECT ON employee_directory TO hr_staff;
REVOKE SELECT ON employees FROM hr_staff;

-- Row-level security through views
CREATE VIEW my_orders AS
SELECT 
    order_id,
    order_date,
    status,
    total_amount
FROM orders 
WHERE customer_id = USER_ID(); -- Assumes USER_ID() function returns current user's ID
```

### **Data Abstraction and Simplification**
```sql
-- Complex query simplified into a view
CREATE VIEW customer_metrics AS
SELECT 
    c.customer_id,
    c.customer_name,
    c.registration_date,
    COALESCE(stats.total_orders, 0) as total_orders,
    COALESCE(stats.total_spent, 0) as total_spent,
    COALESCE(stats.avg_order_value, 0) as avg_order_value,
    stats.last_order_date,
    CASE 
        WHEN stats.last_order_date >= CURRENT_DATE - INTERVAL '30 days' THEN 'Active'
        WHEN stats.last_order_date >= CURRENT_DATE - INTERVAL '90 days' THEN 'Recent'
        WHEN stats.last_order_date IS NOT NULL THEN 'Inactive'
        ELSE 'Never Ordered'
    END as customer_status
FROM customers c
LEFT JOIN (
    SELECT 
        customer_id,
        COUNT(*) as total_orders,
        SUM(total_amount) as total_spent,
        AVG(total_amount) as avg_order_value,
        MAX(order_date) as last_order_date
    FROM orders 
    GROUP BY customer_id
) stats ON c.customer_id = stats.customer_id;

-- Simple usage instead of complex JOIN
SELECT customer_name, customer_status, total_spent 
FROM customer_metrics 
WHERE customer_status = 'Active'
ORDER BY total_spent DESC;
```

### **Legacy System Integration**
```sql
-- Provide consistent interface despite schema changes
CREATE VIEW legacy_products AS
SELECT 
    product_id as id,
    product_name as name,
    product_description as description,
    price as unit_price,
    CASE 
        WHEN stock_quantity > 0 THEN 'Y'
        ELSE 'N'
    END as in_stock_flag,
    category_name as category
FROM products p
JOIN categories c ON p.category_id = c.category_id;

-- Legacy applications continue working unchanged
-- while underlying schema evolves
```

## Updatable Views

### **Simple Updatable Views**
```sql
-- View that allows INSERT, UPDATE, DELETE
CREATE VIEW active_customers AS
SELECT 
    customer_id,
    customer_name,
    email,
    status
FROM customers 
WHERE status = 'active';

-- These operations work on simple views
INSERT INTO active_customers (customer_name, email, status) 
VALUES ('John Doe', 'john@example.com', 'active');

UPDATE active_customers 
SET email = 'john.doe@example.com' 
WHERE customer_id = 123;

DELETE FROM active_customers 
WHERE customer_id = 123;
```

### **Complex Views with INSTEAD OF Triggers**
```sql
-- SQL Server: Complex view with INSTEAD OF trigger
CREATE VIEW customer_order_summary AS
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    c.status,
    COUNT(o.order_id) as order_count,
    COALESCE(SUM(o.total_amount), 0) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.email, c.status;

-- Make the view updatable with INSTEAD OF trigger
CREATE TRIGGER tr_customer_order_summary_update
ON customer_order_summary
INSTEAD OF UPDATE
AS
BEGIN
    UPDATE customers 
    SET customer_name = i.customer_name,
        email = i.email,
        status = i.status
    FROM customers c
    INNER JOIN inserted i ON c.customer_id = i.customer_id;
END;

-- Now updates work on the complex view
UPDATE customer_order_summary 
SET customer_name = 'Updated Name',
    email = 'updated@example.com'
WHERE customer_id = 123;
```

## Performance Considerations

### **View Query Optimization**
```sql
-- Bad: Non-selective view that returns too much data
CREATE VIEW all_order_details AS
SELECT 
    o.*,
    c.*,
    oi.*,
    p.*
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;

-- Query becomes expensive
SELECT * FROM all_order_details WHERE order_date = '2024-01-15';

-- Better: Selective view with proper filtering
CREATE VIEW recent_order_details AS
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '30 days';
```

### **Indexed Views for Performance**
```sql
-- SQL Server: Create indexed view for better performance
CREATE VIEW dbo.customer_order_totals
WITH SCHEMABINDING AS
SELECT 
    c.customer_id,
    COUNT_BIG(*) as order_count,
    SUM(o.total_amount) as total_spent,
    AVG(o.total_amount) as avg_order_value
FROM dbo.customers c
JOIN dbo.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;

-- Create clustered index to materialize the view
CREATE UNIQUE CLUSTERED INDEX idx_customer_order_totals
ON dbo.customer_order_totals(customer_id);

-- Queries automatically use the indexed view
SELECT customer_id, total_spent 
FROM customer_order_totals 
WHERE total_spent > 1000;
```

## Advanced View Patterns

### **Recursive Views (Common Table Expressions)**
```sql
-- PostgreSQL/SQL Server: Recursive view for hierarchical data
CREATE VIEW employee_hierarchy AS
WITH RECURSIVE hierarchy AS (
    -- Base case: top-level managers
    SELECT 
        employee_id,
        name,
        manager_id,
        job_title,
        1 as level,
        CAST(name AS VARCHAR(1000)) as path
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: subordinates
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        e.job_title,
        h.level + 1,
        CAST(h.path || ' -> ' || e.name AS VARCHAR(1000))
    FROM employees e
    JOIN hierarchy h ON e.manager_id = h.employee_id
)
SELECT * FROM hierarchy;

-- Usage
SELECT * FROM employee_hierarchy 
WHERE level <= 3 
ORDER BY level, path;
```

### **Partitioned Views**
```sql
-- SQL Server: Partition data across multiple tables
-- Historical orders in separate tables by year
CREATE TABLE orders_2023 (
    order_id INT PRIMARY KEY,
    order_date DATE CHECK (YEAR(order_date) = 2023),
    customer_id INT,
    total_amount DECIMAL(10,2)
);

CREATE TABLE orders_2024 (
    order_id INT PRIMARY KEY,
    order_date DATE CHECK (YEAR(order_date) = 2024),
    customer_id INT,
    total_amount DECIMAL(10,2)
);

-- Partitioned view combines all tables
CREATE VIEW all_orders AS
SELECT order_id, order_date, customer_id, total_amount FROM orders_2023
UNION ALL
SELECT order_id, order_date, customer_id, total_amount FROM orders_2024;

-- Queries automatically route to correct table
SELECT * FROM all_orders 
WHERE order_date = '2024-06-15'; -- Only queries orders_2024
```

### **Secure Views with Dynamic Filtering**
```sql
-- PostgreSQL: Row-level security through views
CREATE VIEW user_accessible_orders AS
SELECT 
    o.order_id,
    o.order_date,
    o.total_amount,
    o.status
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE 
    -- Customers see only their orders
    (current_setting('app.user_role') = 'customer' 
     AND c.email = current_setting('app.user_email'))
    OR
    -- Staff see all orders
    (current_setting('app.user_role') IN ('staff', 'admin'));

-- Application sets context before queries
SET app.user_role = 'customer';
SET app.user_email = 'john@example.com';

-- User automatically sees only their data
SELECT * FROM user_accessible_orders;
```

## View Maintenance and Management

### **View Dependencies**
```sql
-- Check view dependencies
-- PostgreSQL
SELECT 
    dependent_ns.nspname as dependent_schema,
    dependent_view.relname as dependent_view,
    source_ns.nspname as source_schema,
    source_table.relname as source_table
FROM pg_depend 
JOIN pg_rewrite ON pg_depend.objid = pg_rewrite.oid 
JOIN pg_class as dependent_view ON pg_rewrite.ev_class = dependent_view.oid 
JOIN pg_class as source_table ON pg_depend.refobjid = source_table.oid 
JOIN pg_namespace dependent_ns ON dependent_ns.oid = dependent_view.relnamespace
JOIN pg_namespace source_ns ON source_ns.oid = source_table.relnamespace
WHERE source_table.relname = 'customers';

-- SQL Server
SELECT 
    o.name as view_name,
    referenced_entity_name as table_name
FROM sys.sql_expression_dependencies d
JOIN sys.objects o ON d.referencing_id = o.object_id
WHERE o.type = 'V' -- Views only
AND referenced_entity_name = 'customers';
```

### **View Performance Analysis**
```sql
-- PostgreSQL: Analyze view performance
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM customer_metrics 
WHERE customer_status = 'Active';

-- Look for:
-- - Sequential scans on large tables
-- - Missing indexes on joined columns
-- - Expensive aggregations
-- - Unnecessary data retrieval

-- MySQL: Check view execution
EXPLAIN FORMAT=JSON
SELECT * FROM order_summary 
WHERE order_date >= '2024-01-01';
```

## Common View Anti-Patterns

### **Views on Views (Excessive Layering)**
```sql
-- Anti-pattern: Multiple layers of views
CREATE VIEW base_customer_data AS
SELECT customer_id, name, email, status FROM customers;

CREATE VIEW active_customers AS
SELECT * FROM base_customer_data WHERE status = 'active';

CREATE VIEW active_customer_summary AS
SELECT 
    ac.*,
    COUNT(o.order_id) as order_count
FROM active_customers ac
LEFT JOIN orders o ON ac.customer_id = o.customer_id
GROUP BY ac.customer_id, ac.name, ac.email, ac.status;

-- Better: Single, well-designed view
CREATE VIEW customer_summary AS
SELECT 
    c.customer_id,
    c.name,
    c.email,
    c.status,
    COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.status = 'active'
GROUP BY c.customer_id, c.name, c.email, c.status;
```

### **SELECT * in Views**
```sql
-- Anti-pattern: SELECT * makes views fragile
CREATE VIEW order_details AS
SELECT 
    o.*,  -- Bad: includes all columns, breaks if table structure changes
    c.*   -- Bad: column name conflicts possible
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Better: Explicit column selection
CREATE VIEW order_details AS
SELECT 
    o.order_id,
    o.order_date,
    o.total_amount,
    o.status as order_status,
    c.customer_name,
    c.email as customer_email
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

## Best Practices

### **View Design Principles**
- **Single Responsibility**: Each view should serve a specific purpose
- **Explicit Columns**: Always specify column names, avoid SELECT *
- **Meaningful Names**: Use descriptive names that indicate view purpose
- **Documentation**: Comment complex views with their business purpose
- **Performance Testing**: Test view performance with realistic data volumes

### **Security Best Practices**
```sql
-- Use views to implement column-level security
CREATE VIEW public_employee_info AS
SELECT 
    employee_id,
    first_name,
    last_name,
    department,
    job_title
    -- Excludes: salary, ssn, personal_address
FROM employees;

-- Use views for row-level security
CREATE VIEW regional_sales AS
SELECT 
    sale_id,
    sale_date,
    amount,
    customer_id
FROM sales s
JOIN sales_territories st ON s.territory_id = st.territory_id
WHERE st.region = USER_REGION(); -- Assumes USER_REGION() function
```

### **Performance Guidelines**
- **Index Underlying Tables**: Ensure base tables have appropriate indexes
- **Limit Complexity**: Avoid overly complex views with multiple aggregations
- **Consider Materialized Views**: For expensive calculations used frequently
- **Monitor Usage**: Track which views are used and their performance impact
- **Regular Maintenance**: Update statistics and refresh materialized views

## View Testing and Validation

### **View Testing Strategies**
```sql
-- Test view logic against known data
-- 1. Insert test data
INSERT INTO customers VALUES (999, 'Test Customer', 'test@example.com', 'active');
INSERT INTO orders VALUES (999, 999, '2024-01-15', 'completed', 150.00);

-- 2. Test view output
SELECT * FROM customer_metrics WHERE customer_id = 999;
-- Verify calculations: total_orders = 1, total_spent = 150.00

-- 3. Test edge cases
-- Customer with no orders
INSERT INTO customers VALUES (998, 'No Orders', 'none@example.com', 'active');
SELECT * FROM customer_metrics WHERE customer_id = 998;
-- Verify: total_orders = 0, total_spent = 0

-- 4. Clean up test data
DELETE FROM orders WHERE customer_id IN (998, 999);
DELETE FROM customers WHERE customer_id IN (998, 999);
```

## Modern Alternatives and Considerations

### **Application-Level Views**
```python
# Python/Django: Database views vs ORM queries
# Database view approach
class CustomerMetrics(models.Model):
    customer_id = models.IntegerField(primary_key=True)
    customer_name = models.CharField(max_length=100)
    total_orders = models.IntegerField()
    total_spent = models.DecimalField(max_digits=10, decimal_places=2)
    
    class Meta:
        managed = False  # This is a database view
        db_table = 'customer_metrics'

# ORM query approach (more flexible)
def get_customer_metrics():
    return Customer.objects.annotate(
        total_orders=Count('orders'),
        total_spent=Sum('orders__total_amount')
    ).filter(status='active')
```

### **GraphQL and API Views**
```javascript
// GraphQL resolver as "application view"
const resolvers = {
  Query: {
    customerMetrics: async (parent, args, context) => {
      return await context.db.customers.findMany({
        where: { status: 'active' },
        include: {
          orders: {
            select: {
              totalAmount: true
            }
          }
        }
      }).map(customer => ({
        customerId: customer.id,
        customerName: customer.name,
        totalOrders: customer.orders.length,
        totalSpent: customer.orders.reduce((sum, order) => sum + order.totalAmount, 0)
      }));
    }
  }
};
```

## Summary

Database views are powerful tools for data abstraction, security, and simplification. They provide a layer between applications and raw tables, enabling better maintainability and security while potentially improving performance through materialization.

**Key Benefits:**
- **Data Security**: Hide sensitive columns and implement row-level security
- **Abstraction**: Simplify complex queries for application developers
- **Maintainability**: Centralize business logic and provide stable interfaces
- **Performance**: Materialized views can significantly improve query performance

**Best Practices:**
- **Design Thoughtfully**: Single purpose, explicit columns, meaningful names
- **Monitor Performance**: Ensure views don't become performance bottlenecks
- **Manage Dependencies**: Track and document view relationships
- **Test Thoroughly**: Validate view logic and edge cases
- **Consider Alternatives**: Balance database views vs application-level abstraction

**When to Use Views:**
- **Security Requirements**: Column or row-level access control needed
- **Complex Queries**: Frequently used complex joins or calculations
- **Legacy Integration**: Stable interface for changing schemas
- **Reporting**: Pre-aggregated data for dashboards and reports

**When to Avoid Views:**
- **Simple Operations**: Basic CRUD operations on single tables
- **Highly Dynamic Logic**: Business rules that change frequently
- **Performance Critical**: When view overhead impacts critical paths
- **Cross-Database**: When data spans multiple database systems

Views remain an essential database feature for creating maintainable, secure, and performant data access layers, especially when combined with modern application architectures and proper performance monitoring.