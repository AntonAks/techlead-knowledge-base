# CRUD Operations in NoSQL (MongoDB & DynamoDB)

## MongoDB CRUD Operations

### Data Model Overview
MongoDB is a document-oriented database that stores data in BSON (Binary JSON) format. Documents are organized into collections, and each document can have a flexible schema.

**Key Concepts**:
- **Database**: Container for collections
- **Collection**: Group of documents (similar to table in RDBMS)
- **Document**: Individual record in BSON format
- **Field**: Key-value pair within a document
- **_id**: Unique identifier for each document (auto-generated if not provided)

### Create Operations

#### Insert Single Document
```javascript
// Basic insert
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  age: 30,
  city: "New York",
  interests: ["reading", "traveling", "coding"]
});

// Insert with custom _id
db.users.insertOne({
  _id: ObjectId("64a7b8c9d1e2f3a4b5c6d7e8"),
  name: "Jane Smith",
  email: "jane@example.com",
  age: 25,
  profile: {
    bio: "Software developer",
    skills: ["JavaScript", "Python", "MongoDB"]
  }
});

// Insert with timestamp
db.users.insertOne({
  name: "Bob Johnson",
  email: "bob@example.com",
  createdAt: new Date(),
  updatedAt: new Date()
});
```

#### Insert Multiple Documents
```javascript
// Insert array of documents
db.users.insertMany([
  {
    name: "Alice Wilson",
    email: "alice@example.com",
    age: 28,
    department: "Engineering"
  },
  {
    name: "Charlie Brown",
    email: "charlie@example.com",
    age: 35,
    department: "Marketing"
  },
  {
    name: "Diana Prince",
    email: "diana@example.com",
    age: 32,
    department: "Design"
  }
]);

// Insert with options
db.users.insertMany([
  {name: "User1", email: "user1@example.com"},
  {name: "User2", email: "user2@example.com"}
], {
  ordered: false,  // Continue insertion even if one fails
  writeConcern: { w: "majority", j: true }  // Write concern options
});
```

### Read Operations

#### Basic Find Operations
```javascript
// Find all documents
db.users.find();

// Find all documents with pretty formatting
db.users.find().pretty();

// Find with criteria
db.users.find({age: {$gte: 18}});

// Find one document
db.users.findOne({email: "john@example.com"});

// Find with multiple conditions
db.users.find({
  age: {$gte: 25, $lte: 35},
  department: "Engineering"
});
```

#### Advanced Query Operators
```javascript
// Comparison operators
db.users.find({age: {$gt: 30}});           // Greater than
db.users.find({age: {$gte: 30}});          // Greater than or equal
db.users.find({age: {$lt: 30}});           // Less than
db.users.find({age: {$lte: 30}});          // Less than or equal
db.users.find({age: {$ne: 30}});           // Not equal
db.users.find({age: {$in: [25, 30, 35]}});  // In array
db.users.find({age: {$nin: [25, 30]}});     // Not in array

// Logical operators
db.users.find({
  $and: [
    {age: {$gte: 25}},
    {department: "Engineering"}
  ]
});

db.users.find({
  $or: [
    {age: {$lt: 25}},
    {age: {$gt: 35}}
  ]
});

db.users.find({age: {$not: {$gt: 30}}});

// Array operators
db.users.find({interests: "reading"});              // Contains element
db.users.find({interests: {$all: ["reading", "coding"]}}); // Contains all
db.users.find({interests: {$size: 3}});             // Array size
db.users.find({"interests.0": "reading"});          // First element

// Text search
db.users.find({$text: {$search: "software developer"}});

// Regular expressions
db.users.find({name: /^John/});                     // Starts with "John"
db.users.find({email: /@example\.com$/});           // Ends with "@example.com"
```

#### Projection (Field Selection)
```javascript
// Include specific fields
db.users.find({}, {name: 1, email: 1, _id: 0});

// Exclude specific fields
db.users.find({}, {interests: 0, profile: 0});

// Nested field projection
db.users.find({}, {"profile.skills": 1, name: 1, _id: 0});

// Array element projection
db.users.find({}, {interests: {$slice: 2}});        // First 2 elements
db.users.find({}, {interests: {$slice: [1, 2]}});   // Skip 1, limit 2
```

#### Sorting and Limiting
```javascript
// Sort ascending (1) and descending (-1)
db.users.find().sort({age: 1});                     // Sort by age ascending
db.users.find().sort({age: -1, name: 1});           // Sort by age desc, then name asc

// Limit and skip
db.users.find().limit(5);                           // First 5 documents
db.users.find().skip(10).limit(5);                  // Skip 10, then 5 documents

// Pagination example
const page = 2;
const pageSize = 10;
db.users.find()
  .skip((page - 1) * pageSize)
  .limit(pageSize)
  .sort({createdAt: -1});
```

#### Aggregation Pipeline
```javascript
// Basic aggregation
db.users.aggregate([
  {$match: {age: {$gte: 25}}},
  {$group: {
    _id: "$department",
    count: {$sum: 1},
    avgAge: {$avg: "$age"}
  }},
  {$sort: {count: -1}}
]);

// Complex aggregation with multiple stages
db.users.aggregate([
  // Match stage
  {$match: {age: {$gte: 18}}},
  
  // Add computed fields
  {$addFields: {
    fullName: {$concat: ["$firstName", " ", "$lastName"]},
    ageGroup: {
      $switch: {
        branches: [
          {case: {$lt: ["$age", 25]}, then: "Young"},
          {case: {$lt: ["$age", 35]}, then: "Adult"},
          {case: {$gte: ["$age", 35]}, then: "Senior"}
        ]
      }
    }
  }},
  
  // Group by age group
  {$group: {
    _id: "$ageGroup",
    count: {$sum: 1},
    users: {$push: "$fullName"}
  }},
  
  // Sort results
  {$sort: {count: -1}},
  
  // Limit results
  {$limit: 10}
]);
```

### Update Operations

#### Update Single Document
```javascript
// Update with $set operator
db.users.updateOne(
  {email: "john@example.com"},
  {$set: {age: 31, city: "San Francisco"}}
);

// Update with multiple operators
db.users.updateOne(
  {_id: ObjectId("64a7b8c9d1e2f3a4b5c6d7e8")},
  {
    $set: {updatedAt: new Date()},
    $inc: {loginCount: 1},
    $push: {interests: "photography"}
  }
);

// Update nested fields
db.users.updateOne(
  {email: "jane@example.com"},
  {$set: {
    "profile.bio": "Senior Software Developer",
    "profile.location": "Seattle"
  }}
);
```

#### Update Multiple Documents
```javascript
// Update all matching documents
db.users.updateMany(
  {department: "Engineering"},
  {
    $set: {updatedAt: new Date()},
    $inc: {salary: 5000}
  }
);

// Update with array operators
db.users.updateMany(
  {},
  {$addToSet: {interests: "technology"}}  // Add if not exists
);
```

#### Advanced Update Operators
```javascript
// Increment/decrement
db.users.updateOne(
  {email: "john@example.com"},
  {$inc: {age: 1, score: -10}}
);

// Array operations
db.users.updateOne(
  {email: "john@example.com"},
  {
    $push: {interests: "swimming"},           // Add to array
    $addToSet: {skills: "Docker"},           // Add if not exists
    $pull: {interests: "reading"},           // Remove from array
    $pop: {recentActivities: 1}              // Remove last element
  }
);

// Array element update
db.users.updateOne(
  {email: "john@example.com", "skills": "JavaScript"},
  {$set: {"skills.$": "Advanced JavaScript"}}  // Update matched array element
);

// Positional update with arrayFilters
db.users.updateOne(
  {email: "john@example.com"},
  {$set: {"projects.$[elem].status": "completed"}},
  {arrayFilters: [{"elem.name": "Project A"}]}
);
```

#### Upsert Operations
```javascript
// Update or insert if document doesn't exist
db.users.updateOne(
  {email: "newuser@example.com"},
  {
    $set: {
      name: "New User",
      createdAt: new Date(),
      updatedAt: new Date()
    }
  },
  {upsert: true}
);

// Replace entire document or insert
db.users.replaceOne(
  {email: "john@example.com"},
  {
    name: "John Doe Updated",
    email: "john@example.com",
    age: 32,
    updatedAt: new Date()
  },
  {upsert: true}
);
```

### Delete Operations

#### Delete Single Document
```javascript
// Delete one document
db.users.deleteOne({email: "john@example.com"});

// Delete with criteria
db.users.deleteOne({age: {$lt: 18}});
```

#### Delete Multiple Documents
```javascript
// Delete all matching documents
db.users.deleteMany({department: "Marketing"});

// Delete with complex criteria
db.users.deleteMany({
  $and: [
    {age: {$lt: 25}},
    {lastLogin: {$lt: new Date(Date.now() - 365*24*60*60*1000)}}
  ]
});

// Delete all documents in collection
db.users.deleteMany({});
```

## DynamoDB CRUD Operations

### Data Model Overview
DynamoDB is a key-value and document NoSQL database service. Data is organized into tables, and each item must have a primary key.

**Key Concepts**:
- **Table**: Collection of items
- **Item**: Individual record (similar to row in RDBMS)
- **Attribute**: Key-value pair within an item
- **Primary Key**: Uniquely identifies each item (partition key + optional sort key)
- **Partition Key**: Determines which partition stores the item
- **Sort Key**: Used for sorting items within the same partition

### Create Operations

#### Put Single Item
```python
import boto3
from boto3.dynamodb.conditions import Key, Attr

# Initialize DynamoDB client
dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
table = dynamodb.Table('Users')

# Put single item
response = table.put_item(
    Item={
        'userId': 'user123',
        'email': 'john@example.com',
        'name': 'John Doe',
        'age': 30,
        'department': 'Engineering',
        'skills': ['Python', 'JavaScript', 'AWS'],
        'profile': {
            'bio': 'Software developer',
            'location': 'New York'
        },
        'createdAt': '2024-01-15T10:30:00Z'
    }
)

# Put item with condition (only if doesn't exist)
response = table.put_item(
    Item={
        'userId': 'user124',
        'email': 'jane@example.com',
        'name': 'Jane Smith',
        'age': 28
    },
    ConditionExpression='attribute_not_exists(userId)'
)
```

#### Batch Write Operations
```python
# Batch write items (up to 25 items per batch)
with table.batch_writer() as batch:
    batch.put_item(Item={
        'userId': 'user125',
        'email': 'alice@example.com',
        'name': 'Alice Wilson',
        'age': 27
    })
    batch.put_item(Item={
        'userId': 'user126',
        'email': 'bob@example.com',
        'name': 'Bob Johnson',
        'age': 35
    })
    batch.put_item(Item={
        'userId': 'user127',
        'email': 'charlie@example.com',
        'name': 'Charlie Brown',
        'age': 32
    })

# Manual batch write with error handling
response = dynamodb.batch_write_item(
    RequestItems={
        'Users': [
            {
                'PutRequest': {
                    'Item': {
                        'userId': 'user128',
                        'email': 'diana@example.com',
                        'name': 'Diana Prince'
                    }
                }
            },
            {
                'PutRequest': {
                    'Item': {
                        'userId': 'user129',
                        'email': 'eve@example.com',
                        'name': 'Eve Adams'
                    }
                }
            }
        ]
    }
)

# Handle unprocessed items
unprocessed = response.get('UnprocessedItems', {})
while unprocessed:
    response = dynamodb.batch_write_item(RequestItems=unprocessed)
    unprocessed = response.get('UnprocessedItems', {})
```

### Read Operations

#### Get Single Item
```python
# Get item by primary key
response = table.get_item(
    Key={'userId': 'user123'}
)
item = response.get('Item')

# Get item with projection (specific attributes)
response = table.get_item(
    Key={'userId': 'user123'},
    ProjectionExpression='#name, email, age',
    ExpressionAttributeNames={'#name': 'name'}  # 'name' is reserved word
)

# Get item with consistent read
response = table.get_item(
    Key={'userId': 'user123'},
    ConsistentRead=True
)
```

#### Batch Get Operations
```python
# Batch get items (up to 100 items per batch)
response = dynamodb.batch_get_item(
    RequestItems={
        'Users': {
            'Keys': [
                {'userId': 'user123'},
                {'userId': 'user124'},
                {'userId': 'user125'}
            ],
            'ProjectionExpression': '#name, email, age',
            'ExpressionAttributeNames': {'#name': 'name'}
        }
    }
)

items = response['Responses']['Users']

# Handle unprocessed keys
unprocessed = response.get('UnprocessedKeys', {})
while unprocessed:
    response = dynamodb.batch_get_item(RequestItems=unprocessed)
    items.extend(response['Responses'].get('Users', []))
    unprocessed = response.get('UnprocessedKeys', {})
```

#### Query Operations (requires partition key)
```python
# Query items with partition key
response = table.query(
    KeyConditionExpression=Key('userId').eq('user123')
)

# Query with sort key condition (if table has sort key)
# Assuming table has composite key: userId (partition) + timestamp (sort)
response = table.query(
    KeyConditionExpression=Key('userId').eq('user123') & 
                          Key('timestamp').between('2024-01-01', '2024-12-31')
)

# Query with filter expression
response = table.query(
    KeyConditionExpression=Key('userId').eq('user123'),
    FilterExpression=Attr('age').gte(25) & Attr('department').eq('Engineering')
)

# Query with pagination
response = table.query(
    KeyConditionExpression=Key('userId').eq('user123'),
    Limit=10,
    ScanIndexForward=False  # Descending order
)

# Continue pagination with LastEvaluatedKey
last_key = response.get('LastEvaluatedKey')
if last_key:
    response = table.query(
        KeyConditionExpression=Key('userId').eq('user123'),
        ExclusiveStartKey=last_key,
        Limit=10
    )
```

#### Scan Operations (expensive, use sparingly)
```python
# Scan entire table
response = table.scan()

# Scan with filter
response = table.scan(
    FilterExpression=Attr('age').between(25, 35) & 
                    Attr('department').eq('Engineering')
)

# Scan with projection
response = table.scan(
    ProjectionExpression='userId, #name, email',
    ExpressionAttributeNames={'#name': 'name'}
)

# Parallel scan for large tables
def parallel_scan(table, segment, total_segments):
    return table.scan(
        Segment=segment,
        TotalSegments=total_segments,
        FilterExpression=Attr('age').gte(18)
    )

# Scan with pagination
response = table.scan(Limit=100)
all_items = response['Items']

while 'LastEvaluatedKey' in response:
    response = table.scan(
        ExclusiveStartKey=response['LastEvaluatedKey'],
        Limit=100
    )
    all_items.extend(response['Items'])
```

### Update Operations

#### Update Single Item
```python
# Update with SET expression
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='SET age = :age, #name = :name, updatedAt = :timestamp',
    ExpressionAttributeValues={
        ':age': 31,
        ':name': 'John Doe Updated',
        ':timestamp': '2024-06-02T12:00:00Z'
    },
    ExpressionAttributeNames={'#name': 'name'},
    ReturnValues='UPDATED_NEW'
)

# Update with multiple operations
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='''
        SET age = :age, updatedAt = :timestamp
        ADD loginCount :increment, skills :newSkills
        REMOVE tempField
    ''',
    ExpressionAttributeValues={
        ':age': 32,
        ':timestamp': '2024-06-02T12:00:00Z',
        ':increment': 1,
        ':newSkills': {'Docker', 'Kubernetes'}
    },
    ReturnValues='ALL_NEW'
)

# Conditional update
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='SET age = :age',
    ConditionExpression='age < :maxAge',
    ExpressionAttributeValues={
        ':age': 33,
        ':maxAge': 50
    }
)

# Update nested attributes
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='SET profile.bio = :bio, profile.#loc = :location',
    ExpressionAttributeValues={
        ':bio': 'Senior Software Developer',
        ':location': 'San Francisco'
    },
    ExpressionAttributeNames={'#loc': 'location'}
)
```

#### Atomic Counters and Sets
```python
# Increment counter
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='ADD viewCount :increment',
    ExpressionAttributeValues={':increment': 1}
)

# Add to string set
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='ADD skills :newSkill',
    ExpressionAttributeValues={':newSkill': {'React'}}
)

# Remove from string set
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='DELETE skills :oldSkill',
    ExpressionAttributeValues={':oldSkill': {'jQuery'}}
)

# Update list elements
response = table.update_item(
    Key={'userId': 'user123'},
    UpdateExpression='SET projects[0].#status = :status',
    ExpressionAttributeValues={':status': 'completed'},
    ExpressionAttributeNames={'#status': 'status'}
)
```

### Delete Operations

#### Delete Single Item
```python
# Delete item by primary key
response = table.delete_item(
    Key={'userId': 'user123'}
)

# Conditional delete
response = table.delete_item(
    Key={'userId': 'user123'},
    ConditionExpression='attribute_exists(userId) AND age > :minAge',
    ExpressionAttributeValues={':minAge': 18}
)

# Delete and return old values
response = table.delete_item(
    Key={'userId': 'user123'},
    ReturnValues='ALL_OLD'
)
deleted_item = response.get('Attributes')
```

#### Batch Delete Operations
```python
# Batch delete using batch_writer
with table.batch_writer() as batch:
    batch.delete_item(Key={'userId': 'user124'})
    batch.delete_item(Key={'userId': 'user125'})
    batch.delete_item(Key={'userId': 'user126'})

# Manual batch delete
response = dynamodb.batch_write_item(
    RequestItems={
        'Users': [
            {'DeleteRequest': {'Key': {'userId': 'user127'}}},
            {'DeleteRequest': {'Key': {'userId': 'user128'}}},
            {'DeleteRequest': {'Key': {'userId': 'user129'}}}
        ]
    }
)
```

## Key Differences Between MongoDB and DynamoDB

### Data Model
| Aspect | MongoDB | DynamoDB |
|--------|----------|----------|
| **Structure** | Document-based (JSON-like) | Key-value with attributes |
| **Schema** | Flexible, dynamic schema | Key schema required, attributes flexible |
| **Primary Key** | _id field (auto-generated) | Partition key + optional sort key |
| **Collections/Tables** | Collections of documents | Tables of items |

### Query Capabilities
| Operation | MongoDB | DynamoDB |
|-----------|----------|----------|
| **Ad-hoc Queries** | Rich query language | Limited to key-based queries |
| **Indexing** | Multiple index types | GSI/LSI for non-key queries |
| **Aggregation** | Powerful aggregation pipeline | Limited aggregation |
| **Joins** | $lookup for joins | No native joins |
| **Transactions** | Multi-document ACID | Limited transactions |

### Performance and Scaling
| Aspect | MongoDB | DynamoDB |
|--------|----------|----------|
| **Scaling** | Manual sharding | Automatic horizontal scaling |
| **Performance** | Depends on hardware/config | Provisioned/on-demand capacity |
| **Consistency** | Configurable | Eventually consistent (strongly consistent option) |
| **Latency** | Variable | Single-digit millisecond latency |

### Best Practices

#### MongoDB Best Practices
- Design schema based on query patterns
- Use indexes to optimize query performance
- Implement proper error handling for duplicate keys
- Use bulk operations for better performance
- Monitor slow queries and optimize accordingly
- Implement proper connection pooling

#### DynamoDB Best Practices
- Design partition keys to distribute load evenly
- Use sort keys for range queries and sorting
- Implement exponential backoff for throttling
- Use batch operations when possible
- Monitor consumed capacity units
- Design for eventual consistency
- Use Global Secondary Indexes sparingly
- Implement proper error handling for conditional operations

### Error Handling Examples

#### MongoDB Error Handling
```javascript
try {
  const result = await db.users.insertOne({
    email: "duplicate@example.com",
    name: "Test User"
  });
} catch (error) {
  if (error.code === 11000) {
    console.log("Duplicate key error:", error.keyValue);
  } else {
    console.log("Other error:", error.message);
  }
}
```

#### DynamoDB Error Handling
```python
from botocore.exceptions import ClientError

try:
    response = table.put_item(
        Item={'userId': 'user123', 'name': 'John'},
        ConditionExpression='attribute_not_exists(userId)'
    )
except ClientError as e:
    error_code = e.response['Error']['Code']
    if error_code == 'ConditionalCheckFailedException':
        print("Item already exists")
    elif error_code == 'ProvisionedThroughputExceededException':
        print("Throughput exceeded, implement backoff")
    else:
        print(f"Unexpected error: {e}")
```