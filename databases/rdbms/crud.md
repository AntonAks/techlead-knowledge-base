# CRUD Operations

## What is CRUD?

**CRUD** stands for the four basic operations that can be performed on data in persistent storage:
- **Create**: Add new data records
- **Read**: Retrieve existing data records  
- **Update**: Modify existing data records
- **Delete**: Remove data records

CRUD operations form the foundation of data manipulation in databases, APIs, and applications.

## Database CRUD Operations

### **Create (INSERT)**
Adding new records to database tables.

#### **SQL Examples**
```sql
-- Insert single record
INSERT INTO users (name, email, age) 
VALUES ('John Doe', 'john@example.com', 30);

-- Insert multiple records
INSERT INTO users (name, email, age) 
VALUES 
  ('Jane Smith', 'jane@example.com', 25),
  ('Bob Johnson', 'bob@example.com', 35);

-- Insert with auto-generated ID
INSERT INTO users (name, email) 
VALUES ('Alice Brown', 'alice@example.com');
```

#### **Best Practices**
- **Validation**: Validate data before insertion
- **Constraints**: Use database constraints (NOT NULL, UNIQUE, CHECK)
- **Transactions**: Use transactions for multi-table inserts
- **Error Handling**: Handle constraint violations gracefully

### **Read (SELECT)**
Retrieving data from database tables.

#### **SQL Examples**
```sql
-- Select all records
SELECT * FROM users;

-- Select specific columns
SELECT name, email FROM users;

-- Select with conditions
SELECT * FROM users WHERE age > 25;

-- Select with sorting
SELECT * FROM users ORDER BY name ASC;

-- Select with pagination
SELECT * FROM users LIMIT 10 OFFSET 20;

-- Select with joins
SELECT u.name, p.title 
FROM users u 
JOIN posts p ON u.id = p.user_id;
```

#### **Optimization Techniques**
- **Indexing**: Create indexes on frequently queried columns
- **Query Optimization**: Use EXPLAIN to analyze query performance
- **Pagination**: Limit result sets for large tables
- **Caching**: Cache frequently accessed data

### **Update (UPDATE)**
Modifying existing records in database tables.

#### **SQL Examples**
```sql
-- Update single record
UPDATE users 
SET email = 'newemail@example.com' 
WHERE id = 1;

-- Update multiple records
UPDATE users 
SET status = 'active' 
WHERE last_login > '2024-01-01';

-- Update with calculations
UPDATE products 
SET price = price * 1.1 
WHERE category = 'electronics';

-- Conditional update
UPDATE users 
SET login_count = login_count + 1,
    last_login = NOW()
WHERE id = 1;
```

#### **Safety Measures**
- **WHERE Clause**: Always use WHERE to avoid updating all records
- **Transactions**: Use transactions for complex updates
- **Backup**: Backup data before bulk updates
- **Testing**: Test updates on development data first

### **Delete (DELETE)**
Removing records from database tables.

#### **SQL Examples**
```sql
-- Delete single record
DELETE FROM users WHERE id = 1;

-- Delete with conditions
DELETE FROM users WHERE status = 'inactive';

-- Delete with joins (MySQL syntax)
DELETE u FROM users u
JOIN inactive_accounts i ON u.id = i.user_id;

-- Soft delete (recommended)
UPDATE users 
SET deleted_at = NOW(), status = 'deleted' 
WHERE id = 1;
```

#### **Deletion Strategies**
- **Soft Delete**: Mark records as deleted instead of removing
- **Cascading**: Define foreign key cascade behavior
- **Archival**: Move old data to archive tables
- **Audit**: Log all deletion operations

## API CRUD Operations

### **RESTful API Mapping**
CRUD operations map to HTTP methods in REST APIs:

| CRUD Operation | HTTP Method | URL Pattern | Purpose |
|----------------|-------------|-------------|---------|
| **Create** | POST | `/users` | Create new user |
| **Read** | GET | `/users` or `/users/123` | Get users or specific user |
| **Update** | PUT/PATCH | `/users/123` | Update user |
| **Delete** | DELETE | `/users/123` | Delete user |

### **API Examples**

#### **Create (POST)**
```http
POST /api/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

Response: 201 Created
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### **Read (GET)**
```http
# Get all users
GET /api/users
Response: 200 OK
{
  "users": [...],
  "total": 100,
  "page": 1
}

# Get specific user
GET /api/users/123
Response: 200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

#### **Update (PUT/PATCH)**
```http
# Full update (PUT)
PUT /api/users/123
{
  "name": "John Smith",
  "email": "john.smith@example.com",
  "age": 31
}

# Partial update (PATCH)
PATCH /api/users/123
{
  "email": "newemail@example.com"
}

Response: 200 OK
```

#### **Delete (DELETE)**
```http
DELETE /api/users/123
Response: 204 No Content
```

## Application-Level CRUD

### **Object-Relational Mapping (ORM)**
ORMs provide abstraction layer over database CRUD operations.

#### **Examples in Different Languages**

**Python (Django ORM)**
```python
from myapp.models import User

# Create
user = User.objects.create(
    name="John Doe",
    email="john@example.com",
    age=30
)

# Read
users = User.objects.all()
user = User.objects.get(id=123)
filtered_users = User.objects.filter(age__gt=25)

# Update
user = User.objects.get(id=123)
user.email = "newemail@example.com"
user.save()

# Delete
user = User.objects.get(id=123)
user.delete()
```

**Java (JPA/Hibernate)**
```java
// Create
User user = new User("John Doe", "john@example.com", 30);
entityManager.persist(user);

// Read
User user = entityManager.find(User.class, 123L);
List<User> users = entityManager.createQuery(
    "SELECT u FROM User u WHERE u.age > 25", User.class
).getResultList();

// Update
User user = entityManager.find(User.class, 123L);
user.setEmail("newemail@example.com");
entityManager.merge(user);

// Delete
User user = entityManager.find(User.class, 123L);
entityManager.remove(user);
```

### **Repository Pattern**
Encapsulate data access logic in repository classes.

```python
class UserRepository:
    def create(self, user_data):
        return User.objects.create(**user_data)
    
    def get_by_id(self, user_id):
        return User.objects.get(id=user_id)
    
    def get_all(self, filters=None):
        queryset = User.objects.all()
        if filters:
            queryset = queryset.filter(**filters)
        return queryset
    
    def update(self, user_id, user_data):
        User.objects.filter(id=user_id).update(**user_data)
    
    def delete(self, user_id):
        User.objects.filter(id=user_id).delete()
```

## CRUD Design Patterns

### **Active Record Pattern**
Domain objects contain both data and behavior.

```python
class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def activate(self):
        self.status = 'active'
        self.save()
    
    def deactivate(self):
        self.status = 'inactive'
        self.save()
    
    @classmethod
    def find_by_email(cls, email):
        return cls.objects.get(email=email)
```

### **Data Access Object (DAO) Pattern**
Separate data access logic from business logic.

```java
public interface UserDAO {
    void create(User user);
    User findById(Long id);
    List<User> findAll();
    void update(User user);
    void delete(Long id);
}

public class UserDAOImpl implements UserDAO {
    // Implementation using JDBC, JPA, etc.
}
```

### **Unit of Work Pattern**
Maintain list of objects affected by business transaction.

```python
class UnitOfWork:
    def __init__(self):
        self._new_objects = []
        self._dirty_objects = []
        self._removed_objects = []
    
    def register_new(self, obj):
        self._new_objects.append(obj)
    
    def register_dirty(self, obj):
        self._dirty_objects.append(obj)
    
    def register_removed(self, obj):
        self._removed_objects.append(obj)
    
    def commit(self):
        # Insert new objects
        # Update dirty objects
        # Delete removed objects
        pass
```

## Advanced CRUD Patterns

### **Soft Delete**
Mark records as deleted instead of physical removal.

```sql
-- Add deleted_at column
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;

-- Soft delete
UPDATE users SET deleted_at = NOW() WHERE id = 123;

-- Query active records
SELECT * FROM users WHERE deleted_at IS NULL;

-- Restore deleted record
UPDATE users SET deleted_at = NULL WHERE id = 123;
```

### **Audit Trail**
Track all changes to data records.

```sql
-- Create audit table
CREATE TABLE user_audit (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT,
    operation VARCHAR(10), -- INSERT, UPDATE, DELETE
    old_values JSON,
    new_values JSON,
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger for audit trail
CREATE TRIGGER user_audit_trigger
AFTER UPDATE ON users
FOR EACH ROW
INSERT INTO user_audit (user_id, operation, old_values, new_values)
VALUES (NEW.id, 'UPDATE', 
        JSON_OBJECT('name', OLD.name, 'email', OLD.email),
        JSON_OBJECT('name', NEW.name, 'email', NEW.email));
```

### **Optimistic Locking**
Handle concurrent updates using version numbers.

```sql
-- Add version column
ALTER TABLE users ADD COLUMN version INT DEFAULT 0;

-- Update with version check
UPDATE users 
SET name = 'New Name', version = version + 1
WHERE id = 123 AND version = 5;

-- Check affected rows to detect conflicts
```

### **Batch Operations**
Efficient handling of bulk CRUD operations.

```sql
-- Batch insert
INSERT INTO users (name, email) VALUES
  ('User 1', 'user1@example.com'),
  ('User 2', 'user2@example.com'),
  ('User 3', 'user3@example.com');

-- Batch update
UPDATE users 
SET status = 'verified'
WHERE id IN (1, 2, 3, 4, 5);

-- Batch delete
DELETE FROM users WHERE status = 'inactive';
```

## CRUD Security Considerations

### **Input Validation**
- **SQL Injection**: Use parameterized queries
- **Data Validation**: Validate all input data
- **Sanitization**: Clean input data before processing
- **Type Safety**: Use strongly typed parameters

### **Authorization**
- **Access Control**: Check user permissions for each operation
- **Row-Level Security**: Filter data based on user context
- **Field-Level Security**: Restrict access to sensitive fields
- **Operation-Level**: Different permissions for CRUD operations

### **Data Protection**
- **Encryption**: Encrypt sensitive data at rest
- **Masking**: Mask sensitive data in logs and responses
- **Anonymization**: Remove personally identifiable information
- **Retention**: Implement data retention policies

## Performance Optimization

### **Read Optimization**
- **Indexing**: Create appropriate indexes
- **Query Optimization**: Optimize SQL queries
- **Caching**: Cache frequently accessed data
- **Pagination**: Limit result sets
- **Connection Pooling**: Reuse database connections

### **Write Optimization**
- **Batch Operations**: Group multiple operations
- **Transactions**: Use appropriate transaction isolation
- **Async Processing**: Use queues for non-critical operations
- **Prepared Statements**: Reuse query execution plans

### **Monitoring**
- **Query Performance**: Monitor slow queries
- **Database Metrics**: Track connection usage, query times
- **Application Metrics**: Monitor CRUD operation success rates
- **Error Tracking**: Log and alert on CRUD failures

## Testing CRUD Operations

### **Unit Tests**
```python
def test_create_user():
    user_data = {"name": "Test User", "email": "test@example.com"}
    user = User.objects.create(**user_data)
    assert user.id is not None
    assert user.name == "Test User"

def test_read_user():
    user = User.objects.create(name="Test", email="test@example.com")
    retrieved_user = User.objects.get(id=user.id)
    assert retrieved_user.name == "Test"

def test_update_user():
    user = User.objects.create(name="Test", email="test@example.com")
    user.name = "Updated Name"
    user.save()
    assert User.objects.get(id=user.id).name == "Updated Name"

def test_delete_user():
    user = User.objects.create(name="Test", email="test@example.com")
    user_id = user.id
    user.delete()
    with pytest.raises(User.DoesNotExist):
        User.objects.get(id=user_id)
```

### **Integration Tests**
```python
def test_api_crud_operations():
    # Test Create
    response = client.post('/api/users', json={
        "name": "Test User",
        "email": "test@example.com"
    })
    assert response.status_code == 201
    user_id = response.json()['id']
    
    # Test Read
    response = client.get(f'/api/users/{user_id}')
    assert response.status_code == 200
    assert response.json()['name'] == "Test User"
    
    # Test Update
    response = client.patch(f'/api/users/{user_id}', json={
        "name": "Updated User"
    })
    assert response.status_code == 200
    
    # Test Delete
    response = client.delete(f'/api/users/{user_id}')
    assert response.status_code == 204
```

## Error Handling

### **Database Errors**
- **Constraint Violations**: Handle unique, foreign key constraints
- **Connection Errors**: Retry logic for temporary failures
- **Timeout Errors**: Handle long-running operations
- **Deadlocks**: Retry strategy for concurrent operations

### **Application Errors**
- **Validation Errors**: Return meaningful error messages
- **Not Found**: Handle missing records gracefully
- **Conflict Errors**: Handle concurrent modification attempts
- **Permission Errors**: Clear authorization failure messages

### **API Error Responses**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid"
      }
    ]
  }
}
```

## Summary

CRUD operations are fundamental to data management in applications and databases. Implementing them effectively requires consideration of performance, security, error handling, and maintainability.

**Key Principles:**
- **Consistency**: Use consistent patterns across all CRUD operations
- **Security**: Validate input, authorize access, protect sensitive data
- **Performance**: Optimize queries, use appropriate indexing, implement caching
- **Reliability**: Handle errors gracefully, use transactions appropriately
- **Maintainability**: Use established patterns, write comprehensive tests

**Best Practices:**
- **Database Design**: Proper normalization, constraints, and indexing
- **API Design**: RESTful endpoints with appropriate HTTP methods
- **Code Organization**: Repository pattern, separation of concerns
- **Testing**: Comprehensive unit and integration tests
- **Monitoring**: Track performance and error rates

**Modern Considerations:**
- **Microservices**: CRUD operations across service boundaries
- **Event Sourcing**: Alternative to traditional CRUD for some use cases
- **CQRS**: Separate read and write models for complex scenarios
- **Real-time**: WebSocket or Server-Sent Events for live data updates

CRUD operations form the foundation of data-driven applications. Understanding how to implement them efficiently, securely, and reliably is essential for building robust software systems.