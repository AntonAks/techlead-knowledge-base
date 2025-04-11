# Indexes

### What is an Index?

An **index** is a **data structure** that improves the speed of data retrieval operations on a table at the cost of additional storage space and slower write operations (like `INSERT`, `UPDATE`, and `DELETE`). Indexes help the database look up rows efficiently without scanning the entire table.

### Why Use Indexes?

- **Faster Queries**: Indexes drastically reduce the amount of data that needs to be scanned to fulfill a query.
- **Efficient Sorting**: Queries with `ORDER BY` clauses can be optimized with indexes.
- **Quick Searching**: Searching for specific rows using `WHERE` clauses is faster when indexed columns are used.

### How Indexes Work

An index is like a "lookup table" for data in the actual table. Instead of scanning every row (a **sequential scan**) to find what you're looking for, the index points to the location of the data, so PostgreSQL can retrieve it quickly.

### Types of Indexes in PostgreSQL (with Examples)

1. **B-tree Index** (Default and most common)
2. **Hash Index**
3. **GIN (Generalized Inverted Index)**
4. **GiST (Generalized Search Tree)**
5. **BRIN (Block Range INdex)**

### 1. **B-tree Index (Balanced Tree)**

- **Description**: This is the **default index type** in PostgreSQL. It's efficient for operations using comparisons like `=`, `<`, `<=`, `>`, `>=`, and `BETWEEN`.
- **Use Case**: Ideal for equality and range queries on columns.

### Example:

Imagine we have a `Customers` table:

```sql
sql
Copy code
CREATE TABLE Customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    age INT
);

```

Now, let's create a B-tree index on the `email` column to speed up searches:

```sql
sql
Copy code
CREATE INDEX idx_email ON Customers (email);

```

Now, if we run this query:

```sql
sql
Copy code
SELECT * FROM Customers WHERE email = 'john@example.com';

```

PostgreSQL will use the B-tree index to find the row efficiently, rather than scanning the entire table.

### Range Query Example:

Let's say you want to find all customers between ages 20 and 30:

```sql
sql
Copy code
SELECT * FROM Customers WHERE age BETWEEN 20 AND 30;

```

If there’s an index on the `age` column:

```sql
sql
Copy code
CREATE INDEX idx_age ON Customers (age);

```

This will speed up the query significantly because PostgreSQL can look up the relevant rows directly using the index.

### 2. **Hash Index**

- **Description**: Hash indexes are used only for **equality comparisons** (`=`). They are faster than B-tree for this type of operation but cannot be used for range queries (`<`, `>`, `BETWEEN`).
- **Use Case**: Suitable for exact lookups on columns with unique or near-unique values.

### Example:

If you frequently query customers by email, and only for exact matches, you could use a hash index:

```sql
sql
Copy code
CREATE INDEX idx_hash_email ON Customers USING hash (email);

```

Now, queries like this will benefit from the hash index:

```sql
sql
Copy code
SELECT * FROM Customers WHERE email = 'jane@example.com';

```

However, this index would not be useful for range queries like `BETWEEN` or `>`, unlike B-tree indexes.

### 3. **GIN (Generalized Inverted Index)**

- **Description**: GIN indexes are used for indexing **composite values** such as arrays, JSONB, or full-text search. They allow you to quickly search for elements within complex data types.
- **Use Case**: Ideal for array columns, JSON data, or full-text search.

### Example: Indexing an Array Column

Let's say you have a table of products where each product has an array of tags:

```sql
sql
Copy code
CREATE TABLE Products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[]
);

```

You can create a GIN index to make searching for tags faster:

```sql
sql
Copy code
CREATE INDEX idx_tags_gin ON Products USING gin (tags);

```

Now, queries like this will be efficient:

```sql
sql
Copy code
SELECT * FROM Products WHERE tags @> ARRAY['electronics'];

```

This query checks if the `tags` array contains the value `'electronics'`. The GIN index makes this operation fast.

### Example: Full-Text Search with GIN Index

For a `documents` table where you're storing text and want fast full-text search:

```sql
sql
Copy code
CREATE TABLE documents (
    doc_id SERIAL PRIMARY KEY,
    content TEXT
);

-- Create a GIN index for full-text search
CREATE INDEX idx_content_gin ON documents USING gin(to_tsvector('english', content));

```

Now, you can quickly search for documents containing certain words:

```sql
sql
Copy code
SELECT * FROM documents WHERE to_tsvector('english', content) @@ to_tsquery('search_term');

```

### 4. **GiST (Generalized Search Tree)**

- **Description**: GiST indexes are flexible and can handle complex data types such as **geometric data**, **ranges**, and more. They are used for spatial data, like points, lines, and polygons.
- **Use Case**: Commonly used for indexing geometric shapes or ranges (e.g., date ranges).

### Example: Indexing Geometric Data

Let’s say you have a table with geographic points (latitude, longitude):

```sql
sql
Copy code
CREATE TABLE Locations (
    location_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    coordinates POINT
);

```

You can use a GiST index to make spatial queries faster:

```sql
sql
Copy code
CREATE INDEX idx_coordinates_gist ON Locations USING gist (coordinates);

```

Now, you can perform fast spatial queries, like finding all points near a specific location:

```sql
sql
Copy code
SELECT * FROM Locations WHERE coordinates <@> POINT(40.7128, -74.0060) < 10;

```

This query finds all locations within 10 units of the given point.

### 5. **BRIN (Block Range INdex)**

- **Description**: BRIN indexes are compact and very efficient for large tables where data is ordered. Instead of indexing each individual row, they index **blocks** of data. This makes BRIN indexes extremely space-efficient but suitable only for certain types of queries.
- **Use Case**: Ideal for huge tables where the column data is naturally ordered, like timestamped data.

### Example: Indexing a Large Table with a Date Column

Suppose you have an `Orders` table with millions of rows and an `order_date` column:

```sql
sql
Copy code
CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

```

For such a large table, a B-tree index might be too expensive in terms of storage. You can use a BRIN index instead:

```sql
sql
Copy code
CREATE INDEX idx_order_date_brin ON Orders USING brin (order_date);

```

If you query the table for a specific date range:

```sql
sql
Copy code
SELECT * FROM Orders WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';

```

The BRIN index will efficiently locate the blocks where the relevant orders are stored, making the query faster than scanning the entire table.

---

### When NOT to Use Indexes:

1. **Small Tables**: For small tables, sequential scans (scanning every row) can be just as fast or faster than using an index. The overhead of maintaining an index might not be worth it.
2. **Frequent Writes**: If a table is frequently updated (`INSERT`, `UPDATE`, `DELETE`), too many indexes can slow down these operations because PostgreSQL must update the indexes each time the table changes.
3. **Low-Selectivity Columns**: Avoid indexing columns that have very few unique values. For example, a column with only `TRUE` and `FALSE` values (a **low-selectivity** column) will not benefit much from indexing, since the database will still need to scan most of the rows.

---

### Summary:

- **Indexes** improve the performance of data retrieval, but they come with costs in terms of storage and write performance.
- The most commonly used index type is the **B-tree**, which is great for equality and range queries.
- Other index types like **Hash**, **GIN**, **GiST**, and **BRIN** have specialized uses, such as full-text search, geometric data, or large tables with ordered data.