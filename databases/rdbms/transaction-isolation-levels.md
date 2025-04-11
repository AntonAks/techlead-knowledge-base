# Transaction Isolation Levels

In SQL databases, **transaction isolation levels** define how transaction changes are visible to other transactions and how they are affected by others' concurrent operations. Different isolation levels offer trade-offs between **data consistency** and **system performance**. PostgreSQL, like many other relational databases, provides several isolation levels that control how much "interference" is allowed between transactions.

### The Four Standard Isolation Levels:

1. **Read Uncommitted**
2. **Read Committed**
3. **Repeatable Read**
4. **Serializable**

Each isolation level is designed to prevent specific types of problems, also called **concurrency phenomena**: **dirty reads**, **non-repeatable reads**, and **phantom reads**.

### 1. **Read Uncommitted**

- **Definition**: Transactions can read data that **has been modified but not yet committed** by other transactions. This is the lowest isolation level.
- **Key Features**:
    - **Dirty Reads**: Allowed. A transaction can see uncommitted changes from other transactions.
    - **Non-Repeatable Reads**: Possible.
    - **Phantom Reads**: Possible.
- **Use Case**: Rarely used because it sacrifices data integrity. It can improve performance slightly, but at the risk of reading "dirty" or inconsistent data.
- **Example**: Transaction A updates a row but has not yet committed. Transaction B reads the updated row. If A rolls back, B has read data that no longer exists in the final state.

### 2. **Read Committed** (PostgreSQL Default)

- **Definition**: A transaction can only read data that has been **committed** by other transactions. Any data written by uncommitted transactions is **invisible**.
- **Key Features**:
    - **Dirty Reads**: Not allowed. Only committed data is visible.
    - **Non-Repeatable Reads**: Possible. The same query might return different results if another transaction commits changes between queries.
    - **Phantom Reads**: Possible.
- **Use Case**: The default isolation level in PostgreSQL and most other databases. Suitable for applications where slight variations in read results are acceptable but dirty reads are not.
- **Example**: Transaction A starts and reads some data. Transaction B updates and commits changes to the same data before Transaction A reads it again. Transaction A might see different results in its second read.

### 3. **Repeatable Read**

- **Definition**: A transaction reads consistent data throughout its execution. Once a transaction reads data, **no other transaction can modify** that data until the first transaction completes. This prevents **non-repeatable reads**.
- **Key Features**:
    - **Dirty Reads**: Not allowed.
    - **Non-Repeatable Reads**: Not allowed. The data read once in a transaction remains the same for the transaction's duration, even if another transaction modifies the data.
    - **Phantom Reads**: Possible. New rows can still be inserted by other transactions, which the current transaction might not "see."
- **Use Case**: Ensures that if a transaction reads the same row twice, it will get the same result. Good for transactions that need to see a consistent snapshot of the data but don't need the full isolation of `Serializable`.
- **Example**: Transaction A reads a row. Transaction B tries to modify the same row and commit. Transaction A, which uses `Repeatable Read`, will see the original version of the row (before B's changes) if it reads the row again.

### 4. **Serializable** (Strictest Level)

- **Definition**: Transactions are executed in such a way that they appear to run **sequentially**, even though they might be executed concurrently. This level eliminates **phantom reads** and ensures the highest level of isolation.
- **Key Features**:
    - **Dirty Reads**: Not allowed.
    - **Non-Repeatable Reads**: Not allowed.
    - **Phantom Reads**: Not allowed. No other transaction can insert, delete, or update rows in a way that affects the outcome of a running transaction.
- **Use Case**: Suitable for systems where consistency and correctness are paramount. It offers the strongest guarantee of isolation but may reduce concurrency and performance due to higher overhead.
- **Example**: If Transaction A queries a range of rows and Transaction B inserts new rows into that range, Transaction A will not see these new rows, maintaining the appearance that its operations are fully isolated.

---

### **Concurrency Phenomena**:

1. **Dirty Reads**:
    - Occurs when a transaction reads data that has been modified by another transaction but not yet committed.
    - Isolation levels allowing this: **Read Uncommitted**.
2. **Non-Repeatable Reads**:
    - Occurs when a transaction reads the same data twice but gets different results because another transaction modified the data in between.
    - Isolation levels allowing this: **Read Committed**, **Read Uncommitted**.
3. **Phantom Reads**:
    - Occurs when a transaction reads a set of rows that satisfies a condition, and another transaction inserts or deletes rows that affect the result set.
    - Isolation levels allowing this: **Read Committed**, **Repeatable Read**.

---

### **Comparison Table:**

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads |
| --- | --- | --- | --- |
| **Read Uncommitted** | Yes | Yes | Yes |
| **Read Committed** | No | Yes | Yes |
| **Repeatable Read** | No | No | Yes |
| **Serializable** | No | No | No |

---

### How Isolation Levels Affect Performance

- **Higher isolation levels (like Serializable)** ensure stronger consistency but come with a **performance trade-off**. They might lead to more locks, reducing the system’s ability to process many transactions concurrently.
- **Lower isolation levels (like Read Uncommitted)** improve performance but might result in **inconsistent or incorrect data** being read, which can lead to application errors.

In PostgreSQL, the default isolation level is **Read Committed**, which is often a good balance between consistency and performance for most applications. However, if you need stronger guarantees, such as for financial transactions or complex business logic, you might opt for **Repeatable Read** or **Serializable**.