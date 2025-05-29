# Client Server Architecture

## Overview

**Client-Server Architecture** is a distributed computing model where tasks are divided between service providers (servers) and service requesters (clients). This fundamental architectural pattern forms the backbone of most modern networked applications and web services.

## Basic Concepts

### **Client**
- **Initiates requests** for services or resources
- **Consumes services** provided by servers
- Can be applications, web browsers, mobile apps, or other systems
- Typically has a user interface for human interaction

### **Server**
- **Provides services** or resources to clients
- **Processes client requests** and sends responses
- Manages shared resources (databases, files, applications)
- Usually runs continuously waiting for client requests

### **Network**
- **Communication medium** between clients and servers
- Can be internet, intranet, or local network
- Uses protocols like HTTP, TCP/IP, WebSockets

## Basic Client-Server Model

```
┌──────────┐     Request      ┌──────────┐
│  Client  │ ───────────────→ │  Server  │
│          │                  │          │
│          │ ←─────────────── │          │
└──────────┘     Response     └──────────┘
```

## Communication Flow

### **1. Connection Establishment**
```
Client ──→ Server: "I want to connect"
Server ──→ Client: "Connection accepted"
```

### **2. Request-Response Cycle**
```
Client ──→ Server: "Please give me user data for ID 123"
Server ──→ Client: "Here's the user data: {name: 'John', ...}"
```

### **3. Connection Termination**
```
Client ──→ Server: "I'm done, closing connection"
Server ──→ Client: "Connection closed"
```

## Types of Client-Server Architecture

### **1. Two-Tier Architecture**
Direct communication between client and server (database server).

```
┌──────────────┐    Direct DB    ┌──────────────┐
│    Client    │    Connection   │   Database   │
│ Application  │ ──────────────→ │    Server    │
└──────────────┘                 └──────────────┘
```

**Characteristics**:
- Simple architecture
- Client directly accesses database
- Business logic on client side
- Limited scalability

**Use Cases**:
- Small applications
- Desktop database applications
- Internal tools with few users

### **2. Three-Tier Architecture**
Separation of presentation, business logic, and data layers.

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ Presentation │      │   Business   │      │     Data     │
│    Layer     │ ───→ │    Logic     │ ───→ │    Layer     │
│  (Client)    │      │   (Server)   │      │ (Database)   │
└──────────────┘      └──────────────┘      └──────────────┘
```

**Characteristics**:
- Clear separation of concerns
- Business logic centralized on server
- Better scalability and maintainability
- More secure

**Use Cases**:
- Web applications
- Enterprise applications
- E-commerce systems

### **3. N-Tier (Multi-Tier) Architecture**
Multiple layers with specialized responsibilities.

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│Presentation │──→│   Web/API   │──→│  Business   │──→│    Data     │
│   Layer     │   │   Gateway   │   │   Logic     │   │   Layer     │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
                                           │
                                           ▼
                                   ┌─────────────┐
                                   │  External   │
                                   │  Services   │
                                   └─────────────┘
```

**Characteristics**:
- Highly modular and scalable
- Each tier can be scaled independently
- Complex to design and maintain
- Better fault tolerance

## Common Client-Server Patterns

### **Web Browser - Web Server**
```
┌──────────────┐   HTTP Request   ┌──────────────┐
│ Web Browser  │ ──────────────→ │  Web Server  │
│   (Client)   │                 │   (Apache,   │
│              │ ←────────────── │    Nginx)    │
└──────────────┘   HTML/CSS/JS   └──────────────┘
```

### **Mobile App - API Server**
```
┌──────────────┐   REST/GraphQL   ┌──────────────┐
│  Mobile App  │ ──────────────→ │  API Server  │
│   (Client)   │                 │   (Node.js,  │
│              │ ←────────────── │    Python)   │
└──────────────┘   JSON Response └──────────────┘
```

### **Desktop App - Database Server**
```
┌──────────────┐   SQL Queries    ┌──────────────┐
│ Desktop App  │ ──────────────→ │   Database   │
│   (Client)   │                 │    Server    │
│              │ ←────────────── │ (PostgreSQL, │
└──────────────┘   Result Sets   │   MongoDB)   │
                                 └──────────────┘
```

## Client-Server Communication Protocols

### **HTTP/HTTPS**
- Most common web protocol
- Request-response model
- Stateless communication
- RESTful APIs and web applications

```http
GET /api/users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer token123

HTTP/1.1 200 OK
Content-Type: application/json
{"id": 123, "name": "John Doe"}
```

### **TCP/IP**
- Lower-level protocol
- Reliable, connection-oriented
- Custom application protocols
- Real-time applications

```python
# TCP Client Example
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('server.example.com', 8080))
client.send(b'Hello Server')
response = client.recv(1024)
client.close()
```

### **WebSockets**
- Full-duplex communication
- Real-time bidirectional data flow
- Persistent connections
- Chat applications, live updates

```javascript
// WebSocket Client
const ws = new WebSocket('wss://api.example.com/ws');
ws.onmessage = (event) => {
    console.log('Received:', event.data);
};
ws.send('Hello Server');
```

### **gRPC**
- High-performance RPC framework
- Protocol Buffers for serialization
- Strongly typed contracts
- Microservices communication

```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
}

message GetUserRequest {
  int32 user_id = 1;
}
```

## Advantages of Client-Server Architecture

### **Centralized Management**
- **Data Consistency**: Single source of truth
- **Security**: Centralized access control
- **Backup**: Centralized data backup and recovery
- **Updates**: Server-side updates benefit all clients

### **Resource Sharing**
- **Hardware**: Multiple clients share server resources
- **Software**: Applications installed once on server
- **Data**: Shared databases and file systems
- **Services**: Centralized business logic

### **Scalability**
- **Horizontal Scaling**: Add more servers
- **Vertical Scaling**: Upgrade server hardware
- **Load Distribution**: Balance load across servers
- **Client Independence**: Add clients without server changes

### **Security**
- **Centralized Authentication**: Single sign-on capabilities
- **Access Control**: Fine-grained permissions
- **Data Protection**: Sensitive data stays on server
- **Audit Trail**: Centralized logging and monitoring

## Disadvantages and Challenges

### **Single Point of Failure**
- Server downtime affects all clients
- Network issues block communication
- Mitigation: Redundancy, clustering, failover

### **Network Dependency**
- Requires stable network connection
- Performance affected by network latency
- Offline capabilities limited

### **Scalability Bottlenecks**
- Server resources can become limiting factor
- Database connections may be constrained
- Network bandwidth limitations

### **Security Vulnerabilities**
- Server compromise affects all clients
- Network attacks (man-in-the-middle)
- DDoS attacks on servers

## Implementation Examples

### **Simple HTTP API Server (Node.js)**
```javascript
const express = require('express');
const app = express();

app.use(express.json());

// GET endpoint
app.get('/api/users/:id', (req, res) => {
    const userId = req.params.id;
    // Fetch user from database
    const user = database.getUser(userId);
    res.json(user);
});

// POST endpoint
app.post('/api/users', (req, res) => {
    const userData = req.body;
    // Create user in database
    const newUser = database.createUser(userData);
    res.status(201).json(newUser);
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### **Simple HTTP Client (Python)**
```python
import requests

# GET request
response = requests.get('http://api.example.com/users/123')
if response.status_code == 200:
    user = response.json()
    print(f"User: {user['name']}")

# POST request
new_user = {
    'name': 'John Doe',
    'email': 'john@example.com'
}
response = requests.post('http://api.example.com/users', json=new_user)
if response.status_code == 201:
    print("User created successfully")
```

### **Database Client-Server (SQL)**
```python
import psycopg2

# Connect to PostgreSQL server
conn = psycopg2.connect(
    host='db.example.com',
    database='myapp',
    user='username',
    password='password'
)

cursor = conn.cursor()

# Execute query on server
cursor.execute('SELECT id, name FROM users WHERE id = %s', (123,))
user = cursor.fetchone()

conn.close()
```

## Modern Client-Server Patterns

### **Microservices Architecture**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Gateway   │    │   Service   │
│ Application │──→ │   (Proxy)   │──→ │     A       │
└─────────────┘    └─────────────┘    └─────────────┘
                          │           ┌─────────────┐
                          └─────────→ │   Service   │
                                      │     B       │
                                      └─────────────┘
```

### **API Gateway Pattern**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Mobile    │    │             │    │   User      │
│    App      │──→ │             │──→ │  Service    │
└─────────────┘    │             │    └─────────────┘
┌─────────────┐    │  API        │    ┌─────────────┐
│    Web      │──→ │ Gateway     │──→ │   Order     │
│    App      │    │             │    │  Service    │
└─────────────┘    │             │    └─────────────┘
┌─────────────┐    │             │    ┌─────────────┐
│  Partner    │──→ │             │──→ │  Payment    │
│    API      │    │             │    │  Service    │
└─────────────┘    └─────────────┘    └─────────────┘
```

### **Server-Sent Events (SSE)**
```javascript
// Client receives real-time updates
const eventSource = new EventSource('/api/events');

eventSource.onmessage = function(event) {
    const data = JSON.parse(event.data);
    updateUI(data);
};

// Server sends updates
app.get('/api/events', (req, res) => {
    res.writeHead(200, {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive'
    });
    
    setInterval(() => {
        res.write(`data: ${JSON.stringify({timestamp: Date.now()})}\n\n`);
    }, 1000);
});
```

## Load Balancing Strategies

### **Round Robin**
```
Client ──→ Load Balancer ──→ Server 1
                         ──→ Server 2
                         ──→ Server 3
```

### **Weighted Round Robin**
```
Client ──→ Load Balancer ──→ Server 1 (Weight: 3)
                         ──→ Server 2 (Weight: 2)
                         ──→ Server 3 (Weight: 1)
```

### **Least Connections**
- Route to server with fewest active connections
- Better for long-running requests

### **IP Hash**
- Route based on client IP hash
- Ensures session affinity

## Caching Strategies

### **Client-Side Caching**
```javascript
// Browser cache
localStorage.setItem('user_123', JSON.stringify(userData));

// HTTP cache headers
Cache-Control: max-age=3600
ETag: "abc123"
```

### **Server-Side Caching**
```javascript
// In-memory cache
const cache = new Map();

app.get('/api/users/:id', (req, res) => {
    const userId = req.params.id;
    
    if (cache.has(userId)) {
        return res.json(cache.get(userId));
    }
    
    const user = database.getUser(userId);
    cache.set(userId, user);
    res.json(user);
});
```

### **Distributed Caching**
```javascript
// Redis cache
const redis = require('redis');
const client = redis.createClient();

app.get('/api/users/:id', async (req, res) => {
    const userId = req.params.id;
    
    // Check cache first
    const cached = await client.get(`user:${userId}`);
    if (cached) {
        return res.json(JSON.parse(cached));
    }
    
    // Fetch from database
    const user = await database.getUser(userId);
    
    // Cache for 1 hour
    await client.setex(`user:${userId}`, 3600, JSON.stringify(user));
    
    res.json(user);
});
```

## Security Considerations

### **Authentication**
```http
# Bearer token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# API key
X-API-Key: your-api-key-here

# Basic auth
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

### **HTTPS/TLS**
- Encrypt all client-server communication
- Prevent man-in-the-middle attacks
- Certificate validation

### **Input Validation**
```javascript
app.post('/api/users', (req, res) => {
    // Validate input
    const { error } = userSchema.validate(req.body);
    if (error) {
        return res.status(400).json({ error: error.details });
    }
    
    // Sanitize input
    const sanitizedData = sanitize(req.body);
    
    // Process request
    const user = database.createUser(sanitizedData);
    res.json(user);
});
```

### **Rate Limiting**
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});

app.use('/api/', limiter);
```

## Best Practices

### **API Design**
- Follow RESTful principles
- Use appropriate HTTP methods and status codes
- Implement proper error handling
- Version your APIs

### **Performance**
- Implement caching at multiple levels
- Use compression (gzip)
- Optimize database queries
- Implement pagination for large datasets

### **Scalability**
- Design for horizontal scaling
- Use load balancers
- Implement circuit breakers
- Monitor and alert on performance metrics

### **Security**
- Use HTTPS everywhere
- Implement proper authentication and authorization
- Validate and sanitize all inputs
- Regular security audits

### **Monitoring**
- Log all requests and responses
- Monitor server health and performance
- Track error rates and response times
- Implement distributed tracing

## Summary

Client-Server architecture is a fundamental pattern that enables distributed computing and forms the basis for most modern applications. Understanding its principles, advantages, and challenges is crucial for designing scalable, secure, and maintainable systems. The pattern continues to evolve with new technologies like microservices, serverless computing, and edge computing, but the core concepts remain essential for system architects and developers.