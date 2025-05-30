# Stored Procedures, Functions, and Triggers

## Overview

**Stored Procedures**, **Functions**, and **Triggers** are database objects that encapsulate business logic within the database server. They provide powerful capabilities for data processing, validation, and automation while offering performance benefits through reduced network traffic and pre-compiled execution.

## Stored Procedures

### **What are Stored Procedures?**
Pre-compiled SQL statements stored in the database that can accept parameters, execute complex logic, and return results. They encapsulate business logic at the database level.

### **Key Characteristics**
- **Pre-compiled**: Faster execution than dynamic SQL
- **Parameterized**: Accept input and output parameters
- **Reusable**: Called from multiple applications
- **Secure**: Controlled access to data operations
- **Transactional**: Support transaction management

### **Basic Syntax Examples**

#### **PostgreSQL**
```sql
-- Create stored procedure
CREATE OR REPLACE PROCEDURE process_order(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    OUT p_order_id INT
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Insert order
    INSERT INTO orders (customer_id, order_date, status)
    VALUES (p_customer_id, CURRENT_DATE, 'pending')
    RETURNING order_id INTO p_order_id;
    
    -- Insert order items
    INSERT INTO order_items (order_id, product_id, quantity, unit_price)
    SELECT p_order_id, p_product_id, p_quantity, price
    FROM products WHERE product_id = p_product_id;
    
    -- Update inventory
    UPDATE products 
    SET stock_quantity = stock_quantity - p_quantity
    WHERE product_id = p_product_id;
    
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
$$;

-- Call procedure
CALL process_order(123, 456, 2, NULL);
```

#### **MySQL**
```sql
-- Create stored procedure
DELIMITER //
CREATE PROCEDURE ProcessOrder(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    OUT p_order_id INT
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;
    
    START TRANSACTION;
    
    -- Insert order
    INSERT INTO orders (customer_id, order_date, status)
    VALUES (p_customer_id, CURDATE(), 'pending');
    
    SET p_order_id = LAST_INSERT_ID();
    
    -- Insert order items
    INSERT INTO order_items (order_id, product_id, quantity, unit_price)
    SELECT p_order_id, p_product_id, p_quantity, price
    FROM products WHERE product_id = p_product_id;
    
    -- Update inventory
    UPDATE products 
    SET stock_quantity = stock_quantity - p_quantity
    WHERE product_id = p_product_id;
    
    COMMIT;
END//
DELIMITER ;

-- Call procedure
CALL ProcessOrder(123, 456, 2, @order_id);
SELECT @order_id;
```

#### **SQL Server**
```sql
-- Create stored procedure
CREATE PROCEDURE ProcessOrder
    @CustomerID INT,
    @ProductID INT,
    @Quantity INT,
    @OrderID INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    
    BEGIN TRY
        BEGIN TRANSACTION;
        
        -- Insert order
        INSERT INTO orders (customer_id, order_date, status)
        VALUES (@CustomerID, GETDATE(), 'pending');
        
        SET @OrderID = SCOPE_IDENTITY();
        
        -- Insert order items
        INSERT INTO order_items (order_id, product_id, quantity, unit_price)
        SELECT @OrderID, @ProductID, @Quantity, price
        FROM products WHERE product_id = @ProductID;
        
        -- Update inventory
        UPDATE products 
        SET stock_quantity = stock_quantity - @Quantity
        WHERE product_id = @ProductID;
        
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;

-- Call procedure
DECLARE @NewOrderID INT;
EXEC ProcessOrder 123, 456, 2, @NewOrderID OUTPUT;
SELECT @NewOrderID;
```

### **Common Use Cases**
- **Business Logic**: Complex calculations and data processing
- **Data Validation**: Enforce business rules
- **Batch Operations**: Process large datasets efficiently
- **Security**: Controlled data access through procedures
- **Integration**: API endpoints for external systems

## Functions

### **What are Functions?**
Database objects that accept parameters, perform calculations or data retrieval, and return a single value or table. Unlike procedures, functions must return a value and cannot modify database state (in most databases).

### **Types of Functions**

#### **Scalar Functions**
Return a single value.

```sql
-- PostgreSQL: Calculate tax amount
CREATE OR REPLACE FUNCTION calculate_tax(p_amount DECIMAL, p_rate DECIMAL)
RETURNS DECIMAL
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN p_amount * p_rate;
END;
$$;

-- Usage
SELECT order_id, total_amount, calculate_tax(total_amount, 0.08) as tax_amount
FROM orders;

-- MySQL: Format customer name
DELIMITER //
CREATE FUNCTION FormatCustomerName(first_name VARCHAR(50), last_name VARCHAR(50))
RETURNS VARCHAR(101)
DETERMINISTIC
READS SQL DATA
BEGIN
    RETURN CONCAT(UPPER(LEFT(first_name, 1)), LOWER(SUBSTRING(first_name, 2)), ' ', 
                  UPPER(LEFT(last_name, 1)), LOWER(SUBSTRING(last_name, 2)));
END//
DELIMITER ;

-- Usage
SELECT customer_id, FormatCustomerName(first_name, last_name) as formatted_name
FROM customers;
```

#### **Table-Valued Functions**
Return a table of data.

```sql
-- SQL Server: Get customer orders
CREATE FUNCTION GetCustomerOrders(@CustomerID INT, @StartDate DATE, @EndDate DATE)
RETURNS TABLE
AS
RETURN (
    SELECT 
        o.order_id,
        o.order_date,
        o.total_amount,
        o.status,
        COUNT(oi.order_item_id) as item_count
    FROM orders o
    LEFT JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.customer_id = @CustomerID
    AND o.order_date BETWEEN @StartDate AND @EndDate
    GROUP BY o.order_id, o.order_date, o.total_amount, o.status
);

-- Usage
SELECT * FROM GetCustomerOrders(123, '2024-01-01', '2024-12-31');

-- PostgreSQL: Get product sales summary
CREATE OR REPLACE FUNCTION get_product_sales_summary(p_start_date DATE, p_end_date DATE)
RETURNS TABLE (
    product_id INT,
    product_name VARCHAR,
    total_quantity BIGINT,
    total_revenue DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        p.product_id,
        p.product_name,
        COALESCE(SUM(oi.quantity), 0) as total_quantity,
        COALESCE(SUM(oi.quantity * oi.unit_price), 0) as total_revenue
    FROM products p
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date BETWEEN p_start_date AND p_end_date
    GROUP BY p.product_id, p.product_name
    ORDER BY total_revenue DESC;
END;
$$;

-- Usage
SELECT * FROM get_product_sales_summary('2024-01-01', '2024-12-31');
```

### **Function Characteristics**
- **Deterministic**: Same inputs always produce same outputs
- **Read-Only**: Cannot modify database state (generally)
- **Cacheable**: Results can be cached for performance
- **Composable**: Can be used in SELECT, WHERE, ORDER BY clauses

## Triggers

### **What are Triggers?**
Special stored procedures that automatically execute (fire) in response to specific database events such as INSERT, UPDATE, or DELETE operations.

### **Types of Triggers**

#### **BEFORE Triggers**
Execute before the triggering event.

```sql
-- PostgreSQL: Validate data before insert
CREATE OR REPLACE FUNCTION validate_order_before_insert()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Validate customer exists
    IF NOT EXISTS (SELECT 1 FROM customers WHERE customer_id = NEW.customer_id) THEN
        RAISE EXCEPTION 'Customer ID % does not exist', NEW.customer_id;
    END IF;
    
    -- Set default values
    IF NEW.order_date IS NULL THEN
        NEW.order_date := CURRENT_DATE;
    END IF;
    
    IF NEW.status IS NULL THEN
        NEW.status := 'pending';
    END IF;
    
    RETURN NEW;
END;
$$;

CREATE TRIGGER tr_orders_before_insert
    BEFORE INSERT ON orders
    FOR EACH ROW
    EXECUTE FUNCTION validate_order_before_insert();

-- MySQL: Auto-generate SKU before product insert
DELIMITER //
CREATE TRIGGER tr_products_before_insert
BEFORE INSERT ON products
FOR EACH ROW
BEGIN
    IF NEW.sku IS NULL OR NEW.sku = '' THEN
        SET NEW.sku = CONCAT('PRD', LPAD(NEW.product_id, 6, '0'));
    END IF;
    
    SET NEW.created_at = NOW();
END//
DELIMITER ;
```

#### **AFTER Triggers**
Execute after the triggering event.

```sql
-- PostgreSQL: Update inventory after order item insert
CREATE OR REPLACE FUNCTION update_inventory_after_order()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Update product inventory
    UPDATE products 
    SET stock_quantity = stock_quantity - NEW.quantity,
        last_modified = CURRENT_TIMESTAMP
    WHERE product_id = NEW.product_id;
    
    -- Log inventory change
    INSERT INTO inventory_log (product_id, change_type, quantity_change, order_id, logged_at)
    VALUES (NEW.product_id, 'order', -NEW.quantity, NEW.order_id, CURRENT_TIMESTAMP);
    
    RETURN NEW;
END;
$$;

CREATE TRIGGER tr_order_items_after_insert
    AFTER INSERT ON order_items
    FOR EACH ROW
    EXECUTE FUNCTION update_inventory_after_order();

-- SQL Server: Audit trail trigger
CREATE TRIGGER tr_customers_audit
ON customers
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    
    INSERT INTO customer_audit (
        customer_id, 
        old_email, 
        new_email, 
        old_status, 
        new_status,
        modified_by, 
        modified_at
    )
    SELECT 
        i.customer_id,
        d.email as old_email,
        i.email as new_email,
        d.status as old_status,
        i.status as new_status,
        SUSER_SNAME() as modified_by,
        GETDATE() as modified_at
    FROM inserted i
    INNER JOIN deleted d ON i.customer_id = d.customer_id
    WHERE i.email != d.email OR i.status != d.status;
END;
```

#### **INSTEAD OF Triggers**
Replace the triggering event (mainly for views).

```sql
-- SQL Server: Updatable view with INSTEAD OF trigger
CREATE VIEW customer_order_summary AS
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    COUNT(o.order_id) as order_count,
    COALESCE(SUM(o.total_amount), 0) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.email;

CREATE TRIGGER tr_customer_order_summary_update
ON customer_order_summary
INSTEAD OF UPDATE
AS
BEGIN
    UPDATE customers 
    SET customer_name = i.customer_name,
        email = i.email
    FROM customers c
    INNER JOIN inserted i ON c.customer_id = i.customer_id;
END;
```

## Advanced Patterns and Techniques

### **Error Handling**

#### **PostgreSQL Exception Handling**
```sql
CREATE OR REPLACE FUNCTION safe_transfer_funds(
    p_from_account INT,
    p_to_account INT,
    p_amount DECIMAL
)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
DECLARE
    v_from_balance DECIMAL;
BEGIN
    -- Check source account balance
    SELECT balance INTO v_from_balance
    FROM accounts WHERE account_id = p_from_account
    FOR UPDATE;
    
    IF v_from_balance < p_amount THEN
        RAISE EXCEPTION 'Insufficient funds. Balance: %, Requested: %', 
                       v_from_balance, p_amount;
    END IF;
    
    -- Perform transfer
    UPDATE accounts SET balance = balance - p_amount 
    WHERE account_id = p_from_account;
    
    UPDATE accounts SET balance = balance + p_amount 
    WHERE account_id = p_to_account;
    
    RETURN TRUE;
    
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Transfer failed: %', SQLERRM;
        RETURN FALSE;
END;
$$;
```

### **Dynamic SQL and Cursor Operations**

#### **MySQL Dynamic SQL**
```sql
DELIMITER //
CREATE PROCEDURE GenerateReport(IN table_name VARCHAR(64), IN date_column VARCHAR(64))
BEGIN
    SET @sql = CONCAT('SELECT COUNT(*) as total_records, 
                              DATE(', date_column, ') as report_date
                       FROM ', table_name, ' 
                       WHERE ', date_column, ' >= DATE_SUB(NOW(), INTERVAL 30 DAY)
                       GROUP BY DATE(', date_column, ')
                       ORDER BY report_date');
    
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END//
DELIMITER ;
```

#### **SQL Server Cursor Example**
```sql
CREATE PROCEDURE ProcessLargeDataset
AS
BEGIN
    DECLARE @customer_id INT, @total_orders INT;
    
    DECLARE customer_cursor CURSOR FOR
    SELECT customer_id FROM customers WHERE status = 'active';
    
    OPEN customer_cursor;
    
    FETCH NEXT FROM customer_cursor INTO @customer_id;
    
    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Process each customer
        SELECT @total_orders = COUNT(*)
        FROM orders 
        WHERE customer_id = @customer_id;
        
        -- Update customer summary
        UPDATE customer_summary 
        SET total_orders = @total_orders,
            last_updated = GETDATE()
        WHERE customer_id = @customer_id;
        
        FETCH NEXT FROM customer_cursor INTO @customer_id;
    END;
    
    CLOSE customer_cursor;
    DEALLOCATE customer_cursor;
END;
```

## Performance Considerations

### **Optimization Techniques**
- **Minimize Logic**: Keep procedures simple and focused
- **Use Indexes**: Ensure queries within procedures use appropriate indexes
- **Avoid Cursors**: Use set-based operations when possible
- **Parameter Sniffing**: Be aware of execution plan caching issues
- **Batch Operations**: Process data in batches for large datasets

### **Caching and Recompilation**
```sql
-- SQL Server: Recompile option for parameter-sensitive procedures
CREATE PROCEDURE GetOrdersByDateRange
    @StartDate DATE,
    @EndDate DATE
WITH RECOMPILE
AS
BEGIN
    SELECT order_id, customer_id, order_date, total_amount
    FROM orders 
    WHERE order_date BETWEEN @StartDate AND @EndDate;
END;

-- PostgreSQL: Prepared statements for better performance
PREPARE get_customer_orders (INT) AS
SELECT order_id, order_date, total_amount
FROM orders 
WHERE customer_id = $1
ORDER BY order_date DESC;

-- Execute prepared statement
EXECUTE get_customer_orders(123);
```

## Security and Best Practices

### **Security Considerations**
- **SQL Injection Prevention**: Use parameterized queries
- **Principle of Least Privilege**: Grant minimal necessary permissions
- **Input Validation**: Validate all input parameters
- **Error Handling**: Don't expose sensitive information in error messages

### **Example: Secure Parameter Validation**
```sql
-- PostgreSQL: Input validation function
CREATE OR REPLACE FUNCTION validate_email(p_email TEXT)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    -- Basic email validation
    IF p_email IS NULL OR LENGTH(p_email) = 0 THEN
        RETURN FALSE;
    END IF;
    
    IF p_email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' THEN
        RETURN FALSE;
    END IF;
    
    RETURN TRUE;
END;
$$;

-- Use in stored procedure
CREATE OR REPLACE PROCEDURE create_customer(
    p_name VARCHAR(100),
    p_email VARCHAR(255)
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Validate inputs
    IF LENGTH(TRIM(p_name)) = 0 THEN
        RAISE EXCEPTION 'Customer name cannot be empty';
    END IF;
    
    IF NOT validate_email(p_email) THEN
        RAISE EXCEPTION 'Invalid email format: %', p_email;
    END IF;
    
    -- Insert customer
    INSERT INTO customers (name, email, created_at)
    VALUES (TRIM(p_name), LOWER(TRIM(p_email)), CURRENT_TIMESTAMP);
END;
$$;
```

## Testing and Debugging

### **Unit Testing Procedures**
```sql
-- PostgreSQL: Test framework example
CREATE OR REPLACE FUNCTION test_calculate_tax()
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
DECLARE
    v_result DECIMAL;
    v_expected DECIMAL := 8.00;
BEGIN
    -- Test the calculate_tax function
    SELECT calculate_tax(100.00, 0.08) INTO v_result;
    
    IF v_result = v_expected THEN
        RAISE NOTICE 'TEST PASSED: calculate_tax(100.00, 0.08) = %', v_result;
        RETURN TRUE;
    ELSE
        RAISE NOTICE 'TEST FAILED: Expected %, Got %', v_expected, v_result;
        RETURN FALSE;
    END IF;
END;
$$;

-- Run test
SELECT test_calculate_tax();
```

### **Debugging Techniques**
```sql
-- PostgreSQL: Debug logging
CREATE OR REPLACE PROCEDURE debug_order_processing(p_order_id INT)
LANGUAGE plpgsql
AS $$
DECLARE
    v_step TEXT;
    v_customer_id INT;
    v_order_total DECIMAL;
BEGIN
    v_step := 'Getting order details';
    RAISE NOTICE 'DEBUG: %', v_step;
    
    SELECT customer_id, total_amount INTO v_customer_id, v_order_total
    FROM orders WHERE order_id = p_order_id;
    
    RAISE NOTICE 'DEBUG: Order % belongs to customer %, total: %', 
                 p_order_id, v_customer_id, v_order_total;
    
    v_step := 'Processing payment';
    RAISE NOTICE 'DEBUG: %', v_step;
    
    -- Processing logic here
    
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'ERROR at step "%": %', v_step, SQLERRM;
        RAISE;
END;
$$;
```

## Modern Alternatives and Considerations

### **Application vs Database Logic**
**Database Logic (Procedures/Functions):**
- **Pros**: Performance, data integrity, centralized logic
- **Cons**: Database vendor lock-in, harder to version control, limited debugging

**Application Logic:**
- **Pros**: Language flexibility, easier testing, better version control
- **Cons**: Network overhead, potential data inconsistency

### **Microservices Considerations**
- **Database per Service**: Stored procedures become service-specific
- **Event-Driven**: Triggers can publish events for other services
- **API Gateway**: Procedures can provide API endpoints
- **Data Consistency**: Complex transactions across services

## Summary

Stored procedures, functions, and triggers provide powerful capabilities for database-level logic implementation. They offer performance benefits and ensure data consistency but require careful consideration of maintainability, testability, and architectural constraints.

**Key Benefits:**
- **Performance**: Pre-compiled execution and reduced network traffic
- **Security**: Controlled data access and SQL injection prevention
- **Consistency**: Centralized business logic and data validation
- **Automation**: Triggers provide automatic data processing

**Best Practices:**
- **Keep It Simple**: Focus procedures on specific, well-defined tasks
- **Validate Inputs**: Always validate and sanitize input parameters
- **Handle Errors**: Implement comprehensive error handling
- **Document Thoroughly**: Maintain clear documentation and examples
- **Test Regularly**: Implement unit tests for database logic

**Modern Considerations:**
- **Balance**: Choose between database and application logic based on requirements
- **Maintainability**: Consider long-term maintenance and debugging needs
- **Scalability**: Evaluate impact on horizontal scaling and distributed systems
- **Team Skills**: Ensure team has necessary database programming expertise

**Decision Framework:**
- **Use for**: Data validation, complex calculations, audit trails, batch processing
- **Avoid for**: Simple CRUD operations, frequently changing business logic
- **Consider**: Team expertise, deployment complexity, testing requirements

The key is finding the right balance between leveraging database capabilities and maintaining application flexibility, always considering the specific requirements and constraints of your system architecture.