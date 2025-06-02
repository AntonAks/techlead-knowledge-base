# Indexing in NoSQL Databases

## Overview
Indexing in NoSQL databases improves query performance by creating data structures that allow faster data retrieval. Different NoSQL databases implement various indexing strategies based on their data models and use cases.

## MongoDB Indexing

### Index Types

#### Single Field Index
The most basic type of index, created on a single field.

```javascript
// Create ascending index
db.users.createIndex({email: 1})

// Create descending index
db.users.createIndex({age: -1})

// Create index with options
db.users.createIndex(
  {username: 1}, 
  {
    unique: true,
    background: true,
    name: "username_unique_idx"
  }
)
```

#### Compound Index
Index on multiple fields, order matters for query optimization.

```javascript
// Create compound index
db.users.createIndex({status: 1, age: -1, city: 1})

// Query patterns that can use this index:
// - {status: "active"}
// - {status: "active", age: {$gte: 25}}
// - {status: "active", age: {$gte: 25}, city: "NYC"}

// This query cannot use the index efficiently:
// - {age: {$gte: 25}, city: "NYC"} // Missing status prefix
```

#### Text Index
Enables full-text search capabilities.

```javascript
// Create text index on single field
db.articles.createIndex({title: "text"})

// Create text index on multiple fields
db.articles.createIndex({
  title: "text",
  content: "text",
  tags: "text"
})

// Create text index with weights
db.articles.createIndex(
  {
    title: "text",
    content: "text"
  },
  {
    weights: {
      title: 10,
      content: 5
    }
  }
)

// Text search query
db.articles.find({$text: {$search: "mongodb indexing"}})
```

#### Geospatial Indexes

##### 2dsphere Index (for spherical geometry)
```javascript
// Create 2dsphere index
db.locations.createIndex({coordinates: "2dsphere"})

// Store GeoJSON data
db.locations.insertOne({
  name: "Central Park",
  coordinates: {
    type: "Point",
    coordinates: [-73.968285, 40.785091] // [longitude, latitude]
  }
})

// Geospatial queries
// Find locations within radius
db.locations.find({
  coordinates: {
    $geoWithin: {
      $centerSphere: [[-73.968285, 40.785091], 1/3963.2] // 1 mile radius
    }
  }
})

// Find locations near a point
db.locations.find({
  coordinates: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.968285, 40.785091]
      },
      $maxDistance: 1000 // meters
    }
  }
})
```

##### 2d Index (for flat geometry)
```javascript
// Create 2d index
db.places.createIndex({location: "2d"})

// Store coordinate data
db.places.insertOne({
  name: "Store A",
  location: [40.7128, -74.0060] // [latitude, longitude]
})

// Query with 2d index
db.places.find({
  location: {
    $near: [40.7128, -74.0060],
    $maxDistance: 0.1
  }
})
```

#### Multikey Index
Automatically created when indexing array fields.

```javascript
// Create index on array field
db.products.createIndex({tags: 1})

// Insert document with array
db.products.insertOne({
  name: "Laptop",
  tags: ["electronics", "computers", "portable"]
})

// Query can match any array element
db.products.find({tags: "electronics"})
db.products.find({tags: {$in: ["computers", "mobile"]}})
```

#### Hashed Index
Distributes documents evenly, good for sharding.

```javascript
// Create hashed index
db.users.createIndex({userId: "hashed"})

// Useful for sharding key
sh.shardCollection("mydb.users", {userId: "hashed"})
```

#### Wildcard Index
Indexes all fields in embedded documents.

```javascript
// Create wildcard index
db.products.createIndex({"attributes.$**": 1})

// Insert documents with varying structures
db.products.insertMany([
  {
    name: "Laptop",
    attributes: {
      brand: "Dell",
      processor: "Intel i7",
      ram: "16GB"
    }
  },
  {
    name: "Phone",
    attributes: {
      brand: "Apple",
      storage: "256GB",
      color: "Black"
    }
  }
])

// Query any attribute field
db.products.find({"attributes.brand": "Dell"})
db.products.find({"attributes.storage": "256GB"})
```

### Index Properties

#### Unique Index
Prevents duplicate values in indexed field(s).

```javascript
// Create unique index
db.users.createIndex({email: 1}, {unique: true})

// Create unique compound index
db.users.createIndex({username: 1, tenant_id: 1}, {unique: true})

// Handle duplicate key errors
try {
  db.users.insertOne({email: "duplicate@example.com"})
} catch (error) {
  if (error.code === 11000) {
    console.log("Duplicate email address")
  }
}
```

#### Partial Index
Indexes only documents matching specified criteria.

```javascript
// Create partial index for active users only
db.users.createIndex(
  {email: 1},
  {
    partialFilterExpression: {
      status: "active",
      age: {$gte: 18}
    }
  }
)

// Create partial index excluding null values
db.users.createIndex(
  {phone: 1},
  {
    partialFilterExpression: {
      phone: {$exists: true, $ne: null}
    }
  }
)
```

#### Sparse Index
Excludes documents without the indexed field.

```javascript
// Create sparse index
db.users.createIndex({phone: 1}, {sparse: true})

// Only documents with phone field are indexed
// Documents without phone field won't appear in index-only queries
```

#### TTL (Time To Live) Index
Automatically removes documents after specified time.

```javascript
// Create TTL index - documents expire after 30 days
db.sessions.createIndex(
  {createdAt: 1},
  {expireAfterSeconds: 2592000} // 30 days in seconds
)

// Create TTL index with date field
db.logs.createIndex(
  {expiryDate: 1},
  {expireAfterSeconds: 0} // Expire at the specified date
)

// Insert document that will expire
db.sessions.insertOne({
  userId: "user123",
  sessionId: "session456",
  createdAt: new Date()
})
```

### Index Management

#### List Indexes
```javascript
// List all indexes in collection
db.users.getIndexes()

// List index names only
db.users.getIndexes().forEach(index => print(index.name))
```

#### Drop Indexes
```javascript
// Drop specific index by name
db.users.dropIndex("email_1")

// Drop index by specification
db.users.dropIndex({email: 1})

// Drop all indexes except _id
db.users.dropIndexes()
```

#### Index Statistics
```javascript
// Get index usage statistics
db.users.aggregate([{$indexStats: {}}])

// Check if query uses index
db.users.find({email: "john@example.com"}).explain("executionStats")
```

## Cassandra Indexing

### Primary Index
Automatically created on partition key and clustering columns.

```sql
-- Table with compound primary key
CREATE TABLE user_activities (
  user_id UUID,
  activity_date DATE,
  activity_time TIME,
  activity_type TEXT,
  details TEXT,
  PRIMARY KEY (user_id, activity_date, activity_time)
);

-- Partition key: user_id
-- Clustering columns: activity_date, activity_time
-- Automatically indexed for efficient queries
```

### Secondary Index
Created on non-primary key columns for additional query patterns.

```sql
-- Create secondary index
CREATE INDEX ON users (email);
CREATE INDEX ON users (age);
CREATE INDEX IF NOT EXISTS users_city_idx ON users (city);

-- Query using secondary index
SELECT * FROM users WHERE email = 'john@example.com';
SELECT * FROM users WHERE age > 25;

-- Note: Secondary indexes in Cassandra have limitations
-- - Not suitable for high cardinality fields
-- - Can cause performance issues if not used carefully
-- - Better to use materialized views for complex queries
```

### Search Index (with DSE)
DataStax Enterprise provides Solr-based search indexes.

```sql
-- Create search index
CREATE SEARCH INDEX ON products;

-- Search queries
SELECT * FROM products WHERE solr_query = 'name:smartphone';
SELECT * FROM products WHERE solr_query = 'description:fast AND category:electronics';
```

### Materialized Views
Precomputed views with different primary keys for alternative query patterns.

```sql
-- Base table
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email TEXT,
  name TEXT,
  city TEXT,
  age INT
);

-- Materialized view for querying by email
CREATE MATERIALIZED VIEW users_by_email AS
  SELECT user_id, email, name, city, age
  FROM users
  WHERE email IS NOT NULL AND user_id IS NOT NULL
  PRIMARY KEY (email, user_id);

-- Materialized view for querying by city
CREATE MATERIALIZED VIEW users_by_city AS
  SELECT user_id, email, name, city, age
  FROM users
  WHERE city IS NOT NULL AND user_id IS NOT NULL
  PRIMARY KEY (city, user_id);

-- Query materialized views
SELECT * FROM users_by_email WHERE email = 'john@example.com';
SELECT * FROM users_by_city WHERE city = 'New York';
```

### Index Best Practices for Cassandra
```sql
-- Good: Query by partition key
SELECT * FROM user_activities WHERE user_id = uuid_value;

-- Good: Query by partition key + clustering columns
SELECT * FROM user_activities 
WHERE user_id = uuid_value AND activity_date = '2024-06-01';

-- Avoid: Query without partition key (full table scan)
-- SELECT * FROM user_activities WHERE activity_type = 'login';

-- Better: Use materialized view
CREATE MATERIALIZED VIEW activities_by_type AS
  SELECT user_id, activity_date, activity_time, activity_type, details
  FROM user_activities
  WHERE activity_type IS NOT NULL AND user_id IS NOT NULL 
    AND activity_date IS NOT NULL AND activity_time IS NOT NULL
  PRIMARY KEY (activity_type, user_id, activity_date, activity_time);
```

## DynamoDB Indexing

### Global Secondary Index (GSI)
Different partition key from main table, eventually consistent.

```python
import boto3

# Create table with GSI
dynamodb = boto3.resource('dynamodb')

table = dynamodb.create_table(
    TableName='Users',
    KeySchema=[
        {'AttributeName': 'userId', 'KeyType': 'HASH'}
    ],
    AttributeDefinitions=[
        {'AttributeName': 'userId', 'AttributeType': 'S'},
        {'AttributeName': 'email', 'AttributeType': 'S'},
        {'AttributeName': 'city', 'AttributeType': 'S'},
        {'AttributeName': 'age', 'AttributeType': 'N'}
    ],
    GlobalSecondaryIndexes=[
        {
            'IndexName': 'EmailIndex',
            'KeySchema': [
                {'AttributeName': 'email', 'KeyType': 'HASH'}
            ],
            'Projection': {'ProjectionType': 'ALL'},
            'ProvisionedThroughput': {
                'ReadCapacityUnits': 5,
                'WriteCapacityUnits': 5
            }
        },
        {
            'IndexName': 'CityAgeIndex',
            'KeySchema': [
                {'AttributeName': 'city', 'KeyType': 'HASH'},
                {'AttributeName': 'age', 'KeyType': 'RANGE'}
            ],
            'Projection': {
                'ProjectionType': 'INCLUDE',
                'NonKeyAttributes': ['name', 'email']
            },
            'ProvisionedThroughput': {
                'ReadCapacityUnits': 5,
                'WriteCapacityUnits': 5
            }
        }
    ],
    ProvisionedThroughput={
        'ReadCapacityUnits': 10,
        'WriteCapacityUnits': 10
    }
)

# Query GSI
response = table.query(
    IndexName='EmailIndex',
    KeyConditionExpression='email = :email',
    ExpressionAttributeValues={':email': 'john@example.com'}
)

# Query GSI with range key
response = table.query(
    IndexName='CityAgeIndex',
    KeyConditionExpression='city = :city AND age BETWEEN :min_age AND :max_age',
    ExpressionAttributeValues={
        ':city': 'New York',
        ':min_age': 25,
        ':max_age': 35
    }
)
```

### Local Secondary Index (LSI)
Same partition key as main table, different sort key, strongly consistent.

```python
# Create table with LSI
table = dynamodb.create_table(
    TableName='UserPosts',
    KeySchema=[
        {'AttributeName': 'userId', 'KeyType': 'HASH'},
        {'AttributeName': 'postId', 'KeyType': 'RANGE'}
    ],
    AttributeDefinitions=[
        {'AttributeName': 'userId', 'AttributeType': 'S'},
        {'AttributeName': 'postId', 'AttributeType': 'S'},
        {'AttributeName': 'timestamp', 'AttributeType': 'N'},
        {'AttributeName': 'category', 'AttributeType': 'S'}
    ],
    LocalSecondaryIndexes=[
        {
            'IndexName': 'TimestampIndex',
            'KeySchema': [
                {'AttributeName': 'userId', 'KeyType': 'HASH'},
                {'AttributeName': 'timestamp', 'KeyType': 'RANGE'}
            ],
            'Projection': {'ProjectionType': 'ALL'}
        },
        {
            'IndexName': 'CategoryIndex',
            'KeySchema': [
                {'AttributeName': 'userId', 'KeyType': 'HASH'},
                {'AttributeName': 'category', 'KeyType': 'RANGE'}
            ],
            'Projection': {
                'ProjectionType': 'KEYS_ONLY'
            }
        }
    ],
    ProvisionedThroughput={
        'ReadCapacityUnits': 10,
        'WriteCapacityUnits': 10
    }
)

# Query LSI (strongly consistent)
response = table.query(
    IndexName='TimestampIndex',
    KeyConditionExpression='userId = :userId AND #ts BETWEEN :start AND :end',
    ExpressionAttributeNames={'#ts': 'timestamp'},
    ExpressionAttributeValues={
        ':userId': 'user123',
        ':start': 1640995200,  # Jan 1, 2022
        ':end': 1672531200     # Jan 1, 2023
    },
    ConsistentRead=True
)
```

### Projection Types in DynamoDB Indexes

```python
# ALL: Include all attributes
'Projection': {'ProjectionType': 'ALL'}

# KEYS_ONLY: Only key attributes
'Projection': {'ProjectionType': 'KEYS_ONLY'}

# INCLUDE: Specific attributes
'Projection': {
    'ProjectionType': 'INCLUDE',
    'NonKeyAttributes': ['name', 'email', 'age']
}
```

## Elasticsearch Indexing

### Inverted Index
Core data structure for text search capabilities.

```json
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "content": {
        "type": "text",
        "analyzer": "english"
      },
      "author": {
        "type": "keyword"
      },
      "publish_date": {
        "type": "date"
      },
      "tags": {
        "type": "keyword"
      }
    }
  }
}
```

### Field Types and Indexing

#### Text Fields
```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          },
          "suggest": {
            "type": "completion"
          }
        }
      }
    }
  }
}
```

#### Keyword Fields
```json
{
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      },
      "user_id": {
        "type": "keyword",
        "index": false  // Don't index, only store
      }
    }
  }
}
```

#### Numeric Fields
```json
{
  "mappings": {
    "properties": {
      "price": {
        "type": "double"
      },
      "quantity": {
        "type": "integer"
      },
      "score": {
        "type": "float"
      }
    }
  }
}
```

#### Date Fields
```json
{
  "mappings": {
    "properties": {
      "created_date": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "updated_at": {
        "type": "date"
      }
    }
  }
}
```

#### Geo Fields
```json
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      },
      "area": {
        "type": "geo_shape"
      }
    }
  }
}
```

### Custom Analyzers
```json
PUT /products
{
  "settings": {
    "analysis": {
      "analyzer": {
        "product_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "snowball"
          ]
        },
        "autocomplete_analyzer": {
          "type": "custom",
          "tokenizer": "autocomplete_tokenizer",
          "filter": ["lowercase"]
        }
      },
      "tokenizer": {
        "autocomplete_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 20,
          "token_chars": ["letter", "digit"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "product_analyzer",
        "fields": {
          "autocomplete": {
            "type": "text",
            "analyzer": "autocomplete_analyzer"
          }
        }
      }
    }
  }
}
```

## Graph Database Indexing

### Neo4j Indexing

#### Node Property Indexes
```cypher
-- Create index on single property
CREATE INDEX FOR (p:Person) ON (p.name);
CREATE INDEX FOR (c:Company) ON (c.name);

-- Create composite index
CREATE INDEX FOR (p:Person) ON (p.name, p.age);

-- Create index with specific name
CREATE INDEX person_email_idx FOR (p:Person) ON (p.email);
```

#### Relationship Property Indexes
```cypher
-- Create index on relationship property
CREATE INDEX FOR ()-[r:PURCHASED]-() ON (r.date);
CREATE INDEX FOR ()-[r:WORKS_FOR]-() ON (r.start_date, r.position);
```

#### Full-text Indexes
```cypher
-- Create full-text index
CALL db.index.fulltext.createNodeIndex(
  "personFulltext", 
  ["Person"], 
  ["name", "description"]
);

CALL db.index.fulltext.createRelationshipIndex(
  "reviewFulltext",
  ["REVIEWED"],
  ["content", "summary"]
);

-- Query full-text index
CALL db.index.fulltext.queryNodes("personFulltext", "software developer")
YIELD node, score
RETURN node.name, score;
```

#### Range Indexes
```cypher
-- Create range index for numerical/date queries
CREATE RANGE INDEX FOR (p:Person) ON (p.age);
CREATE RANGE INDEX FOR (e:Event) ON (e.date);

-- Query using range
MATCH (p:Person)
WHERE p.age > 25 AND p.age < 65
RETURN p;
```

### Index Management in Neo4j
```cypher
-- List all indexes
SHOW INDEXES;

-- Show index details
SHOW INDEXES YIELD name, type, entityType, labelsOrTypes, properties;

-- Drop index
DROP INDEX person_email_idx;

-- Check index usage
PROFILE MATCH (p:Person {email: 'john@example.com'}) RETURN p;
```

## Index Design Principles

### Consider Query Patterns
Understanding your application's query patterns is crucial for effective indexing.

#### MongoDB Query Pattern Analysis
```javascript
// Analyze slow operations
db.setProfilingLevel(2, {slowms: 100})

// Check profiling data
db.system.profile.find().limit(5).sort({ts: -1}).pretty()

// Common query patterns to index:
// 1. Equality queries
db.users.find({status: "active"})  // Index: {status: 1}

// 2. Range queries
db.orders.find({amount: {$gte: 100, $lte: 500}})  // Index: {amount: 1}

// 3. Sort operations
db.posts.find().sort({created_at: -1})  // Index: {created_at: -1}

// 4. Compound queries
db.users.find({city: "NYC", age: {$gte: 25}}).sort({name: 1})
// Index: {city: 1, age: 1, name: 1}
```

### Selectivity Considerations
High selectivity indexes are more effective.

```javascript
// High selectivity (good for indexing)
db.users.createIndex({email: 1})  // Unique values
db.users.createIndex({user_id: 1})  // High cardinality

// Low selectivity (consider carefully)
db.users.createIndex({gender: 1})  // Only 2-3 values
db.users.createIndex({status: 1})  // Few distinct values

// Improve selectivity with compound indexes
db.users.createIndex({status: 1, city: 1, age: 1})
```

### Write Performance Impact
```javascript
// Monitor write performance impact
db.users.insertMany([...largeArray])  // Test with and without indexes

// Use background index creation for large collections
db.users.createIndex({field: 1}, {background: true})

// Consider partial indexes for write-heavy collections
db.logs.createIndex(
  {level: 1},
  {partialFilterExpression: {level: {$in: ["ERROR", "FATAL"]}}}
)
```

### Index Maintenance
```javascript
// Regular index maintenance tasks
db.users.reIndex()  // Rebuild all indexes

// Monitor index usage
db.users.aggregate([{$indexStats: {}}])

// Remove unused indexes
db.users.dropIndex("unused_field_1")

// Optimize index order based on query patterns
// Put most selective fields first in compound indexes
```

## Performance Considerations

### Index Size and Memory
```javascript
// Check index sizes
db.stats()
db.users.stats().indexSizes

// Estimate index size before creation
db.users.find().count() * averageKeySize * numberOfIndexes
```

### Query Optimization
```javascript
// Use explain to optimize queries
db.users.find({city: "NYC", age: {$gte: 25}}).explain("executionStats")

// Look for:
// - "stage": "IXSCAN" (index scan)
// - "totalDocsExamined" vs "totalDocsReturned"
// - "executionTimeMillis"

// Optimize with covered queries
db.users.createIndex({city: 1, age: 1, name: 1})
db.users.find(
  {city: "NYC", age: {$gte: 25}},
  {name: 1, _id: 0}  // Only return indexed fields
).explain()
```

### Scaling Considerations
```javascript
// Sharding considerations
// Choose shard key that distributes data evenly
sh.shardCollection("mydb.users", {user_id: "hashed"})

// Index all shard keys
db.users.createIndex({user_id: 1})

// Consider compound shard keys for better distribution
sh.shardCollection("mydb.events", {user_id: 1, timestamp: 1})
```

This comprehensive guide covers indexing strategies across different NoSQL databases, providing practical examples and best practices for optimizing query performance while considering the trade-offs involved in index design and maintenance.