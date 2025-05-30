# Normalization & Denormalization

## Overview

**Normalization** is the process of organizing data in a database to reduce redundancy and improve data integrity. **Denormalization** is the deliberate introduction of redundancy to improve query performance. Both are important database design techniques with specific use cases and trade-offs.

## Database Normalization

### **What is Normalization?**
Normalization is a systematic approach to decomposing tables to eliminate data redundancy and undesirable characteristics like insertion, update, and deletion anomalies.

### **Goals of Normalization**
- **Eliminate Redundancy**: Remove duplicate data storage
- **Ensure Data Integrity**: Maintain consistent and accurate data
- **Reduce Storage Space**: Minimize disk space usage
- **Prevent Anomalies**: Avoid insertion, update, and deletion problems
- **Simplify Maintenance**: Make data updates easier and safer

## Normal Forms

### **First Normal Form (1NF)**
Each table cell contains only atomic (indivisible) values, and each column contains values of a single type.

#### **Violations of 1NF**
```sql
-- BAD: Multiple values in single cell
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    phone_numbers VARCHAR(255)  -- "555-1234, 555-5678, 555-9999"
);
```

#### **1NF Compliant**
```sql
-- GOOD: Atomic values only
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employee_phones (
    employee_id INT,
    phone_number VARCHAR(20),
    phone_type VARCHAR(20),  -- 'home', 'work', 'mobile'
    PRIMARY KEY (employee_id, phone_number),
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);
```

### **Second Normal Form (2NF)**
Must be in 1NF and all non-key attributes must be fully functionally dependent on the entire primary key.

#### **Violations of 2NF**
```sql
-- BAD: Partial dependency on composite key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),    -- Depends only on product_id
    product_price DECIMAL(10,2),  -- Depends only on product_id
    quantity INT,                 -- Depends on both order_id and product_id
    PRIMARY KEY (order_id, product_id)
);
```

#### **2NF Compliant**
```sql
-- GOOD: Separate product information
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### **Third Normal Form (3NF)**
Must be in 2NF and all attributes must be directly dependent on the primary key (no transitive dependencies).

#### **Violations of 3NF**
```sql
-- BAD: Transitive dependency
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    department_name VARCHAR(100),    -- Depends on department_id, not employee_id
    department_budget DECIMAL(12,2)  -- Depends on department_id, not employee_id
);
```

#### **3NF Compliant**
```sql
-- GOOD: Remove transitive dependencies
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100),
    department_budget DECIMAL(12,2)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

### **Boyce-Codd Normal Form (BCNF)**
Stronger version of 3NF where every determinant must be a candidate key.

#### **Example BCNF Issue**
```sql
-- Student can have multiple majors, advisors are specific to majors
-- Problem: advisor determines major, but (student, major) is the key
CREATE TABLE student_advisors (
    student_id INT,
    major VARCHAR(50),
    advisor VARCHAR(100),
    PRIMARY KEY (student_id, major)
);
-- Issue: advisor -> major (advisor determines major)
-- But advisor is not a candidate key
```

#### **BCNF Solution**
```sql
-- Split into two tables
CREATE TABLE advisor_majors (
    advisor VARCHAR(100) PRIMARY KEY,
    major VARCHAR(50)
);

CREATE TABLE student_advisors (
    student_id INT,
    advisor VARCHAR(100),
    PRIMARY KEY (student_id, advisor),
    FOREIGN KEY (advisor) REFERENCES advisor_majors(advisor)
);
```

### **Fourth Normal Form (4NF)**
Must be in BCNF and have no multi-valued dependencies.

#### **4NF Example**
```sql
-- BAD: Multi-valued dependencies
CREATE TABLE student_courses_hobbies (
    student_id INT,
    course VARCHAR(50),
    hobby VARCHAR(50),
    PRIMARY KEY (student_id, course, hobby)
);
-- Problem: courses and hobbies are independent of each other
```

#### **4NF Solution**
```sql
-- Split into separate tables
CREATE TABLE student_courses (
    student_id INT,
    course VARCHAR(50),
    PRIMARY KEY (student_id, course)
);

CREATE TABLE student_hobbies (
    student_id INT,
    hobby VARCHAR(50),
    PRIMARY KEY (student_id, hobby)
);
```

## Benefits of Normalization

### **Data Integrity**
- **Consistency**: Single source of truth for each piece of data
- **Referential Integrity**: Foreign key constraints ensure valid relationships
- **No Redundancy**: Eliminates duplicate data storage
- **Atomic Updates**: Changes made in one place automatically reflect everywhere

### **Storage Efficiency**
- **Reduced Storage**: Less disk space due to eliminated redundancy
- **Faster Backups**: Smaller database size means faster backup operations
- **Lower Costs**: Reduced storage requirements decrease costs

### **Maintenance Benefits**
- **Easier Updates**: Change data in one place instead of multiple locations
- **Reduced Errors**: Less chance of inconsistent data
- **Clear Structure**: Logical, organized database design
- **Flexibility**: Easier to modify schema as requirements change

## Drawbacks of Normalization

### **Performance Issues**
- **Complex Queries**: Require multiple JOINs to retrieve related data
- **Query Performance**: JOINs can be expensive for large datasets
- **Increased Complexity**: More tables make queries more complex

### **Example Performance Impact**
```sql
-- Normalized: Requires multiple JOINs
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity,
    p.price * oi.quantity as total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= '2024-01-01';

-- vs. Denormalized: Simple query
SELECT 
    order_id,
    customer_name,
    product_name,
    quantity,
    total_amount
FROM order_summary
WHERE order_date >= '2024-01-01';
```

## Denormalization

### **What is Denormalization?**
Denormalization is the deliberate introduction of redundancy into a database design to improve query performance, often at the cost of storage space and data consistency complexity.

### **When to Denormalize**
- **Read-Heavy Workloads**: Frequent queries with complex JOINs
- **Performance Requirements**: Query response time is critical
- **Reporting Systems**: Data warehouses and analytics databases
- **Scalability Needs**: When JOINs become performance bottlenecks
- **Specific Use Cases**: Search indexes, caching layers

### **Types of Denormalization**

#### **Storing Derived Data**
```sql
-- Store calculated values
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    subtotal DECIMAL(10,2),
    tax_amount DECIMAL(10,2),
    total_amount DECIMAL(10,2),  -- Derived: subtotal + tax_amount
    item_count INT               -- Derived: COUNT of order items
);
```

#### **Duplicating Reference Data**
```sql
-- Store frequently accessed reference data
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),    -- Duplicated from products table
    product_category VARCHAR(50), -- Duplicated from products table
    unit_price DECIMAL(10,2),     -- Duplicated from products table
    quantity INT,
    line_total DECIMAL(10,2)
);
```

#### **Flattening Hierarchies**
```sql
-- Flatten category hierarchy for faster queries
CREATE TABLE products_flat (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_l1 VARCHAR(50),  -- Top level category
    category_l2 VARCHAR(50),  -- Second level category
    category_l3 VARCHAR(50),  -- Third level category
    full_category_path VARCHAR(200) -- "Electronics > Computers > Laptops"
);
```

## Denormalization Strategies

### **Materialized Views**
Pre-computed views that store query results physically.

```sql
-- Create materialized view for reporting
CREATE MATERIALIZED VIEW monthly_sales_summary AS
SELECT 
    DATE_TRUNC('month', order_date) as month,
    customer_id,
    COUNT(*) as order_count,
    SUM(total_amount) as total_sales,
    AVG(total_amount) as avg_order_value
FROM orders 
WHERE order_date >= '2023-01-01'
GROUP BY DATE_TRUNC('month', order_date), customer_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW monthly_sales_summary;
```

### **Summary Tables**
Pre-aggregated data for reporting and analytics.

```sql
-- Daily sales summary table
CREATE TABLE daily_sales_summary (
    summary_date DATE PRIMARY KEY,
    total_orders INT,
    total_revenue DECIMAL(12,2),
    avg_order_value DECIMAL(10,2),
    unique_customers INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Populate via scheduled job
INSERT INTO daily_sales_summary (
    summary_date, total_orders, total_revenue, avg_order_value, unique_customers
)
SELECT 
    DATE(order_date),
    COUNT(*),
    SUM(total_amount),
    AVG(total_amount),
    COUNT(DISTINCT customer_id)
FROM orders 
WHERE DATE(order_date) = CURRENT_DATE - INTERVAL 1 DAY
GROUP BY DATE(order_date);
```

### **Search Tables**
Denormalized tables optimized for search functionality.

```sql
-- Product search table with denormalized data
CREATE TABLE product_search (
    product_id INT PRIMARY KEY,
    search_text TEXT,  -- Combined searchable fields
    category_path VARCHAR(200),
    brand VARCHAR(100),
    price DECIMAL(10,2),
    rating DECIMAL(3,2),
    review_count INT,
    in_stock BOOLEAN,
    FULLTEXT(search_text)
);

-- Populate from normalized tables
INSERT INTO product_search 
SELECT 
    p.product_id,
    CONCAT(p.name, ' ', p.description, ' ', b.brand_name, ' ', c.category_name) as search_text,
    c.full_path as category_path,
    b.brand_name,
    p.price,
    COALESCE(r.avg_rating, 0) as rating,
    COALESCE(r.review_count, 0) as review_count,
    p.stock_quantity > 0 as in_stock
FROM products p
LEFT JOIN brands b ON p.brand_id = b.brand_id
LEFT JOIN categories c ON p.category_id = c.category_id
LEFT JOIN product_ratings r ON p.product_id = r.product_id;
```

## Maintaining Data Consistency

### **Triggers for Automatic Updates**
```sql
-- Update denormalized order total when order items change
CREATE TRIGGER update_order_totals
AFTER INSERT OR UPDATE OR DELETE ON order_items
FOR EACH ROW
BEGIN
    UPDATE orders 
    SET 
        subtotal = (
            SELECT COALESCE(SUM(unit_price * quantity), 0)
            FROM order_items 
            WHERE order_id = COALESCE(NEW.order_id, OLD.order_id)
        ),
        item_count = (
            SELECT COUNT(*)
            FROM order_items 
            WHERE order_id = COALESCE(NEW.order_id, OLD.order_id)
        )
    WHERE order_id = COALESCE(NEW.order_id, OLD.order_id);
END;
```

### **Batch Update Processes**
```sql
-- Scheduled job to update denormalized product data
UPDATE order_items oi
SET 
    product_name = p.name,
    product_category = c.name,
    current_price = p.price
FROM products p
JOIN categories c ON p.category_id = c.category_id
WHERE oi.product_id = p.product_id
AND oi.last_updated < p.last_modified;
```

### **Event-Driven Updates**
```python
# Application-level consistency using events
class OrderService:
    def add_order_item(self, order_id, product_id, quantity):
        # Add the order item
        order_item = OrderItem.create(
            order_id=order_id,
            product_id=product_id,
            quantity=quantity
        )
        
        # Update denormalized order totals
        self.update_order_summary(order_id)
        
        # Publish event for other systems
        EventBus.publish('order_item_added', {
            'order_id': order_id,
            'product_id': product_id,
            'quantity': quantity
        })
    
    def update_order_summary(self, order_id):
        # Update denormalized fields
        order = Order.get(order_id)
        items = OrderItem.filter(order_id=order_id)
        
        order.item_count = len(items)
        order.subtotal = sum(item.unit_price * item.quantity for item in items)
        order.save()
```

## Real-World Examples

### **E-commerce Platform**

#### **Normalized Design**
```sql
-- Highly normalized structure
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    description TEXT,
    category_id INT,
    brand_id INT,
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date TIMESTAMP,
    status VARCHAR(20)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)
);
```

#### **Partially Denormalized for Performance**
```sql
-- Add denormalized fields for common queries
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    customer_name VARCHAR(100),  -- Denormalized from customers
    customer_email VARCHAR(100), -- Denormalized from customers
    order_date TIMESTAMP,
    status VARCHAR(20),
    item_count INT,              -- Calculated field
    subtotal DECIMAL(12,2),      -- Calculated field
    tax_amount DECIMAL(12,2),
    total_amount DECIMAL(12,2)   -- Calculated field
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),   -- Denormalized from products
    product_sku VARCHAR(50),     -- Denormalized from products
    category_name VARCHAR(50),   -- Denormalized from categories
    quantity INT,
    unit_price DECIMAL(10,2),
    line_total DECIMAL(10,2),    -- Calculated field
    PRIMARY KEY (order_id, product_id)
);
```

### **Data Warehouse Design**
```sql
-- Star schema with denormalized fact table
CREATE TABLE sales_fact (
    sale_id BIGINT PRIMARY KEY,
    date_key INT,                -- Reference to date dimension
    product_key INT,             -- Reference to product dimension
    customer_key INT,            -- Reference to customer dimension
    store_key INT,               -- Reference to store dimension
    
    -- Denormalized measures
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(12,2),
    cost_amount DECIMAL(12,2),
    profit_amount DECIMAL(12,2),
    
    -- Denormalized attributes for faster filtering
    product_category VARCHAR(50),
    customer_segment VARCHAR(20),
    store_region VARCHAR(50),
    
    -- Pre-calculated metrics
    is_profitable BOOLEAN,
    margin_percentage DECIMAL(5,2)
);
```

## Decision Framework: When to Normalize vs Denormalize

### **Choose Normalization When**
- **OLTP Systems**: Transaction processing with frequent updates
- **Data Integrity Critical**: Accuracy and consistency paramount
- **Storage Constraints**: Limited storage space
- **Frequent Updates**: Data changes often
- **Complex Business Rules**: Many integrity constraints

### **Choose Denormalization When**
- **OLAP Systems**: Analytics and reporting workloads
- **Read-Heavy Workloads**: Queries far outnumber updates
- **Performance Critical**: Query speed more important than storage
- **Simple Aggregations**: Common summary calculations
- **Scalability Requirements**: Need to avoid complex JOINs

### **Hybrid Approach**
Most real-world systems use a combination:
- **Normalize transactional tables** for data integrity
- **Denormalize reporting tables** for query performance
- **Use materialized views** for complex aggregations
- **Implement caching layers** for frequently accessed data

## Best Practices

### **Normalization Best Practices**
- **Start Normalized**: Begin with a normalized design
- **Document Dependencies**: Clearly identify functional dependencies
- **Use Foreign Keys**: Enforce referential integrity with constraints
- **Regular Reviews**: Periodically review and optimize schema
- **Performance Testing**: Monitor query performance as data grows

### **Denormalization Best Practices**
- **Justify Each Decision**: Document why denormalization is needed
- **Maintain Consistency**: Implement update mechanisms for redundant data
- **Monitor Performance**: Measure actual performance improvements
- **Plan for Maintenance**: Consider ongoing maintenance costs
- **Version Control**: Track schema changes and their rationale

### **General Guidelines**
- **Measure First**: Base decisions on actual performance data
- **Iterate**: Start normalized, denormalize specific pain points
- **Document Trade-offs**: Record decisions and their reasoning
- **Monitor Continuously**: Performance characteristics change over time
- **Consider Alternatives**: Views, indexes, partitioning before denormalization

## Modern Considerations

### **NoSQL and Denormalization**
- **Document Databases**: MongoDB, CouchDB favor denormalized documents
- **Key-Value Stores**: Redis, DynamoDB often store denormalized data
- **Column Families**: Cassandra designed for denormalized wide rows
- **Graph Databases**: Neo4j balances normalization with traversal performance

### **Cloud and Distributed Systems**
- **Microservices**: Each service may have differently normalized data
- **Data Lakes**: Often store denormalized data for analytics
- **Eventual Consistency**: Distributed systems may accept temporary inconsistency
- **CQRS**: Separate read and write models with different normalization levels

### **Modern Tools and Techniques**
- **Materialized Views**: Automatic maintenance of denormalized aggregations
- **Change Data Capture**: Real-time synchronization of denormalized data
- **Event Sourcing**: Store events, materialize different denormalized views
- **In-Memory Databases**: Fast JOINs may reduce need for denormalization

## Summary

Normalization and denormalization are complementary techniques for database design optimization. The key is understanding when and how to apply each approach based on specific use cases, performance requirements, and maintenance considerations.

**Key Principles:**
- **Start Normalized**: Begin with a clean, normalized design
- **Denormalize Strategically**: Only when justified by performance requirements
- **Maintain Consistency**: Implement proper update mechanisms
- **Monitor Performance**: Base decisions on actual measurements
- **Document Decisions**: Record rationale for future reference

**Modern Approach:**
- **Hybrid Designs**: Combine normalized transactional tables with denormalized views
- **Multiple Models**: Different normalization levels for different use cases
- **Automated Maintenance**: Use tools and frameworks to maintain consistency
- **Continuous Optimization**: Regularly review and adjust based on usage patterns

**Success Factors:**
- **Clear Requirements**: Understand read vs write patterns
- **Performance Monitoring**: Measure actual query performance
- **Maintenance Planning**: Consider long-term maintenance costs
- **Team Understanding**: Ensure team understands design decisions and trade-offs

The goal is not to achieve perfect normalization or denormalization, but to find the right balance for your specific application requirements, performance needs, and maintenance capabilities.