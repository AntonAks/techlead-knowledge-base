# HTTP Status Codes

## Overview

HTTP status codes are three-digit numbers returned by servers to indicate the result of a client's request. They provide a standardized way to communicate the outcome of HTTP operations and help clients understand how to handle responses.

## Status Code Categories

- **1xx**: Informational responses
- **2xx**: Success responses  
- **3xx**: Redirection responses
- **4xx**: Client error responses
- **5xx**: Server error responses

## 1xx Informational Responses

These codes indicate that the request has been received and is being processed.

### **100 Continue**
- Server has received request headers
- Client can continue sending request body
- Used with `Expect: 100-continue` header

```http
# Client sends headers first
POST /api/upload HTTP/1.1
Expect: 100-continue
Content-Length: 1048576

# Server responds
HTTP/1.1 100 Continue

# Client then sends body
[large file data]
```

### **101 Switching Protocols**
- Server is switching to different protocol
- Common with WebSocket upgrades

```http
# Client request
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade

# Server response
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

### **102 Processing** (WebDAV)
- Server is processing request but no response yet
- Prevents client timeouts on long operations

## 2xx Success Responses

These codes indicate that the request was successfully received, understood, and accepted.

### **200 OK**
- Request succeeded
- Most common success status
- Response body contains requested data

```http
GET /api/users/123 HTTP/1.1

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

### **201 Created**
- New resource successfully created
- Should include `Location` header with new resource URI

```http
POST /api/users HTTP/1.1
Content-Type: application/json
{"name": "John Doe", "email": "john@example.com"}

HTTP/1.1 201 Created
Location: /api/users/124
Content-Type: application/json

{
  "id": 124,
  "name": "John Doe",
  "email": "john@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### **202 Accepted**
- Request accepted for processing
- Processing not completed yet
- Used for asynchronous operations

```http
POST /api/reports/generate HTTP/1.1

HTTP/1.1 202 Accepted
Content-Type: application/json

{
  "message": "Report generation started",
  "job_id": "job_12345",
  "status_url": "/api/jobs/job_12345"
}
```

### **204 No Content**
- Request successful but no content to return
- Common for DELETE operations
- Headers may still contain useful information

```http
DELETE /api/users/123 HTTP/1.1

HTTP/1.1 204 No Content
```

### **206 Partial Content**
- Server delivering partial resource
- Used with Range requests (file downloads, streaming)

```http
GET /api/files/video.mp4 HTTP/1.1
Range: bytes=1000-2000

HTTP/1.1 206 Partial Content
Content-Range: bytes 1000-2000/5000
Content-Length: 1001

[partial file data]
```

## 3xx Redirection Responses

These codes indicate that further action needs to be taken to complete the request.

### **301 Moved Permanently**
- Resource permanently moved to new URL
- Update bookmarks and links
- Search engines update their indexes

```http
GET /old-page HTTP/1.1

HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-page
```

### **302 Found** (Temporary Redirect)
- Resource temporarily at different URL
- Don't update bookmarks
- Original URL may work again later

```http
GET /api/users/profile HTTP/1.1

HTTP/1.1 302 Found
Location: /api/users/123
```

### **304 Not Modified**
- Resource hasn't changed since last request
- Used with conditional requests (`If-Modified-Since`, `If-None-Match`)
- Saves bandwidth by not sending unchanged content

```http
GET /api/users/123 HTTP/1.1
If-None-Match: "abc123"

HTTP/1.1 304 Not Modified
ETag: "abc123"
```

### **307 Temporary Redirect**
- Like 302 but method and body must not change
- More specific than 302

### **308 Permanent Redirect**
- Like 301 but method and body must not change
- More specific than 301

## 4xx Client Error Responses

These codes indicate that the client made an error in the request.

### **400 Bad Request**
- Request syntax is invalid
- Server cannot understand the request
- Generic client error

```http
POST /api/users HTTP/1.1
Content-Type: application/json
{"name": , "email": "invalid-email"}

HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid JSON syntax",
  "message": "Unexpected token at position 10"
}
```

### **401 Unauthorized**
- Authentication required or failed
- Must include `WWW-Authenticate` header
- Actually means "unauthenticated"

```http
GET /api/admin/users HTTP/1.1

HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
Content-Type: application/json

{
  "error": "Authentication required",
  "message": "Please provide a valid access token"
}
```

### **403 Forbidden**
- Server understood request but refuses to authorize
- Authentication won't help
- User lacks necessary permissions

```http
DELETE /api/admin/users/1 HTTP/1.1
Authorization: Bearer user_token

HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "error": "Insufficient permissions",
  "message": "Admin privileges required"
}
```

### **404 Not Found**
- Requested resource doesn't exist
- May be temporary or permanent
- Don't reveal whether resource ever existed

```http
GET /api/users/999 HTTP/1.1

HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "error": "User not found",
  "message": "No user exists with ID 999"
}
```

### **405 Method Not Allowed**
- HTTP method not supported for resource
- Must include `Allow` header with supported methods

```http
POST /api/users/123 HTTP/1.1

HTTP/1.1 405 Method Not Allowed
Allow: GET, PUT, DELETE
Content-Type: application/json

{
  "error": "Method not allowed",
  "message": "POST not supported for this resource"
}
```

### **406 Not Acceptable**
- Server cannot produce content matching `Accept` headers

```http
GET /api/users/123 HTTP/1.1
Accept: application/xml

HTTP/1.1 406 Not Acceptable
Content-Type: application/json

{
  "error": "Not acceptable",
  "supported_formats": ["application/json"]
}
```

### **409 Conflict**
- Request conflicts with current resource state
- Common with version conflicts or duplicate resources

```http
POST /api/users HTTP/1.1
{"email": "existing@example.com"}

HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "error": "Email already exists",
  "message": "User with this email already registered"
}
```

### **410 Gone**
- Resource permanently deleted
- More specific than 404
- Won't be available again

```http
GET /api/users/deleted-user HTTP/1.1

HTTP/1.1 410 Gone
Content-Type: application/json

{
  "error": "Resource gone",
  "message": "User account was permanently deleted"
}
```

### **411 Length Required**
- Request must include `Content-Length` header

### **412 Precondition Failed**
- Precondition in request headers evaluated to false

```http
PUT /api/users/123 HTTP/1.1
If-Match: "old-etag"

HTTP/1.1 412 Precondition Failed
ETag: "current-etag"
```

### **413 Payload Too Large**
- Request body exceeds server limits

```http
POST /api/upload HTTP/1.1
Content-Length: 10485760
[large file data]

HTTP/1.1 413 Payload Too Large
Content-Type: application/json

{
  "error": "File too large",
  "max_size": "5MB",
  "received_size": "10MB"
}
```

### **415 Unsupported Media Type**
- Server doesn't support request content type

```http
POST /api/users HTTP/1.1
Content-Type: application/xml

HTTP/1.1 415 Unsupported Media Type
Content-Type: application/json

{
  "error": "Unsupported media type",
  "supported_types": ["application/json"]
}
```

### **422 Unprocessable Entity**
- Request syntax correct but semantically invalid
- Common with validation errors

```http
POST /api/users HTTP/1.1
{"name": "", "email": "invalid-email", "age": -5}

HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "error": "Validation failed",
  "details": [
    {"field": "name", "message": "Name is required"},
    {"field": "email", "message": "Invalid email format"},
    {"field": "age", "message": "Age must be positive"}
  ]
}
```

### **429 Too Many Requests**
- Rate limiting triggered
- Should include `Retry-After` header

```http
GET /api/users HTTP/1.1

HTTP/1.1 429 Too Many Requests
Retry-After: 60
Content-Type: application/json

{
  "error": "Rate limit exceeded",
  "message": "Try again in 60 seconds",
  "limit": 100,
  "window": "1 hour"
}
```

## 5xx Server Error Responses

These codes indicate that the server failed to fulfill a valid request.

### **500 Internal Server Error**
- Generic server error
- Server encountered unexpected condition
- Don't expose internal details to clients

```http
GET /api/users HTTP/1.1

HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "Internal server error",
  "message": "An unexpected error occurred",
  "request_id": "req_12345"
}
```

### **501 Not Implemented**
- Server doesn't support the functionality
- Method not recognized or not supported

```http
PATCH /api/users/123 HTTP/1.1

HTTP/1.1 501 Not Implemented
Content-Type: application/json

{
  "error": "Not implemented",
  "message": "PATCH method not yet supported"
}
```

### **502 Bad Gateway**
- Server acting as gateway received invalid response
- Common with reverse proxies and load balancers

```http
HTTP/1.1 502 Bad Gateway
Content-Type: application/json

{
  "error": "Bad gateway",
  "message": "Upstream server returned invalid response"
}
```

### **503 Service Unavailable**
- Server temporarily unavailable
- Should include `Retry-After` header
- Planned maintenance or overload

```http
HTTP/1.1 503 Service Unavailable
Retry-After: 300
Content-Type: application/json

{
  "error": "Service unavailable",
  "message": "Server under maintenance",
  "estimated_downtime": "5 minutes"
}
```

### **504 Gateway Timeout**
- Gateway didn't receive timely response
- Upstream server too slow

```http
HTTP/1.1 504 Gateway Timeout
Content-Type: application/json

{
  "error": "Gateway timeout",
  "message": "Upstream server response timeout"
}
```

## Status Code Selection Guide

### **Successful Operations**
- **200**: Standard successful GET, PUT, PATCH
- **201**: Resource created (POST)
- **202**: Asynchronous processing started
- **204**: Successful DELETE or update with no content

### **Client Errors**
- **400**: Invalid request syntax or data
- **401**: Authentication required
- **403**: Valid request but access denied
- **404**: Resource not found
- **409**: Conflict with current state
- **422**: Valid syntax but semantic errors
- **429**: Rate limiting

### **Server Errors**
- **500**: Generic server error
- **502**: Gateway/proxy error
- **503**: Service temporarily unavailable
- **504**: Gateway timeout

## API Error Response Best Practices

### **Consistent Error Format**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Email format is invalid"
      }
    ],
    "request_id": "req_12345",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### **Error Code Mapping**
```javascript
const ERROR_CODES = {
  VALIDATION_ERROR: 422,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  RATE_LIMITED: 429,
  SERVER_ERROR: 500
};
```

### **Helpful Error Messages**
```json
// Good - Specific and actionable
{
  "error": "Email already exists",
  "message": "A user with email 'john@example.com' already exists",
  "suggestion": "Try logging in or use password reset"
}

// Bad - Vague and unhelpful
{
  "error": "Error",
  "message": "Something went wrong"
}
```

## Monitoring and Logging

### **Status Code Metrics**
- Track distribution of status codes
- Monitor error rates (4xx/5xx)
- Alert on unusual patterns
- Track response times by status code

### **Common Monitoring Queries**
```
# Error rate
sum(rate(http_requests_total{status=~"[45].."}[5m])) / sum(rate(http_requests_total[5m]))

# 5xx errors
sum(rate(http_requests_total{status=~"5.."}[5m]))

# Most common errors
topk(10, sum by (status) (rate(http_requests_total{status=~"[45].."}[1h])))
```

## Testing Status Codes

### **Unit Tests**
```javascript
// Test successful creation
expect(response.status).toBe(201);
expect(response.headers.location).toBeDefined();

// Test validation error
expect(response.status).toBe(422);
expect(response.body.error.details).toHaveLength(2);

// Test not found
expect(response.status).toBe(404);
expect(response.body.error.message).toContain('not found');
```

### **Integration Tests**
```bash
# Test with curl
curl -X POST https://api.example.com/users \
  -d '{"invalid": "data"}' \
  -w "%{http_code}" \
  -s -o /dev/null
# Should return 422
```

## Summary

HTTP status codes provide essential communication between clients and servers about request outcomes. Proper use of status codes helps create predictable, debuggable APIs that clients can handle appropriately. Choose status codes based on the actual situation rather than convenience, and always provide helpful error messages alongside appropriate status codes for the best developer experience.