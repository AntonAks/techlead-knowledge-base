# HTTP Overview

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is an application-layer protocol used for transmitting data over the web. It defines how messages are formatted and transmitted between web browsers (clients) and web servers, forming the foundation of data communication on the World Wide Web.

## Key Characteristics

- **Stateless**: Each request is independent; server doesn't retain information between requests
- **Text-based**: Human-readable protocol
- **Request-Response Model**: Client sends requests, server sends responses
- **Connectionless**: Connection is closed after each transaction (in HTTP/1.0)
- **Platform Independent**: Works across different operating systems and devices

## HTTP Architecture

```
Client (Browser)  ←──── HTTP Request  ────→  Server
                 ←──── HTTP Response ────→
```

### Components:
- **Client**: Web browser, mobile app, or any HTTP client
- **Server**: Web server hosting resources
- **Resources**: Web pages, images, APIs, files
- **Proxy**: Intermediate servers (caches, load balancers)

## HTTP Message Structure

### HTTP Request Format
```
Method URI HTTP-Version
Header-Name: Header-Value
Header-Name: Header-Value
[blank line]
Message Body (optional)
```

### Example HTTP Request
```http
GET /api/users/123 HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

### HTTP Response Format
```
HTTP-Version Status-Code Reason-Phrase
Header-Name: Header-Value
Header-Name: Header-Value
[blank line]
Message Body (optional)
```

### Example HTTP Response
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 156
Cache-Control: max-age=3600
Date: Mon, 15 Jan 2024 10:30:00 GMT

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

## HTTP Methods (Verbs)

### **GET**
- Retrieve data from the server
- Should not modify server state
- Can be cached
- Parameters in URL query string

```http
GET /api/users?page=1&limit=10 HTTP/1.1
```

### **POST**
- Submit data to create new resources
- Data in request body
- Not idempotent (multiple calls may have different effects)

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{"name": "John Doe", "email": "john@example.com"}
```

### **PUT**
- Update or create resources
- Idempotent (multiple identical calls have same effect)
- Entire resource replacement

```http
PUT /api/users/123 HTTP/1.1
Content-Type: application/json

{"id": 123, "name": "John Smith", "email": "john.smith@example.com"}
```

### **DELETE**
- Remove resources from server
- Idempotent

```http
DELETE /api/users/123 HTTP/1.1
```

### **PATCH**
- Partial resource updates
- Not necessarily idempotent

```http
PATCH /api/users/123 HTTP/1.1
Content-Type: application/json

{"email": "newemail@example.com"}
```

### **HEAD**
- Like GET but returns only headers
- Useful for checking resource existence or metadata

```http
HEAD /api/users/123 HTTP/1.1
```

### **OPTIONS**
- Describes communication options for target resource
- Used in CORS preflight requests

```http
OPTIONS /api/users HTTP/1.1
```

## HTTP Status Codes

### **1xx Informational**
- `100 Continue`: Continue with request
- `101 Switching Protocols`: Protocol upgrade

### **2xx Success**
- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `204 No Content`: Success but no content to return

### **3xx Redirection**
- `301 Moved Permanently`: Resource permanently moved
- `302 Found`: Temporary redirect
- `304 Not Modified`: Resource not changed (caching)

### **4xx Client Error**
- `400 Bad Request`: Invalid request syntax
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Access denied
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded

### **5xx Server Error**
- `500 Internal Server Error`: Generic server error
- `502 Bad Gateway`: Invalid response from upstream server
- `503 Service Unavailable`: Server temporarily unavailable

## HTTP Headers

### **Request Headers**
```http
Host: api.example.com                    # Target server
User-Agent: Mozilla/5.0...               # Client information
Accept: application/json                 # Accepted response formats
Authorization: Bearer token...           # Authentication
Content-Type: application/json          # Request body format
Content-Length: 156                     # Body size in bytes
```

### **Response Headers**
```http
Content-Type: application/json          # Response body format
Content-Length: 256                     # Response size
Cache-Control: max-age=3600            # Caching directives
Set-Cookie: sessionId=abc123           # Set client cookies
Location: /api/users/124               # Resource location (redirects)
```

### **Common Headers**
- **Date**: Message timestamp
- **Connection**: Connection management (`keep-alive`, `close`)
- **Transfer-Encoding**: How body is encoded (`chunked`)
- **Content-Encoding**: Compression method (`gzip`, `deflate`)

## HTTP Cookies

Cookies provide state management in stateless HTTP:

### Setting Cookies
```http
HTTP/1.1 200 OK
Set-Cookie: sessionId=abc123; Path=/; HttpOnly; Secure
Set-Cookie: preferences=theme=dark; Max-Age=3600
```

### Sending Cookies
```http
GET /api/profile HTTP/1.1
Cookie: sessionId=abc123; preferences=theme=dark
```

## HTTP Caching

### Cache-Control Directives
```http
Cache-Control: max-age=3600              # Cache for 1 hour
Cache-Control: no-cache                  # Revalidate before use
Cache-Control: no-store                  # Don't cache at all
Cache-Control: private                   # Cache only in browser
Cache-Control: public                    # Cache anywhere
```

### ETag (Entity Tag)
```http
# Response
ETag: "abc123"

# Conditional request
If-None-Match: "abc123"

# Response if not modified
HTTP/1.1 304 Not Modified
```

## Content Negotiation

Clients can specify preferred content types:

```http
Accept: application/json, application/xml; q=0.8, text/plain; q=0.5
Accept-Language: en-US, en; q=0.9, es; q=0.8
Accept-Encoding: gzip, deflate, br
```

Server responds with appropriate format:
```http
Content-Type: application/json
Content-Language: en-US
Content-Encoding: gzip
```

## HTTP Authentication

### Basic Authentication
```http
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

### Bearer Token
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key
```http
X-API-Key: your-api-key-here
```

## HTTPS (HTTP Secure)

HTTPS is HTTP over TLS/SSL encryption:

- **Encryption**: Data encrypted during transmission
- **Authentication**: Server identity verification
- **Integrity**: Data tampering detection
- **Default Port**: 443 (vs HTTP's 80)

## HTTP Connection Management

### HTTP/1.0
- New connection for each request
- Connection closed after response

### HTTP/1.1
- **Keep-Alive**: Reuse connections
- **Pipelining**: Send multiple requests without waiting

```http
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

## HTTP/2 Improvements

- **Multiplexing**: Multiple requests over single connection
- **Header Compression**: Reduced overhead
- **Server Push**: Server can send resources proactively
- **Binary Protocol**: More efficient than text-based HTTP/1.1

## Common HTTP Use Cases

### **Web Browsing**
```http
GET /index.html HTTP/1.1
Host: www.example.com
```

### **API Communication**
```http
POST /api/v1/orders HTTP/1.1
Content-Type: application/json
Authorization: Bearer token

{"product_id": 123, "quantity": 2}
```

### **File Upload**
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="document.pdf"
Content-Type: application/pdf

[binary file data]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

## Best Practices

### **Performance**
- Use appropriate HTTP methods
- Implement caching strategies
- Compress responses (gzip)
- Use CDNs for static content

### **Security**
- Always use HTTPS for sensitive data
- Implement proper authentication
- Validate all input
- Use security headers (HSTS, CSP)

### **API Design**
- Follow RESTful principles
- Use meaningful status codes
- Implement proper error handling
- Version your APIs

## Debugging HTTP

### **Browser Developer Tools**
- Network tab shows all HTTP requests/responses
- Headers, timing, and payload inspection

### **Command Line Tools**
```bash
# curl - make HTTP requests
curl -X GET https://api.example.com/users/123 \
  -H "Authorization: Bearer token" \
  -H "Accept: application/json"

# httpie - user-friendly HTTP client
http GET api.example.com/users/123 \
  Authorization:"Bearer token" \
  Accept:application/json
```

## Summary

HTTP is the foundation of web communication, providing a simple yet powerful request-response protocol. Understanding HTTP is crucial for web development, API design, and debugging network issues. Modern applications rely heavily on HTTP for everything from web browsing to microservice communication, making it an essential protocol for any developer to master.