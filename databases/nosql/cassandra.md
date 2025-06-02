# Apache Cassandra

## Overview
Apache Cassandra is a distributed NoSQL database designed for handling large amounts of data across multiple commodity servers with no single point of failure. It provides high availability and linear scalability.

## Key Characteristics
- **Distributed Architecture**: Data is distributed across multiple nodes in a cluster
- **Masterless Design**: No single point of failure, all nodes are equal
- **Column-Family Data Model**: Similar to a table but with flexible schema
- **Eventual Consistency**: Prioritizes availability over immediate consistency
- **Tunable Consistency**: Allows configuration of consistency levels per operation

## Architecture Components

### Node
Individual server in the cluster that stores data and participates in the cluster operations.

### Cluster
Collection of nodes that work together to provide distributed database functionality.

### Keyspace
Top-level namespace (similar to database in RDBMS) that defines replication strategy and other properties.

### Column Family/Table
Container for data rows, similar to tables in relational databases but with flexible schema.

### Partition Key
Determines which node stores the data. Critical for data distribution and query performance.

### Clustering Key
Determines ordering of data within a partition. Used for range queries and sorting.

## Data Model Structure
```
Keyspace
├── Column Family (Table)
    ├── Row (identified by partition key)
        ├── Columns (name-value pairs with timestamps)
```

## Consistency Levels

### Write Consistency Levels
- **ONE**: Only one replica needs to acknowledge the write
- **QUORUM**: Majority of replicas must acknowledge the write
- **ALL**: All replicas must acknowledge the write
- **LOCAL_QUORUM**: Majority within local datacenter must acknowledge

### Read Consistency Levels
- **ONE**: Only one replica needs to respond
- **QUORUM**: Majority of replicas must respond with consistent data
- **ALL**: All replicas must respond
- **LOCAL_QUORUM**: Majority within local datacenter must respond

### Consistency Level Examples
```sql
-- Write with specific consistency
INSERT INTO users (id, name, email) VALUES (1, 'John', 'john@example.com') 
USING CONSISTENCY QUORUM;

-- Read with specific consistency
SELECT * FROM users WHERE id = 1 USING CONSISTENCY ONE;
```

## CQL (Cassandra Query Language) Examples

### Keyspace Operations
```sql
-- Create keyspace
CREATE KEYSPACE mykeyspace WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};

-- Use keyspace
USE mykeyspace;
```

### Table Operations
```sql
-- Create table
CREATE TABLE users (
  id UUID PRIMARY KEY,
  name TEXT,
  email TEXT,
  created_at TIMESTAMP
);

-- Create table with compound primary key
CREATE TABLE user_activities (
  user_id UUID,
  activity_date DATE,
  activity_type TEXT,
  details TEXT,
  PRIMARY KEY (user_id, activity_date)
);
```

### Data Operations
```sql
-- Insert data
INSERT INTO users (id, name, email, created_at) 
VALUES (uuid(), 'John Doe', 'john@example.com', toTimestamp(now()));

-- Update data
UPDATE users SET email = 'newemail@example.com' WHERE id = user_id;

-- Delete data
DELETE FROM users WHERE id = user_id;

-- Select data
SELECT * FROM users WHERE id = user_id;
SELECT * FROM user_activities WHERE user_id = user_id AND activity_date >= '2024-01-01';
```

## Data Modeling Best Practices

### Design for Queries
- Design tables based on query patterns
- Avoid joins and complex operations
- Denormalize data when necessary

### Partition Key Selection
- Choose keys that distribute data evenly
- Avoid hot partitions
- Consider query patterns when selecting partition keys

### Clustering Key Usage
- Use for sorting and range queries
- Order columns by query frequency
- Consider cardinality and access patterns

## Replication Strategies

### SimpleStrategy
Used for single datacenter deployments:
```sql
CREATE KEYSPACE mykeyspace WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};
```

### NetworkTopologyStrategy
Used for multi-datacenter deployments:
```sql
CREATE KEYSPACE mykeyspace WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'dc1': 3,
  'dc2': 2
};
```

## Performance Optimization

### Read Performance
- Use appropriate consistency levels
- Design efficient partition keys
- Utilize clustering columns for sorting
- Enable compression

### Write Performance
- Batch related writes
- Use prepared statements
- Configure write consistency appropriately
- Monitor compaction strategies

## Use Cases

### Time-Series Data
- IoT sensor data
- Application logs
- Metrics and monitoring data
- Financial tick data

### Real-Time Analytics
- User activity tracking
- Click stream analysis
- Real-time dashboards
- Event processing

### Content Management
- User profiles and preferences
- Product catalogs
- Content metadata
- Social media posts

### High-Write Applications
- Chat applications
- Gaming leaderboards
- Audit logs
- Event sourcing

## Advantages

### Scalability
- Linear scalability by adding nodes
- No single point of failure
- Automatic data distribution
- Handle massive datasets

### Performance
- Excellent write performance
- Low-latency reads with proper modeling
- Configurable consistency levels
- Efficient range queries with clustering

### Availability
- Always-on architecture
- Multi-datacenter replication
- Automatic failover
- Tunable consistency vs availability

### Operational
- Masterless architecture simplifies operations
- Rolling upgrades with zero downtime
- Automatic data repair
- Comprehensive monitoring tools

## Disadvantages

### Query Limitations
- No joins between tables
- Limited aggregation capabilities
- No complex transactions
- Restricted query flexibility

### Complexity
- Requires careful data modeling
- Understanding of distributed systems concepts
- Complex troubleshooting
- Learning curve for developers

### Consistency Trade-offs
- Eventually consistent by default
- Potential for data inconsistencies
- Requires understanding of consistency levels
- Complex conflict resolution

### Resource Requirements
- Memory-intensive operations
- Requires multiple nodes for production
- Network bandwidth for replication
- Storage overhead for multiple replicas

## Monitoring and Maintenance

### Key Metrics
- Read/write latency
- Throughput (operations per second)
- Compaction performance
- Disk usage and growth
- Memory utilization

### Common Issues
- Hot partitions
- Large partitions
- Compaction lag
- Network partitions
- Consistency issues

### Best Practices
- Regular backups
- Monitor cluster health
- Proper capacity planning
- Regular compaction tuning
- Security configuration

## Tools and Ecosystem

### Administration Tools
- **cqlsh**: Command-line interface
- **nodetool**: Node management utility
- **DataStax DevCenter**: GUI for development
- **OpsCenter**: Management and monitoring

### Drivers and Integration
- Official drivers for Java, Python, C#, Node.js
- Spring Data Cassandra
- Kafka Connect integration
- Spark Cassandra Connector

### Monitoring Solutions
- DataStax OpsCenter
- Prometheus + Grafana
- New Relic
- Custom JMX monitoring