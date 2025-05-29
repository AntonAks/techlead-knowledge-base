# HTTP Request Methods

## Overview

HTTP request methods (also called HTTP verbs) indicate the desired action to be performed on a resource. Each method has specific semantics and properties that determine how it should be used in web applications and APIs.

## Primary HTTP Methods

### **GET**

**Purpose**: Retrieve data from the server

**Characteristics**:
- **Safe**: Should not modify server state
- **Idempotent**: Multiple identical requests have the same effect
- **Cacheable**: Responses can be cached
- **Parameters**: Sent in URL query string

**Examples**:
```http
# Get user profile
GET /api/users/123 HTTP/1.1
Host: api.example.com

# Search with parameters
GET /api/products?category=electronics&price_max=500 HTTP/1.1
Host: api.example.com

# Get list with pagination
GET /api/articles?page=2&limit=20&sort=created_date HTTP/1.1
Host: api.example.com
```

**Use Cases**:
- Fetching web pages
- Retrieving API data
- Loading images, CSS, JavaScript files
- Search operations
- Getting resource lists

### **POST**

**Purpose**: Submit data to create new resources or trigger operations

**Characteristics**:
- **Not Safe**: May modify server state
- **Not Idempotent**: Multiple calls may have different effects
- **Not Cacheable**: Responses typically not cached
- **Parameters**: Sent in request body

**Examples**:
```http
# Create new user
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securePassword"
}

# Upload file
POST /api/files/upload HTTP/1.1
Host: api.example.com
Content-Type: multipart/form-data

[file data]

# Process payment
POST /api/payments HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "amount": 99.99,
  "currency": "USD",
  "card_token": "tok_1234567890"
}
```

**Use Cases**:
- Creating new resources
- Submitting forms
- File uploads
- Authentication (login)
- Triggering server-side processes

### **PUT**

**Purpose**: Update or create resources (complete replacement)

**Characteristics**:
- **Not Safe**: Modifies server state
- **Idempotent**: Multiple identical calls have same effect
- **Not Cacheable**: Responses typically not cached
- **Complete Replacement**: Entire resource is replaced

**Examples**:
```http
# Update user (complete replacement)
PUT /api/users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "id": 123,
  "name": "John Smith",
  "email": "john.smith@example.com",
  "role": "admin",
  "active": true
}

# Create or update configuration
PUT /api/config/app-settings HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "theme": "dark",
  "notifications": true,
  "language": "en"
}
```

**Use Cases**:
- Complete resource updates
- Replacing entire documents
- Creating resources with known IDs
- Configuration updates

### **PATCH**

**Purpose**: Partial updates to resources

**Characteristics**:
- **Not Safe**: Modifies server state
- **Not necessarily Idempotent**: Depends on implementation
- **Partial Update**: Only specified fields are modified

**Examples**:
```http
# Update only email address
PATCH /api/users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "email": "newemail@example.com"
}

# JSON Patch format
PATCH /api/articles/456 HTTP/1.1
Host: api.example.com
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/title", "value": "New Article Title" },
  { "op": "add", "path": "/tags/-", "value": "javascript" }
]

# Increment counter
PATCH /api/counters/page-views HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "operation": "increment",
  "value": 1
}
```

**Use Cases**:
- Updating specific fields
- Incremental changes
- Status updates
- Adding/removing items from collections

### **DELETE**

**Purpose**: Remove resources from the server

**Characteristics**:
- **Not Safe**: Modifies server state
- **Idempotent**: Multiple deletions have same effect
- **No Body**: Typically no request body

**Examples**:
```http
# Delete specific user
DELETE /api/users/123 HTTP/1.1
Host: api.example.com

# Delete user's session (logout)
DELETE /api/sessions/current HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Delete file
DELETE /api/files/document.pdf HTTP/1.1
Host: api.example.com
```

**Use Cases**:
- Removing resources
- User logout
- Clearing caches
- Canceling operations

### **HEAD**

**Purpose**: Get response headers without body (like GET but headers only)

**Characteristics**:
- **Safe**: Should not modify server state
- **Idempotent**: Multiple calls have same effect
- **No Body**: Response has no message body

**Examples**:
```http
# Check if resource exists
HEAD /api/users/123 HTTP/1.1
Host: api.example.com

# Get file metadata
HEAD /api/files/document.pdf HTTP/1.1
Host: api.example.com

# Response example
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Length: 1048576
Last-Modified: Mon, 15 Jan 2024 10:30:00 GMT
ETag: "abc123"
```

**Use Cases**:
- Checking resource existence
- Getting metadata without downloading content
- Validating caches
- Checking file sizes before download

### **OPTIONS**

**Purpose**: Describe communication options for target resource

**Characteristics**:
- **Safe**: Should not modify server state
- **Used for CORS**: Cross-Origin Resource Sharing preflight requests
- **Discovery**: Find out what methods are allowed

**Examples**:
```http
# CORS preflight request
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Origin: https://webapp.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization

# Response
HTTP/1.1 200 OK
Allow: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Origin: https://webapp.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

**Use Cases**:
- CORS preflight requests
- API discovery
- Checking allowed methods
- Server capability negotiation

## Less Common HTTP Methods

### **CONNECT**
- Establishes tunnel to server (used by proxies)
- Primarily for HTTPS through HTTP proxies

### **TRACE**
- Performs message loop-back test
- Rarely used due to security concerns

## Method Properties Comparison

| Method | Safe | Idempotent | Cacheable | Body Allowed |
|--------|------|------------|-----------|--------------|
| GET | ✅ | ✅ | ✅ | ❌ |
| POST | ❌ | ❌ | ❌ | ✅ |
| PUT | ❌ | ✅ | ❌ | ✅ |
| PATCH | ❌ | ❌* | ❌ | ✅ |
| DELETE | ❌ | ✅ | ❌ | ❌* |
| HEAD | ✅ | ✅ | ✅ | ❌ |
| OPTIONS | ✅ | ✅ | ❌ | ❌ |

*May vary depending on implementation

## RESTful API Method Usage

### **Resource Collections**
```http
GET /api/users           # Get list of users
POST /api/users          # Create new user
PUT /api/users           # Replace entire collection (rare)
DELETE /api/users        # Delete all users (dangerous)
```

### **Individual Resources**
```http
GET /api/users/123       # Get specific user
PUT /api/users/123       # Update/replace user
PATCH /api/users/123     # Partially update user
DELETE /api/users/123    # Delete user
HEAD /api/users/123      # Check if user exists
```

### **Nested Resources**
```http
GET /api/users/123/posts       # Get user's posts
POST /api/users/123/posts      # Create post for user
GET /api/users/123/posts/456   # Get specific post
PUT /api/users/123/posts/456   # Update specific post
DELETE /api/users/123/posts/456 # Delete specific post
```

## Method Selection Guidelines

### **Use GET when**:
- Retrieving data
- Operation is safe and idempotent
- Result can be cached
- No sensitive data in URL

### **Use POST when**:
- Creating new resources
- Submitting forms
- Uploading files
- Operations that aren't idempotent
- Sensitive data that shouldn't be in URL

### **Use PUT when**:
- Replacing entire resources
- Operations need to be idempotent
- Client knows the complete resource structure

### **Use PATCH when**:
- Making partial updates
- Only specific fields need modification
- Want to minimize data transfer

### **Use DELETE when**:
- Removing resources
- Operations should be idempotent
- Clear intent to remove data

## Security Considerations

### **GET Requests**
- Don't put sensitive data in URLs
- URLs may be logged by servers/proxies
- Can be cached by browsers/intermediaries

```http
# Bad - sensitive data in URL
GET /api/users?password=secret123

# Good - use POST for sensitive data
POST /api/auth/login
Content-Type: application/json

{"username": "user", "password": "secret123"}
```

### **Method Override**
Some clients/proxies only support GET/POST:

```http
POST /api/users/123 HTTP/1.1
X-HTTP-Method-Override: DELETE
```

### **CSRF Protection**
- GET requests should be safe
- Use CSRF tokens for state-changing operations
- Validate Origin/Referer headers

## Error Handling by Method

### **GET Errors**
```http
# Resource not found
HTTP/1.1 404 Not Found

# Server error
HTTP/1.1 500 Internal Server Error

# Unauthorized
HTTP/1.1 401 Unauthorized
```

### **POST Errors**
```http
# Validation error
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Validation failed",
  "details": [
    {"field": "email", "message": "Invalid email format"}
  ]
}

# Conflict (duplicate resource)
HTTP/1.1 409 Conflict
```

### **PUT/PATCH Errors**
```http
# Resource not found
HTTP/1.1 404 Not Found

# Version conflict
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "error": "Resource has been modified",
  "current_version": 5,
  "provided_version": 3
}
```

## Testing HTTP Methods

### **Using curl**
```bash
# GET request
curl -X GET https://api.example.com/users/123

# POST with JSON data
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# PUT request
curl -X PUT https://api.example.com/users/123 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Smith","email":"john.smith@example.com"}'

# PATCH request
curl -X PATCH https://api.example.com/users/123 \
  -H "Content-Type: application/json" \
  -d '{"email":"newemail@example.com"}'

# DELETE request
curl -X DELETE https://api.example.com/users/123

# HEAD request
curl -I https://api.example.com/users/123

# OPTIONS request
curl -X OPTIONS https://api.example.com/users
```

## Best Practices

### **Method Selection**
- Choose methods based on their semantic meaning
- Follow REST conventions consistently
- Use appropriate status codes for responses

### **Idempotency**
- Design PUT and DELETE operations to be idempotent
- Consider idempotency keys for POST operations
- Handle duplicate requests gracefully

### **Safety**
- Ensure GET and HEAD operations don't modify state
- Use POST for non-idempotent operations
- Implement proper error handling

### **Performance**
- Use appropriate caching for GET requests
- Consider conditional requests (If-Modified-Since, ETag)
- Implement compression for large responses

## Summary

HTTP request methods provide semantic meaning to different types of operations in web applications. Understanding when and how to use each method is crucial for building well-designed APIs and web services. The choice of method affects caching behavior, security considerations, and client expectations, making it important to select the appropriate method for each use case.