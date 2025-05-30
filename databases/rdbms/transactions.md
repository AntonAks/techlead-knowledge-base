# Database Transactions

## What are Transactions?

A **transaction** is a sequence of database operations that are executed as a single logical unit of work. Transactions ensure data consistency and integrity by following the ACID properties: Atomicity, Consistency, Isolation, and Durability.

## ACID Properties

### **Atomicity**
All operations in a transaction succeed or fail together—no partial completion.

```sql
-- Bank transfer example: Both operations must succeed or both fail
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 500 WHERE account_id = 2;

-- If either UPDATE fails, both are rolled back
COMMIT;
```

### **Consistency**
Transactions bring the database from one valid state to another, maintaining all rules and constraints.

```sql
-- Ensure inventory never goes negative
BEGIN TRANSACTION;

-- Check current stock
SELECT stock_quantity INTO @current_stock 
FROM products WHERE product_id = 123;

-- Only proceed if enough stock
IF @current_stock >= 5 THEN
    UPDATE products SET stock_quantity = stock_quantity - 5 
    WHERE product_id = 123;
    
    INSERT INTO order_items (product_id, quantity) 
    VALUES (123, 5);
    
    COMMIT;
ELSE
    ROLLBACK;
END IF;
```

### **Isolation**
Concurrent transactions don't interfere with each other.

```sql
-- Transaction 1: Reading account balance
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 1; -- Reads $1000
-- ... other operations
COMMIT;

-- Transaction 2: Updating same account (running concurrently)
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;
```

### **Durability**
Committed changes are permanently stored, even if system crashes.

```sql
-- Once committed, this change survives system restart
BEGIN TRANSACTION;
INSERT INTO critical_data (value) VALUES ('important');
COMMIT; -- Data is now durable
```

## Transaction Syntax

### **PostgreSQL**
```sql
-- Explicit transaction
BEGIN;
    INSERT INTO orders (customer_id, total) VALUES (123, 99.99);
    INSERT INTO order_items (order_id, product_id) VALUES (LASTVAL(), 456);
    UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 456;
COMMIT;

-- With error handling
BEGIN;
    -- Operations here
    IF some_condition THEN
        ROLLBACK;
    ELSE
        COMMIT;
    END IF;

-- Savepoints for partial rollback
BEGIN;
    INSERT INTO table1 VALUES (1);
    SAVEPOINT sp1;
    INSERT INTO table2 VALUES (2);
    ROLLBACK TO sp1; -- Only rolls back table2 insert
    INSERT INTO table3 VALUES (3);
COMMIT;
```

### **MySQL**
```sql
-- Explicit transaction
START TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- With error handling
START TRANSACTION;
    UPDATE products SET stock = stock - 1 WHERE id = 123;
    
    -- Check if update affected any rows
    IF ROW_COUNT() = 0 THEN
        ROLLBACK;
        SELECT 'Product not found' as error;
    ELSE
        INSERT INTO sales (product_id, quantity) VALUES (123, 1);
        COMMIT;
    END IF;

-- Autocommit control
SET autocommit = 0; -- Disable autocommit
UPDATE table1 SET col1 = 'value';
UPDATE table2 SET col2 = 'value';
COMMIT; -- Manual commit required
SET autocommit = 1; -- Re-enable autocommit
```

### **SQL Server**
```sql
-- Explicit transaction
BEGIN TRANSACTION;
    UPDATE inventory SET quantity = quantity - 5 WHERE product_id = 123;
    INSERT INTO sales (product_id, quantity, sale_date) 
    VALUES (123, 5, GETDATE());
    
    IF @@ERROR = 0
        COMMIT;
    ELSE
        ROLLBACK;

-- Try-Catch error handling
BEGIN TRY
    BEGIN TRANSACTION;
        -- Operations here
        UPDATE accounts SET balance = balance - @amount WHERE id = @from_account;
        UPDATE accounts SET balance = balance + @amount WHERE id = @to_account;
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
    THROW;
END CATCH;

-- Named transactions
BEGIN TRANSACTION transfer_funds;
    -- Operations
    IF some_error_condition
        ROLLBACK TRANSACTION transfer_funds;
    ELSE
        COMMIT TRANSACTION transfer_funds;
```

## Transaction Isolation Levels

### **Read Uncommitted**
Lowest isolation level—can read uncommitted changes from other transactions.

```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE id = 1; -- May read uncommitted data
COMMIT;

-- Session 2 (concurrent)
BEGIN TRANSACTION;
UPDATE accounts SET balance = 5000 WHERE id = 1; -- Not yet committed
-- Session 1 might see balance = 5000
ROLLBACK; -- Change is undone, but Session 1 may have seen it
```

### **Read Committed**
Can only read committed data (default in most databases).

```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE id = 1; -- Always reads committed data
-- If run again, might see different value if another transaction committed
COMMIT;
```

### **Repeatable Read**
Ensures same data is read throughout the transaction.

```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE id = 1; -- Reads $1000
-- ... other operations
SELECT balance FROM accounts WHERE id = 1; -- Still reads $1000
COMMIT;

-- Session 2 cannot modify account 1 until Session 1 commits
```

### **Serializable**
Highest isolation level—transactions appear to run sequentially.

```sql
-- Prevents phantom reads
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
SELECT COUNT(*) FROM orders WHERE status = 'pending'; -- Gets count = 5
-- No other transaction can insert/delete pending orders
SELECT COUNT(*) FROM orders WHERE status = 'pending'; -- Still gets 5
COMMIT;
```

## Deadlocks and Concurrency

### **Deadlock Example**
```sql
-- Transaction 1
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- Locks account 1
-- ... some delay
UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- Waits for account 2
COMMIT;

-- Transaction 2 (concurrent)
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE id = 2;  -- Locks account 2
-- ... some delay  
UPDATE accounts SET balance = balance + 50 WHERE id = 1;  -- Waits for account 1
COMMIT;

-- Deadlock: Each transaction waits for the other's lock
```

### **Deadlock Prevention**
```sql
-- Solution: Always acquire locks in same order
-- Transaction 1
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- Lock account 1 first
UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- Then account 2
COMMIT;

-- Transaction 2
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance + 50 WHERE id = 1;  -- Lock account 1 first
UPDATE accounts SET balance = balance - 50 WHERE id = 2;  -- Then account 2
COMMIT;
```

### **Lock Timeout Handling**
```sql
-- SQL Server: Set lock timeout
SET LOCK_TIMEOUT 5000; -- 5 seconds

BEGIN TRY
    BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    COMMIT;
END TRY
BEGIN CATCH
    IF ERROR_NUMBER() = 1222 -- Lock timeout
        PRINT 'Transaction timed out due to lock';
    ROLLBACK;
END CATCH;

-- PostgreSQL: Statement timeout
SET statement_timeout = '5s';
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

## Transaction Patterns

### **Optimistic Concurrency Control**
```sql
-- Using version numbers
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    version INT DEFAULT 0
);

-- Application reads product with version
SELECT id, name, price, version FROM products WHERE id = 123;
-- Returns: id=123, name="Widget", price=19.99, version=5

-- Update only if version hasn't changed
UPDATE products 
SET name = 'Updated Widget', 
    price = 24.99, 
    version = version + 1
WHERE id = 123 AND version = 5; -- Original version

-- Check if update succeeded
IF @@ROWCOUNT = 0 THEN
    -- Another transaction modified the record
    RAISERROR('Record was modified by another user', 16, 1);
END IF;
```

### **Pessimistic Locking**
```sql
-- PostgreSQL: SELECT FOR UPDATE
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE; -- Exclusive lock
-- Perform business logic
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- SQL Server: Lock hints
BEGIN TRANSACTION;
SELECT balance FROM accounts WITH (UPDLOCK) WHERE id = 1;
-- Other transactions cannot read this row until commit
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

### **Compensating Transactions**
```sql
-- Saga pattern: Sequence of transactions with compensation
-- Step 1: Reserve inventory
BEGIN TRANSACTION;
UPDATE inventory SET reserved = reserved + 5 WHERE product_id = 123;
INSERT INTO reservations (product_id, quantity, reservation_id) 
VALUES (123, 5, 'res_001');
COMMIT;

-- Step 2: Process payment (external service)
-- If payment fails, compensate step 1:
BEGIN TRANSACTION;
UPDATE inventory SET reserved = reserved - 5 WHERE product_id = 123;
DELETE FROM reservations WHERE reservation_id = 'res_001';
COMMIT;
```

## Distributed Transactions

### **Two-Phase Commit (2PC)**
```sql
-- Coordinator prepares all participants
-- Phase 1: Prepare
BEGIN DISTRIBUTED TRANSACTION;
    -- Database 1
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    PREPARE TRANSACTION 'transfer_001_db1';
    
    -- Database 2  
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    PREPARE TRANSACTION 'transfer_001_db2';

-- Phase 2: Commit (if all prepared successfully)
COMMIT PREPARED 'transfer_001_db1';
COMMIT PREPARED 'transfer_001_db2';

-- Or rollback if any failed
-- ROLLBACK PREPARED 'transfer_001_db1';
-- ROLLBACK PREPARED 'transfer_001_db2';
```

### **XA Transactions (Java Example)**
```java
// Java application coordinating XA transaction
UserTransaction ut = getUserTransaction();
ut.begin();

try {
    // Database 1 operation
    Connection conn1 = getXAConnection("database1");
    PreparedStatement ps1 = conn1.prepareStatement(
        "UPDATE accounts SET balance = balance - ? WHERE id = ?");
    ps1.setDouble(1, 100.00);
    ps1.setInt(2, 1);
    ps1.executeUpdate();
    
    // Database 2 operation
    Connection conn2 = getXAConnection("database2");
    PreparedStatement ps2 = conn2.prepareStatement(
        "UPDATE accounts SET balance = balance + ? WHERE id = ?");
    ps2.setDouble(1, 100.00);
    ps2.setInt(2, 2);
    ps2.executeUpdate();
    
    ut.commit(); // 2PC happens automatically
} catch (Exception e) {
    ut.rollback();
    throw e;
}
```

## Error Handling and Recovery

### **Transaction Recovery Patterns**
```sql
-- PostgreSQL: Exception handling with rollback
CREATE OR REPLACE FUNCTION safe_transfer(
    from_account INT, 
    to_account INT, 
    amount DECIMAL
) 
RETURNS BOOLEAN AS $$
BEGIN
    -- Start transaction implicitly
    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    
    -- Check if source account has sufficient funds
    IF NOT FOUND OR (SELECT balance FROM accounts WHERE id = from_account) < 0 THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;
    
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;
    
    RETURN TRUE;
    
EXCEPTION
    WHEN OTHERS THEN
        -- Transaction automatically rolled back
        RAISE NOTICE 'Transfer failed: %', SQLERRM;
        RETURN FALSE;
END;
$$ LANGUAGE plpgsql;
```

### **Retry Logic for Deadlocks**
```sql
-- Application-level retry pattern
DECLARE @retry_count INT = 0;
DECLARE @max_retries INT = 3;

WHILE @retry_count < @max_retries
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;
            -- Your transaction logic here
            UPDATE accounts SET balance = balance - @amount WHERE id = @from_id;
            UPDATE accounts SET balance = balance + @amount WHERE id = @to_id;
        COMMIT TRANSACTION;
        
        BREAK; -- Success, exit loop
        
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;
            
        -- Check if deadlock occurred
        IF ERROR_NUMBER() = 1205 -- Deadlock
        BEGIN
            SET @retry_count = @retry_count + 1;
            WAITFOR DELAY '00:00:01'; -- Wait 1 second before retry
        END
        ELSE
        BEGIN
            THROW; -- Re-throw non-deadlock errors
        END
    END CATCH
END;
```

## Performance Considerations

### **Transaction Size and Duration**
```sql
-- Bad: Long-running transaction locks resources
BEGIN TRANSACTION;
    -- Process 1 million records
    DECLARE @id INT = 1;
    WHILE @id <= 1000000
    BEGIN
        UPDATE large_table SET processed = 1 WHERE id = @id;
        SET @id = @id + 1;
    END;
COMMIT; -- Holds locks for long time

-- Better: Batch processing with smaller transactions
DECLARE @batch_size INT = 1000;
DECLARE @start_id INT = 1;
DECLARE @end_id INT;

WHILE @start_id <= 1000000
BEGIN
    SET @end_id = @start_id + @batch_size - 1;
    
    BEGIN TRANSACTION;
        UPDATE large_table 
        SET processed = 1 
        WHERE id BETWEEN @start_id AND @end_id;
    COMMIT;
    
    SET @start_id = @end_id + 1;
END;
```

### **Index Optimization for Transactions**
```sql
-- Create indexes to minimize lock duration
CREATE INDEX idx_accounts_id ON accounts(id); -- Primary key lookup
CREATE INDEX idx_inventory_product ON inventory(product_id); -- Fast updates

-- Use specific WHERE clauses to minimize locked rows
UPDATE inventory 
SET quantity = quantity - 5 
WHERE product_id = 123 AND warehouse_id = 1; -- Locks fewer rows

-- Rather than
UPDATE inventory 
SET quantity = quantity - 5 
WHERE product_id = 123; -- Locks all warehouses for product
```

## Monitoring and Troubleshooting

### **Transaction Monitoring**
```sql
-- PostgreSQL: View current transactions
SELECT 
    pid,
    usename,
    application_name,
    state,
    query_start,
    xact_start,
    query
FROM pg_stat_activity 
WHERE state IN ('active', 'idle in transaction')
ORDER BY xact_start;

-- MySQL: Show running transactions
SELECT 
    trx_id,
    trx_state,
    trx_started,
    trx_isolation_level,
    trx_tables_locked,
    trx_rows_locked
FROM information_schema.innodb_trx;

-- SQL Server: Active transactions
SELECT 
    s.session_id,
    t.name as transaction_name,
    t.transaction_begin_time,
    s.login_name,
    s.host_name,
    r.command,
    r.wait_type
FROM sys.dm_tran_active_transactions t
JOIN sys.dm_exec_sessions s ON t.session_id = s.session_id
LEFT JOIN sys.dm_exec_requests r ON s.session_id = r.session_id;
```

### **Deadlock Detection**
```sql
-- SQL Server: Enable deadlock monitoring
DBCC TRACEON(1222, -1); -- Log deadlock information

-- PostgreSQL: Log lock waits
SET log_lock_waits = on;
SET deadlock_timeout = '1s';

-- Check for deadlocks in logs
SELECT * FROM pg_stat_database_conflicts;
```

## Best Practices

### **Transaction Design**
- **Keep Transactions Short**: Minimize lock duration
- **Consistent Lock Order**: Prevent deadlocks by acquiring locks in same order
- **Avoid User Interaction**: Don't wait for user input within transactions
- **Handle Errors**: Always include proper error handling and rollback
- **Use Appropriate Isolation**: Choose lowest isolation level that maintains consistency

### **Performance Optimization**
- **Batch Operations**: Process large datasets in smaller batches
- **Index Strategy**: Ensure queries use appropriate indexes
- **Connection Pooling**: Reuse database connections efficiently
- **Monitor Metrics**: Track transaction duration, deadlock frequency, lock waits

### **Error Handling**
- **Retry Logic**: Implement retry for transient errors (deadlocks, timeouts)
- **Compensation**: Design compensating transactions for distributed scenarios
- **Logging**: Log transaction failures for debugging
- **Graceful Degradation**: Handle transaction failures gracefully in application

## Modern Transaction Patterns

### **Event Sourcing**
```sql
-- Store events instead of current state
CREATE TABLE events (
    event_id BIGSERIAL PRIMARY KEY,
    aggregate_id UUID,
    event_type VARCHAR(50),
    event_data JSONB,
    event_timestamp TIMESTAMP DEFAULT NOW(),
    version INT
);

-- Account balance is calculated from events
INSERT INTO events (aggregate_id, event_type, event_data, version)
VALUES 
    ('account-123', 'AccountCreated', '{"initial_balance": 1000}', 1),
    ('account-123', 'MoneyWithdrawn', '{"amount": 100}', 2),
    ('account-123', 'MoneyDeposited', '{"amount": 50}', 3);

-- Calculate current balance
SELECT 
    aggregate_id,
    SUM(CASE 
        WHEN event_type = 'AccountCreated' THEN (event_data->>'initial_balance')::DECIMAL
        WHEN event_type = 'MoneyDeposited' THEN (event_data->>'amount')::DECIMAL
        WHEN event_type = 'MoneyWithdrawn' THEN -(event_data->>'amount')::DECIMAL
        ELSE 0
    END) as current_balance
FROM events 
WHERE aggregate_id = 'account-123'
GROUP BY aggregate_id;
```

### **Saga Pattern**
```sql
-- Orchestrated saga with compensation
CREATE TABLE saga_transactions (
    saga_id UUID PRIMARY KEY,
    step_number INT,
    service_name VARCHAR(50),
    operation VARCHAR(50),
    compensation VARCHAR(50),
    status VARCHAR(20), -- 'pending', 'completed', 'compensated', 'failed'
    request_data JSONB,
    response_data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Example saga steps
INSERT INTO saga_transactions VALUES
    ('saga-001', 1, 'inventory', 'reserve_items', 'release_items', 'completed', '{"product_id": 123, "quantity": 5}', '{"reservation_id": "res-001"}', NOW()),
    ('saga-001', 2, 'payment', 'charge_card', 'refund_card', 'completed', '{"amount": 99.99, "card_token": "tok-123"}', '{"charge_id": "chg-001"}', NOW()),
    ('saga-001', 3, 'shipping', 'create_shipment', 'cancel_shipment', 'failed', '{"address": "123 Main St"}', '{"error": "address_invalid"}', NOW());

-- If step fails, run compensation for completed steps in reverse order
```

## Summary

Database transactions are fundamental for maintaining data integrity and consistency in concurrent systems. Understanding transaction properties, isolation levels, and common patterns is essential for building reliable applications.

**Key Principles:**
- **ACID Compliance**: Ensure atomicity, consistency, isolation, and durability
- **Appropriate Isolation**: Use the lowest isolation level that maintains data integrity
- **Short Duration**: Keep transactions as brief as possible to minimize lock contention
- **Error Handling**: Always include comprehensive error handling and recovery logic
- **Deadlock Prevention**: Design transactions to avoid deadlock scenarios

**Common Patterns:**
- **Optimistic Concurrency**: Use versioning for high-concurrency scenarios
- **Pessimistic Locking**: Use explicit locks when conflicts are likely
- **Batch Processing**: Break large operations into smaller transactions
- **Compensation**: Design reversible operations for distributed scenarios

**Modern Considerations:**
- **Microservices**: Use saga patterns for distributed transactions
- **Event Sourcing**: Store events instead of current state for auditability
- **NoSQL**: Understand limited transaction support in NoSQL databases
- **Cloud Databases**: Leverage cloud-specific transaction features

**Performance Tips:**
- **Monitor Actively**: Track transaction metrics and deadlock frequency
- **Index Strategically**: Ensure fast access to frequently updated data
- **Connection Management**: Use connection pooling and proper resource cleanup
- **Testing**: Test transaction behavior under concurrent load

Effective transaction management requires balancing data consistency requirements with performance and scalability needs, always considering the specific requirements of your application and the capabilities of your chosen database system.