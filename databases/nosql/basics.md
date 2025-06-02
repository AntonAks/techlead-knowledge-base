# Basics of NoSQL - Database, Collection, Document

## What is NoSQL?

**NoSQL** (Not Only SQL) refers to non-relational database systems designed to handle large volumes of data, provide flexible schemas, and scale horizontally. Unlike traditional relational databases, NoSQL databases don't require fixed table structures and can store various data types.

## Core Concepts: Database, Collection, Document

### **Database**
The top-level container that holds collections, similar to a database in relational systems but with more flexibility.

```javascript
// MongoDB example - connecting to a database
use ecommerce_db;  // Creates/switches to database named 'ecommerce_db'

// List all databases
show dbs;

// Current database
db;
```

**Key Characteristics:**
- **Namespace**: Logical grouping of related collections
- **No Schema**: No predefined structure required
- **Flexible**: Can contain collections with different data types
- **Scalable**: Can be distributed across multiple servers

### **Collection**
A group of documents, similar to a table in relational databases but without enforced schema.

```javascript
// MongoDB - working with collections
db.customers;  // References the 'customers' collection

// Create collection explicitly (optional)
db.createCollection("products");

// List collections in current database
show collections;

// Collection operations
db.customers.insertOne({name: "John Doe", email: "john@example.com"});
```

**Collection Features:**
- **Schema-less**: Documents can have different structures
- **Dynamic**: Structure can evolve over time
- **Indexed**: Can have indexes for performance
- **Flexible**: No foreign key constraints

### **Document**
The basic unit of data storage, similar to a row in relational databases but stored in formats like JSON, BSON, or XML.

```javascript
// MongoDB document examples
// Simple document
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

// Complex nested document
{
  "_id": ObjectId("507f1f77bcf86cd799439012"),
  "customerId": "CUST001",
  "name": "Jane Smith",
  "email": "jane@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001",
    "country": "USA"
  },
  "preferences": {
    "newsletter": true,
    "notifications": ["email", "sms"],
    "theme": "dark"
  },
  "orders": [
    {
      "orderId": "ORD001",
      "date": "2024-01-15",
      "total": 99.99,
      "items": [
        {"productId": "PROD001", "quantity": 2, "price": 29.99},
        {"productId": "PROD002", "quantity": 1, "price": 39.99}
      ]
    }
  ],
  "tags": ["premium", "frequent_buyer"],
  "createdAt": ISODate("2024-01-10T10:30:00Z"),
  "lastLogin": ISODate("2024-01-20T15:45:00Z")
}
```

## Document Structure and Features

### **Flexible Schema**
Documents in the same collection can have completely different structures.

```javascript
// Customer documents with different structures
// Document 1: Individual customer
{
  "_id": ObjectId("..."),
  "type": "individual",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "phone": "+1-555-0123"
}

// Document 2: Business customer (different structure)
{
  "_id": ObjectId("..."),
  "type": "business",
  "companyName": "Acme Corp",
  "contactPerson": "Jane Smith",
  "email": "contact@acme.com",
  "taxId": "TAX123456",
  "employees": 50,
  "industry": "technology"
}

// Document 3: Minimal customer
{
  "_id": ObjectId("..."),
  "email": "minimal@example.com",
  "status": "prospect"
}
```

### **Nested Data Structures**
Documents can contain arrays, objects, and deeply nested structures.

```javascript
// Product document with complex nested structure
{
  "_id": ObjectId("..."),
  "sku": "LAPTOP001",
  "name": "Gaming Laptop",
  "category": "electronics",
  "specifications": {
    "processor": {
      "brand": "Intel",
      "model": "Core i7-12700H",
      "cores": 14,
      "baseFrequency": "2.3 GHz",
      "maxFrequency": "4.7 GHz"
    },
    "memory": {
      "size": "32 GB",
      "type": "DDR4",
      "speed": "3200 MHz"
    },
    "storage": [
      {
        "type": "SSD",
        "capacity": "1 TB",
        "interface": "NVMe"
      }
    ],
    "display": {
      "size": "15.6 inches",
      "resolution": "1920x1080",
      "refreshRate": "144 Hz",
      "panelType": "IPS"
    }
  },
  "pricing": {
    "cost": 800.00,
    "msrp": 1299.99,
    "salePrice": 1199.99,
    "currency": "USD"
  },
  "inventory": {
    "inStock": 25,
    "reserved": 5,
    "available": 20,
    "locations": [
      {"warehouse": "NY", "quantity": 15},
      {"warehouse": "CA", "quantity": 10}
    ]
  },
  "reviews": [
    {
      "userId": "USER001",
      "rating": 5,
      "comment": "Excellent performance!",
      "date": "2024-01-18",
      "helpful": 12
    }
  ],
  "tags": ["gaming", "high-performance", "portable"],
  "metadata": {
    "createdBy": "admin",
    "createdAt": "2024-01-10T09:00:00Z",
    "lastModified": "2024-01-20T14:30:00Z",
    "version": 3
  }
}
```

## Basic CRUD Operations

### **Create (Insert)**
```javascript
// Insert single document
db.customers.insertOne({
  name: "Alice Johnson",
  email: "alice@example.com",
  registrationDate: new Date(),
  status: "active"
});

// Insert multiple documents
db.customers.insertMany([
  {
    name: "Bob Wilson",
    email: "bob@example.com",
    age: 35,
    city: "Boston"
  },
  {
    name: "Carol Davis",
    email: "carol@example.com",
    age: 28,
    city: "Seattle",
    preferences: {
      newsletter: true,
      theme: "light"
    }
  }
]);
```

### **Read (Find)**
```javascript
// Find all documents
db.customers.find();

// Find with condition
db.customers.find({status: "active"});

// Find with complex conditions
db.customers.find({
  age: {$gte: 25, $lte: 40},
  city: {$in: ["New York", "Boston", "Seattle"]}
});

// Find nested fields
db.customers.find({"address.city": "New York"});

// Find with projection (select specific fields)
db.customers.find(
  {status: "active"},
  {name: 1, email: 1, _id: 0}  // Include name and email, exclude _id
);

// Find one document
db.customers.findOne({email: "alice@example.com"});
```

### **Update**
```javascript
// Update single document
db.customers.updateOne(
  {email: "alice@example.com"},
  {
    $set: {
      status: "premium",
      lastLogin: new Date()
    }
  }
);

// Update multiple documents
db.customers.updateMany(
  {city: "New York"},
  {
    $set: {region: "Northeast"}
  }
);

// Update with operators
db.customers.updateOne(
  {email: "bob@example.com"},
  {
    $inc: {loginCount: 1},           // Increment
    $push: {tags: "frequent_user"},  // Add to array
    $set: {lastSeen: new Date()}     // Set field
  }
);

// Upsert (update or insert)
db.customers.updateOne(
  {email: "new@example.com"},
  {
    $set: {
      name: "New Customer",
      status: "active",
      createdAt: new Date()
    }
  },
  {upsert: true}  // Create if doesn't exist
);
```

### **Delete**
```javascript
// Delete single document
db.customers.deleteOne({email: "alice@example.com"});

// Delete multiple documents
db.customers.deleteMany({status: "inactive"});

// Delete all documents in collection
db.customers.deleteMany({});
```

## Data Types in Documents

### **Supported Data Types**
```javascript
{
  // String
  "name": "John Doe",
  
  // Number (Integer and Float)
  "age": 30,
  "price": 99.99,
  
  // Boolean
  "isActive": true,
  
  // Date
  "createdAt": ISODate("2024-01-15T10:30:00Z"),
  
  // Array
  "tags": ["premium", "verified"],
  "numbers": [1, 2, 3, 4, 5],
  
  // Object/Subdocument
  "address": {
    "street": "123 Main St",
    "city": "New York"
  },
  
  // ObjectId
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  
  // Null
  "middleName": null,
  
  // Binary data
  "profilePicture": BinData(0, "base64encodeddata"),
  
  // Regular Expression
  "pattern": /^[a-z]+$/i,
  
  // Decimal (for precise calculations)
  "precisePrice": NumberDecimal("99.99")
}
```

## Advantages of Document Model

### **Flexibility**
- **Schema Evolution**: Add/remove fields without migrations
- **Varied Structures**: Documents can have different structures
- **Rapid Development**: No need to design schema upfront

### **Natural Data Representation**
```javascript
// Relational approach (multiple tables)
// customers table: id, name, email
// addresses table: id, customer_id, street, city, state
// orders table: id, customer_id, total, date
// order_items table: order_id, product_id, quantity, price

// NoSQL approach (single document)
{
  "_id": ObjectId("..."),
  "name": "John Doe",
  "email": "john@example.com",
  "addresses": [
    {
      "type": "home",
      "street": "123 Main St",
      "city": "New York",
      "state": "NY"
    },
    {
      "type": "work",
      "street": "456 Business Ave",
      "city": "New York",
      "state": "NY"
    }
  ],
  "orders": [
    {
      "orderId": "ORD001",
      "date": "2024-01-15",
      "total": 199.98,
      "items": [
        {"productId": "PROD001", "quantity": 2, "price": 49.99},
        {"productId": "PROD002", "quantity": 1, "price": 99.99}
      ]
    }
  ]
}
```

### **Performance Benefits**
- **Reduced Joins**: Related data stored together
- **Atomic Operations**: Update entire document in single operation
- **Locality**: Related data physically stored together

## Document Design Patterns

### **Embedding vs Referencing**

#### **Embedding (Denormalized)**
```javascript
// Blog post with embedded comments
{
  "_id": ObjectId("..."),
  "title": "Introduction to NoSQL",
  "content": "NoSQL databases are...",
  "author": {
    "name": "Jane Doe",
    "email": "jane@example.com"
  },
  "comments": [
    {
      "commentId": "CMT001",
      "author": "John Smith",
      "text": "Great article!",
      "date": "2024-01-16T10:00:00Z",
      "likes": 5
    },
    {
      "commentId": "CMT002", 
      "author": "Alice Johnson",
      "text": "Very informative.",
      "date": "2024-01-16T11:30:00Z",
      "likes": 3
    }
  ],
  "tags": ["nosql", "database", "tutorial"],
  "publishedAt": "2024-01-15T09:00:00Z",
  "views": 1250
}
```

#### **Referencing (Normalized)**
```javascript
// Blog post with referenced comments
{
  "_id": ObjectId("..."),
  "title": "Introduction to NoSQL",
  "content": "NoSQL databases are...",
  "authorId": ObjectId("..."),  // Reference to users collection
  "commentIds": [               // References to comments collection
    ObjectId("..."),
    ObjectId("...")
  ],
  "tags": ["nosql", "database", "tutorial"],
  "publishedAt": "2024-01-15T09:00:00Z",
  "views": 1250
}

// Separate comments collection
{
  "_id": ObjectId("..."),
  "postId": ObjectId("..."),    // Reference back to post
  "author": "John Smith",
  "text": "Great article!",
  "date": "2024-01-16T10:00:00Z",
  "likes": 5
}
```

### **When to Embed vs Reference**

**Use Embedding When:**
- Data is frequently accessed together
- Related data doesn't change often
- Document size remains reasonable (<16MB in MongoDB)
- One-to-few relationships

**Use References When:**
- Data is accessed independently
- Related data changes frequently
- Document would become too large
- Many-to-many relationships
- Data is shared across multiple documents

## Document Validation

### **Schema Validation (MongoDB)**
```javascript
// Create collection with validation rules
db.createCollection("products", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "price", "category"],
      properties: {
        name: {
          bsonType: "string",
          description: "Product name is required and must be a string"
        },
        price: {
          bsonType: "decimal",
          minimum: 0,
          description: "Price must be a positive decimal"
        },
        category: {
          bsonType: "string",
          enum: ["electronics", "clothing", "books", "sports"],
          description: "Category must be one of the allowed values"
        },
        tags: {
          bsonType: "array",
          items: {
            bsonType: "string"
          },
          description: "Tags must be an array of strings"
        }
      }
    }
  }
});

// Valid document
db.products.insertOne({
  name: "Wireless Mouse",
  price: NumberDecimal("29.99"),
  category: "electronics",
  tags: ["wireless", "computer", "accessory"]
});

// Invalid document (will be rejected)
db.products.insertOne({
  name: "Invalid Product",
  price: -10,  // Negative price not allowed
  category: "invalid_category"  // Not in enum list
});
```

## Indexing Documents

### **Basic Indexing**
```javascript
// Create single field index
db.customers.createIndex({email: 1});  // Ascending order

// Create compound index
db.customers.createIndex({city: 1, age: -1});  // city ascending, age descending

// Create index on nested field
db.customers.createIndex({"address.city": 1});

// Create index on array elements
db.products.createIndex({tags: 1});

// Text index for search
db.products.createIndex({
  name: "text",
  description: "text"
});

// Geospatial index
db.stores.createIndex({location: "2dsphere"});
```

### **Query with Indexes**
```javascript
// These queries will use indexes efficiently
db.customers.find({email: "john@example.com"});
db.customers.find({city: "New York", age: {$gte: 25}});
db.products.find({$text: {$search: "wireless mouse"}});
```

## Real-World Examples

### **E-commerce Product Catalog**
```javascript
{
  "_id": ObjectId("..."),
  "sku": "PHONE001",
  "name": "Smartphone X",
  "brand": "TechCorp",
  "category": "electronics",
  "subcategory": "smartphones",
  "description": "Latest smartphone with advanced features",
  "specifications": {
    "display": "6.1 inch OLED",
    "storage": "128 GB",
    "camera": "12 MP dual camera",
    "battery": "3000 mAh",
    "os": "Android 13"
  },
  "pricing": {
    "cost": 300.00,
    "retail": 699.99,
    "sale": 599.99,
    "currency": "USD"
  },
  "inventory": {
    "total": 100,
    "available": 75,
    "reserved": 10,
    "sold": 15
  },
  "images": [
    {"url": "/images/phone001_1.jpg", "alt": "Front view"},
    {"url": "/images/phone001_2.jpg", "alt": "Back view"}
  ],
  "variants": [
    {"color": "black", "storage": "128GB", "sku": "PHONE001-BK-128"},
    {"color": "white", "storage": "256GB", "sku": "PHONE001-WH-256"}
  ],
  "reviews": {
    "average": 4.5,
    "count": 128,
    "breakdown": {
      "5star": 75,
      "4star": 35,
      "3star": 12,
      "2star": 4,
      "1star": 2
    }
  },
  "seo": {
    "metaTitle": "TechCorp Smartphone X - Best Smartphone 2024",
    "metaDescription": "Buy TechCorp Smartphone X with advanced features...",
    "keywords": ["smartphone", "android", "techcorp"]
  },
  "status": "active",
  "createdAt": "2024-01-10T10:00:00Z",
  "updatedAt": "2024-01-20T15:30:00Z"
}
```

### **User Profile with Activity History**
```javascript
{
  "_id": ObjectId("..."),
  "userId": "USER12345",
  "profile": {
    "firstName": "Sarah",
    "lastName": "Connor",
    "email": "sarah@example.com",
    "phone": "+1-555-0199",
    "dateOfBirth": "1985-05-15",
    "avatar": "/avatars/sarah_connor.jpg"
  },
  "preferences": {
    "language": "en",
    "timezone": "America/New_York",
    "currency": "USD",
    "notifications": {
      "email": true,
      "push": true,
      "sms": false
    },
    "privacy": {
      "profileVisible": true,
      "activityVisible": false
    }
  },
  "subscription": {
    "plan": "premium",
    "status": "active",
    "startDate": "2024-01-01",
    "renewalDate": "2025-01-01",
    "autoRenew": true
  },
  "activityHistory": [
    {
      "action": "login",
      "timestamp": "2024-01-20T08:30:00Z",
      "ip": "192.168.1.100",
      "userAgent": "Mozilla/5.0...",
      "location": "New York, NY"
    },
    {
      "action": "purchase",
      "timestamp": "2024-01-19T14:15:00Z",
      "orderId": "ORD789",
      "amount": 99.99,
      "items": ["PROD001", "PROD002"]
    }
  ],
  "addresses": [
    {
      "type": "home",
      "street": "123 Elm Street",
      "city": "New York",
      "state": "NY",
      "zipCode": "10001",
      "country": "USA",
      "isDefault": true
    }
  ],
  "paymentMethods": [
    {
      "type": "credit_card",
      "last4": "4242",
      "brand": "visa",
      "expiryMonth": 12,
      "expiryYear": 2026,
      "isDefault": true
    }
  ],
  "socialLinks": {
    "twitter": "@sarahconnor",
    "linkedin": "sarah-connor-tech"
  },
  "metadata": {
    "createdAt": "2023-06-15T09:00:00Z",
    "lastLogin": "2024-01-20T08:30:00Z",
    "loginCount": 342,
    "emailVerified": true,
    "phoneVerified": true
  }
}
```

## Summary

The document model in NoSQL databases provides significant flexibility and performance advantages for many use cases. Understanding the hierarchy of Database → Collection → Document is fundamental to working effectively with NoSQL systems.

**Key Benefits:**
- **Flexible Schema**: Evolve data structure without migrations
- **Natural Modeling**: Represent complex data relationships intuitively
- **Performance**: Reduce joins through embedded data
- **Scalability**: Horizontal scaling capabilities

**Design Considerations:**
- **Embedding vs Referencing**: Choose based on access patterns
- **Document Size**: Keep documents reasonable in size
- **Query Patterns**: Design documents for how they'll be queried
- **Indexing Strategy**: Create indexes for efficient queries

**When NoSQL Documents Work Well:**
- **Rapid Development**: Changing requirements and quick iterations
- **Complex Nested Data**: Product catalogs, user profiles, content management
- **High Read Performance**: Embedded data reduces query complexity
- **Horizontal Scaling**: Distributed across multiple servers

The document model's flexibility makes it ideal for modern applications with evolving requirements, complex data structures, and the need for rapid development cycles.