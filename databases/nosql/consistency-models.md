# Consistency Models (ACID, BASE, CAP Theorem)

## ACID Properties
Traditional relational database properties that ensure reliable transaction processing.

### Atomicity
**Definition**: Transactions are all-or-nothing operations.

**Characteristics**:
- Either all operations in a transaction complete successfully, or none do
- No partial transaction states are visible to other transactions
- System remains in a consistent state even if transaction fails
- Rollback mechanisms ensure atomicity

**Example**:
```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
COMMIT; -- Both updates succeed or both fail
```

**Implementation Mechanisms**:
- Write-ahead logging (WAL)
- Shadow paging
- Transaction isolation
- Rollback segments

### Consistency
**Definition**: Database moves from one valid state to another valid state.

**Characteristics**:
- All database constraints are maintained
- Referential integrity is preserved
- Business rules are enforced
- Data validation occurs before commit

**Types of Consistency**:
- **Entity Integrity**: Primary key constraints
- **Referential Integrity**: Foreign key constraints
- **Domain Integrity**: Data type and value constraints
- **User-Defined Integrity**: Custom business rules

**Example**:
```sql
-- Constraint ensures consistency
ALTER TABLE orders 
ADD CONSTRAINT check_positive_amount 
CHECK (amount > 0);

-- This insert would fail due to consistency constraint
INSERT INTO orders (id, amount) VALUES (1, -50); -- ERROR
```

### Isolation
**Definition**: Concurrent transactions don't interfere with each other.

**Isolation Levels**:

#### Read Uncommitted (Level 0)
- Transactions can read uncommitted changes
- Dirty reads possible
- Lowest isolation, highest performance

#### Read Committed (Level 1)
- Transactions only read committed data
- Prevents dirty reads
- Non-repeatable reads still possible

#### Repeatable Read (Level 2)
- Same data returns same values within transaction
- Prevents dirty and non-repeatable reads
- Phantom reads still possible

#### Serializable (Level 3)
- Highest isolation level
- Transactions appear to execute serially
- Prevents all read phenomena

**Common Read Phenomena**:
```sql
-- Dirty Read Example
Transaction A: UPDATE products SET price = 150 WHERE id = 1; -- Not committed
Transaction B: SELECT price FROM products WHERE id = 1; -- Reads 150 (dirty)
Transaction A: ROLLBACK; -- Price reverts to original value
```

**Implementation Mechanisms**:
- Locking (shared, exclusive, intention locks)
- Multi-version concurrency control (MVCC)
- Timestamp ordering
- Snapshot isolation

### Durability
**Definition**: Committed transactions persist even after system failure.

**Characteristics**:
- Data survives hardware failures, power outages, crashes
- Changes are written to non-volatile storage
- Recovery mechanisms restore committed data
- Backup and recovery procedures ensure durability

**Implementation Mechanisms**:
- Write-ahead logging
- Force-write policies
- Database checkpoints
- Redundant storage systems
- Transaction log archiving

**Example Scenario**:
```sql
INSERT INTO orders (id, customer_id, amount) VALUES (123, 456, 100.00);
COMMIT; -- Transaction committed to disk

-- Even if system crashes immediately after commit,
-- the order record will be available after recovery
```

## BASE Properties
NoSQL alternative to ACID, designed for distributed systems that prioritize availability and scalability.

### Basically Available
**Definition**: System remains available for operations most of the time.

**Characteristics**:
- System may not be available 100% of the time
- Graceful degradation under high load or failures
- Partial failures don't bring down entire system
- Best-effort availability

**Implementation Strategies**:
- Load balancing and failover
- Circuit breakers
- Graceful degradation
- Redundancy and replication

**Example**:
```javascript
// E-commerce site during high traffic
if (inventoryService.isDown()) {
  return "Product availability information temporarily unavailable";
} else {
  return inventoryService.getStock(productId);
}
```

### Soft State
**Definition**: System state may change over time, even without new input.

**Characteristics**:
- Data may be in flux during consistency propagation
- Intermediate states are acceptable
- System doesn't guarantee immediate consistency
- State changes due to eventual consistency mechanisms

**Examples**:
- DNS propagation across servers
- Cache invalidation and refresh
- Distributed counter updates
- Social media "like" counts

**Implementation Patterns**:
```javascript
// Counter that updates eventually
class DistributedCounter {
  constructor() {
    this.localCount = 0;
    this.lastSyncTime = Date.now();
  }
  
  increment() {
    this.localCount++;
    // Async sync with other nodes
    this.scheduleSync();
  }
  
  getApproximateCount() {
    // Returns soft state - may not be exact
    return this.localCount + this.estimatedRemoteCount;
  }
}
```

### Eventually Consistent
**Definition**: System will become consistent over time, given that no new updates occur.

**Characteristics**:
- Immediate consistency not guaranteed
- Consistency achieved through propagation mechanisms
- Updates eventually reach all replicas
- Conflicts resolved through various strategies

**Consistency Models**:
- **Strong Eventual Consistency**: Replicas that have received same updates have same state
- **Weak Consistency**: No guarantees on when data will be consistent
- **Causal Consistency**: Causally related operations appear in same order

**Conflict Resolution Strategies**:
```javascript
// Last-Write-Wins (LWW)
function resolveConflict(localValue, remoteValue) {
  return localValue.timestamp > remoteValue.timestamp ? 
    localValue : remoteValue;
}

// Vector Clocks for causal ordering
class VectorClock {
  constructor(nodeId) {
    this.nodeId = nodeId;
    this.clock = {};
  }
  
  tick() {
    this.clock[this.nodeId] = (this.clock[this.nodeId] || 0) + 1;
  }
  
  update(otherClock) {
    for (let node in otherClock) {
      this.clock[node] = Math.max(this.clock[node] || 0, otherClock[node]);
    }
  }
}
```

## CAP Theorem
States that in a distributed system, you can only guarantee two of the following three properties simultaneously.

### Consistency (C)
**Definition**: All nodes see the same data at the same time.

**Characteristics**:
- Strong consistency ensures immediate updates across all nodes
- All reads receive the most recent write
- Linearizability or sequential consistency
- May require coordination between nodes

**Implementation Approaches**:
- Consensus protocols (Raft, PBFT)
- Two-phase commit
- Distributed locking
- Synchronous replication

**Trade-offs**:
- Higher latency due to coordination
- Reduced availability during network partitions
- More complex implementation

### Availability (A)
**Definition**: System remains operational and responsive.

**Characteristics**:
- Every request receives a response (success or failure)
- System continues operating despite failures
- No single point of failure
- Graceful handling of node failures

**Implementation Strategies**:
- Redundancy and replication
- Load balancing
- Failover mechanisms
- Asynchronous processing

**Trade-offs**:
- May serve stale or inconsistent data
- Complexity in handling conflicts
- Eventual consistency requirements

### Partition Tolerance (P)
**Definition**: System continues to operate despite network failures.

**Characteristics**:
- Can handle communication breakdowns between nodes
- Essential for truly distributed systems
- Network splits don't cause system failure
- Nodes can operate independently when partitioned

**Implementation Requirements**:
- Distributed architecture
- Independent node operation
- Partition detection mechanisms
- Recovery protocols

**Real-world Considerations**:
- Network partitions are inevitable in distributed systems
- Must choose between consistency and availability during partitions
- Recovery strategies for partition healing

## CAP Theorem in Practice

### CP Systems (Consistency + Partition Tolerance)
**Examples**: MongoDB (with strong consistency), HBase, Redis Cluster

**Characteristics**:
- Sacrifice availability during network partitions
- Ensure data consistency across available nodes
- May become unavailable if partition prevents majority consensus

**Use Cases**:
- Financial systems requiring strong consistency
- Systems where stale data is unacceptable
- Applications requiring ACID properties

**Implementation Example**:
```python
# MongoDB with strong read concern
collection.find_one(
  {"_id": user_id},
  read_concern=ReadConcern("majority")  # Ensures consistency
)
```

### AP Systems (Availability + Partition Tolerance)
**Examples**: Cassandra, DynamoDB, CouchDB

**Characteristics**:
- Remain available during network partitions
- Accept temporary inconsistency
- Eventually converge to consistent state

**Use Cases**:
- Social media platforms
- Content delivery networks
- Systems where availability is critical

**Implementation Example**:
```python
# Cassandra with eventual consistency
session.execute(
  "SELECT * FROM users WHERE user_id = %s",
  [user_id],
  consistency_level=ConsistencyLevel.ONE  # Prioritizes availability
)
```

### CA Systems (Consistency + Availability)
**Examples**: Traditional RDBMS in single-node deployment

**Characteristics**:
- Cannot handle network partitions
- Not truly distributed systems
- Suitable for single-node or tightly coupled systems

**Limitations**:
- Single point of failure
- Limited scalability
- Not suitable for distributed architectures

## Consistency Patterns in Distributed Systems

### Strong Consistency
**Definition**: All reads receive the most recent write.

**Implementation**:
- Synchronous replication
- Consensus protocols
- Distributed locking

**Trade-offs**:
- Higher latency
- Reduced availability
- Complex coordination

### Weak Consistency
**Definition**: No guarantees about when data will be consistent.

**Characteristics**:
- Best effort approach
- Application must handle inconsistencies
- High performance and availability

**Use Cases**:
- Real-time gaming
- Live video streaming
- Voice over IP

### Eventual Consistency
**Definition**: System becomes consistent over time.

**Variants**:
- **Causal Consistency**: Causally related operations ordered
- **Read-your-writes**: Users see their own updates immediately
- **Session Consistency**: Consistency within user session
- **Monotonic Read Consistency**: No reading older values after newer ones

### Implementation Patterns

#### Read Repair
```javascript
// Cassandra read repair example
function readWithRepair(key) {
  let responses = readFromAllReplicas(key);
  let latestValue = findLatestValue(responses);
  
  // Repair inconsistent replicas
  responses.forEach(response => {
    if (response.timestamp < latestValue.timestamp) {
      repairReplica(response.node, key, latestValue);
    }
  });
  
  return latestValue;
}
```

#### Hinted Handoff
```javascript
// Handle temporary node failures
function writeWithHintedHandoff(key, value) {
  let targetNodes = getReplicaNodes(key);
  let availableNodes = targetNodes.filter(node => node.isAvailable());
  
  if (availableNodes.length < requiredReplicas) {
    // Store hints for unavailable nodes
    let hints = createHints(key, value, unavailableNodes);
    storeHints(hints);
  }
  
  return writeToNodes(availableNodes, key, value);
}
```

## Choosing the Right Consistency Model

### Factors to Consider

#### Application Requirements
- Data accuracy requirements
- Acceptable inconsistency windows
- User experience expectations
- Business rule constraints

#### System Characteristics
- Network reliability
- Geographic distribution
- Scalability requirements
- Performance needs

#### Trade-off Analysis
- Consistency vs. Performance vs. Availability
- Development complexity
- Operational overhead
- Cost implications

### Decision Matrix

| Use Case | Consistency Model | Example Systems |
|----------|------------------|-----------------|
| Banking/Financial | Strong Consistency | Traditional RDBMS, MongoDB |
| Social Media | Eventual Consistency | Cassandra, DynamoDB |
| Gaming Leaderboards | Weak Consistency | Redis, In-memory systems |
| E-commerce Catalog | Eventual Consistency | Elasticsearch, CouchDB |
| Real-time Analytics | Weak/Eventual | Apache Kafka, ClickHouse |
| User Sessions | Session Consistency | Application-level caching |