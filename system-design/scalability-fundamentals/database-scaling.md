# Database Scaling (Sharding, Replication)

## Overview
Database scaling is essential for handling increased data volume and query load. There are two primary approaches: vertical scaling (scaling up) and horizontal scaling (scaling out). This guide focuses on horizontal scaling techniques including replication and sharding.

## Database Replication

### What is Replication?
Database replication involves creating and maintaining multiple copies of the same database across different servers to improve availability, performance, and fault tolerance.

### Types of Replication

#### Master-Slave (Primary-Secondary) Replication
**How it works**: One master database handles all write operations, while multiple slave databases handle read operations.

**Architecture**:
```
Master Database (Primary)
├── Write Operations
├── Binary Log
└── Slave 1 (Read Replica)
    ├── Read Operations
    └── Slave 2 (Read Replica)
        └── Read Operations
```

**Advantages**:
- Improved read performance
- High availability
- Geographic distribution
- Backup and recovery
- Load distribution

**Disadvantages**:
- Single point of failure (master)
- Replication lag
- Write bottleneck
- Complex failover

**Implementation Example**:
```sql
-- Master configuration
[mysqld]
log-bin=mysql-bin
server-id=1
binlog_format=ROW

-- Slave configuration
[mysqld]
server-id=2
relay-log=relay-bin
read_only=1
```

#### Master-Master (Multi-Master) Replication
**How it works**: Multiple databases can handle both read and write operations.

**Architecture**:
```
Master 1 (Primary)
├── Write Operations
├── Binary Log
└── Master 2 (Primary)
    ├── Write Operations
    └── Binary Log
```

**Advantages**:
- No single point of failure
- Geographic distribution
- Improved write performance
- High availability

**Disadvantages**:
- Complex conflict resolution
- Data consistency challenges
- Increased complexity
- Performance overhead

**Conflict Resolution Strategies**:
1. **Timestamp-based**: Use timestamps to resolve conflicts
2. **Application-level**: Handle conflicts in application code
3. **Last-write-wins**: Accept the most recent change
4. **Custom logic**: Implement business-specific resolution

### Replication Topologies

#### Chain Replication
```
Master → Slave 1 → Slave 2 → Slave 3
```

**Advantages**:
- Reduces master load
- Geographic distribution
- Scalable

**Disadvantages**:
- Increased latency
- Single point of failure in chain
- Complex recovery

#### Star Replication
```
        Slave 1
       /
Master → Slave 2
       \
        Slave 3
```

**Advantages**:
- Simple topology
- Low latency
- Easy management

**Disadvantages**:
- Master bottleneck
- Limited scalability

#### Tree Replication
```
Master
├── Slave 1
│   ├── Slave 1.1
│   └── Slave 1.2
└── Slave 2
    ├── Slave 2.1
    └── Slave 2.2
```

**Advantages**:
- Highly scalable
- Geographic distribution
- Load distribution

**Disadvantages**:
- Complex management
- Increased latency
- Difficult recovery

## Database Sharding

### What is Sharding?
Sharding is a database architecture pattern that horizontally partitions data across multiple databases or servers, with each shard containing a subset of the data.

### Sharding Strategies

#### Horizontal Sharding (Range-based)
**How it works**: Data is partitioned based on a range of values.

**Example**:
```sql
-- Shard 1: User IDs 1-1000000
-- Shard 2: User IDs 1000001-2000000
-- Shard 3: User IDs 2000001-3000000
```

**Advantages**:
- Simple to implement
- Easy to understand
- Good for sequential data

**Disadvantages**:
- Uneven distribution
- Hot spots
- Difficult rebalancing

#### Vertical Sharding (Column-based)
**How it works**: Different columns are stored in different databases.

**Example**:
```sql
-- Shard 1: User profiles (id, name, email)
-- Shard 2: User preferences (id, settings, preferences)
-- Shard 3: User activity (id, login_history, actions)
```

**Advantages**:
- Reduced I/O
- Better performance for specific queries
- Easier backup and recovery

**Disadvantages**:
- Complex joins across shards
- Data consistency challenges
- Application complexity

#### Hash-based Sharding
**How it works**: Data is distributed using a hash function on the shard key.

**Example**:
```python
def get_shard(user_id):
    shard_count = 4
    shard_id = hash(user_id) % shard_count
    return f"shard_{shard_id}"

# User ID 12345 → hash(12345) % 4 = 1 → shard_1
# User ID 67890 → hash(67890) % 4 = 2 → shard_2
```

**Advantages**:
- Even distribution
- Predictable performance
- Easy to scale

**Disadvantages**:
- Difficult to add/remove shards
- Complex rebalancing
- No geographic affinity

#### Directory-based Sharding
**How it works**: A lookup table determines which shard contains specific data.

**Example**:
```sql
-- Shard mapping table
CREATE TABLE shard_mapping (
    user_id INT,
    shard_id VARCHAR(10),
    PRIMARY KEY (user_id)
);

-- Lookup shard for user
SELECT shard_id FROM shard_mapping WHERE user_id = 12345;
```

**Advantages**:
- Flexible distribution
- Easy to change mapping
- Good for complex rules

**Disadvantages**:
- Additional lookup overhead
- Single point of failure
- Complex management

### Shard Key Selection

#### Criteria for Good Shard Keys
1. **High Cardinality**: Many unique values
2. **Even Distribution**: Avoids hot spots
3. **Query Affinity**: Supports common query patterns
4. **Immutable**: Doesn't change frequently
5. **Monotonic**: Avoids sequential values

#### Common Shard Keys
- **User ID**: Good for user-centric applications
- **Customer ID**: Suitable for B2B applications
- **Geographic Region**: Good for location-based services
- **Time-based**: Suitable for time-series data
- **Composite Keys**: Multiple fields combined

### Cross-Shard Operations

#### Challenges
1. **Joins**: Difficult to join data across shards
2. **Transactions**: Distributed transactions are complex
3. **Consistency**: Maintaining consistency across shards
4. **Performance**: Network overhead for cross-shard queries

#### Solutions
1. **Denormalization**: Duplicate data to avoid joins
2. **Application-level Joins**: Perform joins in application code
3. **Read Replicas**: Use replicas for complex queries
4. **Eventual Consistency**: Accept eventual consistency for some operations

## Scaling Strategies

### Read Scaling
**Approach**: Use read replicas to distribute read load.

**Implementation**:
```python
class DatabaseRouter:
    def route_read_query(self, query):
        # Route to read replica
        replica = self.get_least_loaded_replica()
        return replica.execute(query)
    
    def route_write_query(self, query):
        # Route to master
        return self.master.execute(query)
```

**Benefits**:
- Improved read performance
- Reduced master load
- Geographic distribution
- High availability

### Write Scaling
**Approach**: Distribute write operations across multiple databases.

**Strategies**:
1. **Sharding**: Partition data by write patterns
2. **CQRS**: Separate read and write models
3. **Event Sourcing**: Use events for write operations
4. **Microservices**: Distribute writes across services

### Hybrid Approaches

#### Read Replicas + Sharding
```
Shard 1 (Master) → Shard 1 Replica 1, Shard 1 Replica 2
Shard 2 (Master) → Shard 2 Replica 1, Shard 2 Replica 2
Shard 3 (Master) → Shard 3 Replica 1, Shard 3 Replica 2
```

#### Multi-Region Replication
```
Region 1: Master + Replicas
Region 2: Read Replicas
Region 3: Read Replicas
```

## Data Consistency

### Consistency Models

#### Strong Consistency
**Definition**: All replicas return the same data at the same time.

**Implementation**:
- Synchronous replication
- Distributed transactions
- Consensus protocols

**Trade-offs**:
- Higher latency
- Lower availability
- Complex implementation

#### Eventual Consistency
**Definition**: Replicas will eventually converge to the same state.

**Implementation**:
- Asynchronous replication
- Conflict resolution
- Background synchronization

**Trade-offs**:
- Lower latency
- Higher availability
- Potential for temporary inconsistencies

#### Read-Your-Writes Consistency
**Definition**: A user always sees their own writes.

**Implementation**:
- Route user's reads to same replica
- Session affinity
- Write-through caching

### Consistency Patterns

#### Saga Pattern
**How it works**: Break distributed transactions into local transactions with compensating actions.

**Example**:
```python
def create_order_saga():
    try:
        # Step 1: Reserve inventory
        inventory_service.reserve_items(order.items)
        
        # Step 2: Process payment
        payment_service.process_payment(order.payment)
        
        # Step 3: Create order
        order_service.create_order(order)
        
    except Exception as e:
        # Compensating actions
        inventory_service.release_items(order.items)
        payment_service.refund_payment(order.payment)
        raise e
```

#### Two-Phase Commit (2PC)
**How it works**: Coordinator ensures all participants commit or abort.

**Phases**:
1. **Prepare Phase**: All participants prepare to commit
2. **Commit Phase**: All participants commit or abort

**Advantages**:
- Strong consistency
- ACID properties

**Disadvantages**:
- Blocking protocol
- Performance overhead
- Complex failure handling

## Performance Optimization

### Connection Pooling
**Definition**: Reuse database connections to reduce overhead.

**Implementation**:
```python
import pymysql
from pymysql.cursors import DictCursor

class DatabasePool:
    def __init__(self, max_connections=10):
        self.pool = []
        self.max_connections = max_connections
    
    def get_connection(self):
        if not self.pool:
            return pymysql.connect(
                host='localhost',
                user='user',
                password='password',
                database='mydb',
                cursorclass=DictCursor
            )
        return self.pool.pop()
    
    def return_connection(self, connection):
        if len(self.pool) < self.max_connections:
            self.pool.append(connection)
        else:
            connection.close()
```

### Query Optimization
1. **Indexing**: Create appropriate indexes
2. **Query Rewriting**: Optimize query structure
3. **Caching**: Cache frequently accessed data
4. **Partitioning**: Partition large tables
5. **Connection Management**: Optimize connection usage

### Monitoring and Metrics

#### Key Metrics
1. **Query Performance**: Response times, throughput
2. **Connection Usage**: Active connections, connection pool utilization
3. **Replication Lag**: Time delay between master and replicas
4. **Shard Distribution**: Data distribution across shards
5. **Error Rates**: Failed queries, connection errors

#### Monitoring Tools
- **Database-specific**: MySQL Workbench, pgAdmin
- **APM Tools**: New Relic, Datadog, AppDynamics
- **Custom Metrics**: Application-level monitoring
- **Log Analysis**: Query logs, slow query logs

## Best Practices

### Planning and Design
1. **Start Simple**: Begin with read replicas before sharding
2. **Plan for Growth**: Design for future scaling needs
3. **Choose Appropriate Strategy**: Select based on data patterns
4. **Test Thoroughly**: Validate scaling strategies
5. **Document Architecture**: Maintain clear documentation

### Implementation
1. **Gradual Migration**: Migrate incrementally
2. **Backup Strategy**: Ensure data safety
3. **Rollback Plan**: Ability to revert changes
4. **Monitoring**: Comprehensive monitoring from start
5. **Testing**: Load testing and failure testing

### Operations
1. **Automation**: Automate scaling operations
2. **Monitoring**: Continuous monitoring and alerting
3. **Backup and Recovery**: Regular backups and recovery testing
4. **Capacity Planning**: Plan for future growth
5. **Documentation**: Keep operations documentation updated

## Conclusion

Database scaling is a complex but essential aspect of building scalable systems. The choice between replication and sharding depends on your specific requirements, data patterns, and growth expectations.

**Key Takeaways**:
- Start with replication for read scaling
- Use sharding when write scaling is needed
- Consider hybrid approaches for complex requirements
- Plan for data consistency challenges
- Monitor performance and adjust strategies
- Implement proper backup and recovery procedures

---
*This content provides comprehensive guidance for implementing database scaling strategies in distributed systems.* 