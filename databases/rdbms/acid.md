# ACID

**ACID** stands for **Atomicity**, **Consistency**, **Isolation**, and **Durability**. These properties are designed to guarantee that database transactions are processed reliably, even in the event of failures such as crashes, errors, or concurrent access issues.

### 1. **Atomicity**

- **Definition**: Ensures that each transaction is treated as a single "unit" of work. It either **completes fully** or doesn't happen at all.
- **Example**: Consider a bank transfer where $100 is debited from one account and credited to another. Atomicity ensures that both actions happen, or neither happens. If there’s a system crash after debiting but before crediting, the transaction is rolled back.

**In a nutshell**: All-or-nothing principle. If any part of a transaction fails, the whole transaction fails, and no changes are made to the database.

### 2. **Consistency**

- **Definition**: Guarantees that a transaction brings the database from one **valid state** to another. The database must always adhere to predefined rules (constraints, triggers, etc.).
- **Example**: If an account cannot have a negative balance, consistency ensures that no transaction violates this rule, even during failures. Any transaction that would cause inconsistency is rolled back.

**In a nutshell**: The database must always remain in a valid state. Any transaction that violates rules is aborted.

### 3. **Isolation**

- **Definition**: Ensures that the operations of one transaction are **invisible** to other transactions until the transaction is complete. This prevents "dirty reads" (reading uncommitted data).
- **Example**: If two users are concurrently withdrawing money from the same account, isolation ensures that each withdrawal happens one at a time, without interference. Without isolation, one user might see an incorrect balance.

**In a nutshell**: Transactions are executed as if they are the only transaction in the system, ensuring data consistency even with concurrent transactions.

PostgreSQL and other databases offer different **isolation levels**:

- **Read Uncommitted**: Transactions can read data that is not yet committed.
- **Read Committed**: A transaction can only read data that has been committed.
- **Repeatable Read**: Ensures the same result for each read within the same transaction.
- **Serializable**: The strictest level, ensuring transactions appear as if they are run sequentially.

### 4. **Durability**

- **Definition**: Ensures that once a transaction is committed, its changes are **permanently stored** in the database, even in the event of a crash or power failure.
- **Example**: After a bank transfer is confirmed, the new account balances are written to disk. Even if the system crashes immediately after, the changes will not be lost.

**In a nutshell**: Once a transaction is successfully committed, its changes are guaranteed to be saved, no matter what happens (crashes, power outages, etc.).

---

### **Why ACID is Important:**

- **Data Integrity**: ACID ensures the integrity of the data. Even in multi-user environments with concurrent transactions, these properties help maintain consistency.
- **Fault Tolerance**: ACID properties help the database recover from failures, maintaining the correctness of the data.
- **Concurrency**: With ACID properties, databases can handle multiple transactions safely without corrupting the data or breaking rules.

### Example in PostgreSQL:

Imagine you are transferring money between two bank accounts:

```sql
sql
Copy code
BEGIN;  -- Start the transaction

-- Step 1: Deduct $100 from Account A
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- Step 2: Add $100 to Account B
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Commit the transaction (if both steps succeed)
COMMIT;

```

- **Atomicity**: Either both `UPDATE` statements succeed, or neither will.
- **Consistency**: The total amount of money in both accounts before and after the transaction remains the same.
- **Isolation**: If other transactions are happening at the same time, they won’t interfere with this one.
- **Durability**: Once the transaction is committed, the new balances are saved permanently.

---

### Summary of ACID:

1. **Atomicity**: All or nothing—either the entire transaction succeeds, or none of it does.
2. **Consistency**: Transactions must bring the database from one valid state to another.
3. **Isolation**: Transactions must not interfere with each other.
4. **Durability**: Committed changes are permanent, even in the case of failures.

ACID properties are essential for any **reliable** and **robust** database system, especially when dealing with critical data.