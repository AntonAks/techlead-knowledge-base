# Tech Lead Knowledge Base

This repository contains organized knowledge for preparing to be a Tech Lead.  
Topics are grouped into sections and subtopics in Markdown format.

## 📂 Sections

### 📚 [Databases](./databases/README.md)
Covers relational and non-relational database systems, principles, and real-world practices.

- **RDBMS**
  - [Relationships](./databases/rdbms/relationships.md)
  - [CRUD](./databases/rdbms/crud.md)
  - [ACID](./databases/rdbms/acid.md)
  - [Transactions](./databases/rdbms/transactions.md)
  - [Normalization](./databases/rdbms/normalization.md)
  - [Indexes](./databases/rdbms/indexes.md)
  - [Views](./databases/rdbms/views.md)
  - [Transaction Isolation Levels](./databases/rdbms/transaction-isolation-levels.md)
  - [Backup & Recovery](./databases/rdbms/backup-recovery.md)
  - [Stored Procedures, Functions & Triggers](./databases/rdbms/stored-procedures-functions-triggers.md)
  - [Query Optimization](./databases/rdbms/query-optimization.md)
  - [EXPLAIN](./databases/rdbms/explain.md)

- **NoSQL**
  - [Basics (DB, Collection, Document)](./databases/nosql/basics.md)
  - [CRUD](./databases/nosql/crud.md)
  - [Aggregation](./databases/nosql/aggregation.md)
  - [Indexing](./databases/nosql/indexing.md)
  - [Search](./databases/nosql/search.md)
  - [Key-Value DBs](./databases/nosql/key-value-databases.md)
  - [Memcached](./databases/nosql/memcached.md)
  - [Cassandra](./databases/nosql/cassandra.md)
  - [Redis](./databases/nosql/redis.md)
  - [Consistency Models (ACID, BASE, CAP)](./databases/nosql/consistency-models.md)
  - [NoSQL vs RDBMS](./databases/nosql/rdbms-vs-nosql.md)
  - [Transactions in NoSQL](./databases/nosql/transactions.md)
  - [Replication](./databases/nosql/replication.md)
  - [Sharding](./databases/nosql/sharding.md)
  - [Sharding Strategies](./databases/nosql/sharding-strategies.md)
  - [MapReduce](./databases/nosql/map-reduce.md)
  - [Graph Databases](./databases/nosql/graph-databases.md)
  - [Elasticsearch](./databases/nosql/elasticsearch.md)

### 🌐 [Networking](./networking/README.md)
Covers fundamental networking concepts, protocols, and tools essential for understanding network communication and infrastructure.

- **Network Fundamentals**
  - [IPv4 Address Structure](./networking/ipv4-address-structure.md)
  - [IPv4 vs IPv6](./networking/ipv4-vs-ipv6.md)
  - [Network Address Translation (NAT)](./networking/network-address-translation.md)

- **Network Commands and Tools**
  - [Basic TCP/IP Commands](./networking/basic-tcpip-commands.md)
  - [UNIX Network Commands](./networking/unix-network-commands.md)
  - [Windows Network Commands](./networking/windows-network-commands.md)

- **Network Security and Protocols**
  - [Virtual Private Networks (VPNs)](./networking/vpns.md)
  - [SSL, TLS and HTTPS](./networking/ssl-tls-https.md)
  - [TCP vs UDP](./networking/tcp-vs-udp.md)

- **Protocol Deep Dives**
  - [TCP/IP Protocol Suite](./networking/tcpip-protocol-suite.md)
  - [Network Routing](./networking/network-routing.md)
  - [Network Troubleshooting](./networking/network-troubleshooting.md)

### 🔗 [Web Services](./web-services/README.md)
Covers web services, APIs, protocols, and architectural patterns for distributed systems.

- **Fundamentals**
  - [What is a Web Service?](./web-services/what-is-web-service.md)
  - [JSON](./web-services/json.md)
  - [Client Server Architecture](./web-services/client-server-architecture.md)

- **HTTP & Protocols**
  - [HTTP Overview](./web-services/http-overview.md)
  - [HTTP Request Methods](./web-services/http-request-methods.md)
  - [HTTP Status Codes](./web-services/http-status-codes.md)
  - [HTTP vs HTTPS](./web-services/http-vs-https.md)
  - [HTTPv1 vs HTTPv2](./web-services/httpv1-vs-httpv2.md)

- **Authentication & Security**
  - [Auth & OAuth](./web-services/auth-oauth.md)
  - [SSO (Single Sign-On)](./web-services/sso.md)
  - [JSON Web Tokens (JWT)](./web-services/json-web-tokens.md)

- **Architectural Patterns**
  - [SOA (Service-Oriented Architecture)](./web-services/soa.md)
  - [ROA (Resource-Oriented Architecture)](./web-services/roa.md)
  - [REST](./web-services/rest.md)
  - [Richardson Maturity Model](./web-services/richardson-maturity-model.md)

- **API Standards & Documentation**
  - [OpenAPI/Swagger](./web-services/openapi-swagger.md)
  - [Hypermedia/HATEOAS](./web-services/hypermedia-hateoas.md)

- **Technologies & Protocols**
  - [SOAP](./web-services/soap.md)
  - [gRPC](./web-services/grpc.md)
  - [WebSockets](./web-services/websockets.md)

### 🏗️ [System Design](./system-design/README.md)
Comprehensive system design concepts, patterns, and best practices for building scalable distributed systems.

- **Scalability Fundamentals**
  - [Horizontal vs Vertical Scaling](./system-design/scalability-fundamentals/horizontal-vs-vertical-scaling.md)
  - [Load Balancing Strategies](./system-design/scalability-fundamentals/load-balancing-strategies.md)
  - [Caching Layers and Patterns](./system-design/scalability-fundamentals/caching-layers-and-patterns.md)
  - [Database Scaling (Sharding, Replication)](./system-design/scalability-fundamentals/database-scaling.md)

- **Reliability & Availability**
  - [CAP Theorem and Consistency Models](./system-design/reliability-availability/cap-theorem-consistency-models.md)
  - [Circuit Breakers and Fault Tolerance](./system-design/reliability-availability/circuit-breakers-fault-tolerance.md)
  - [Graceful Degradation Patterns](./system-design/reliability-availability/graceful-degradation-patterns.md)
  - [SLA/SLO/SLI Design](./system-design/reliability-availability/sla-slo-sli-design.md)

- **Service Architecture**
  - [Microservices vs Monolith Trade-offs](./system-design/service-architecture/microservices-vs-monolith.md)
  - [Service Mesh and API Gateways](./system-design/service-architecture/service-mesh-api-gateways.md)
  - [Event-Driven Architecture](./system-design/service-architecture/event-driven-architecture.md)
  - [Message Queues and Event Streaming](./system-design/service-architecture/message-queues-event-streaming.md)

- **Data Architecture**
  - [Database Selection Criteria](./system-design/data-architecture/database-selection-criteria.md)
  - [Polyglot Persistence](./system-design/data-architecture/polyglot-persistence.md)
  - [Data Consistency Patterns](./system-design/data-architecture/data-consistency-patterns.md)
  - [CQRS and Event Sourcing](./system-design/data-architecture/cqrs-event-sourcing.md)

- **Performance & Monitoring**
  - [Capacity Planning and Auto-scaling](./system-design/performance-monitoring/capacity-planning-auto-scaling.md)
  - [Performance Monitoring and Observability](./system-design/performance-monitoring/performance-monitoring-observability.md)
  - [Distributed Tracing](./system-design/performance-monitoring/distributed-tracing.md)
  - [Metrics and Alerting](./system-design/performance-monitoring/metrics-and-alerting.md)

- **Cloud Patterns**
  - [Multi-region Architecture](./system-design/cloud-patterns/multi-region-architecture.md)
  - [Disaster Recovery Strategies](./system-design/cloud-patterns/disaster-recovery-strategies.md)
  - [Serverless Architecture Patterns](./system-design/cloud-patterns/serverless-architecture-patterns.md)
  - [Container Orchestration Design](./system-design/cloud-patterns/container-orchestration-design.md)

- **Security & Compliance**
  - [Zero Trust Architecture](./system-design/security-compliance/zero-trust-architecture.md)
  - [Identity and Access Management](./system-design/security-compliance/identity-access-management.md)
  - [Data Protection and Privacy](./system-design/security-compliance/data-protection-privacy.md)
  - [Compliance and Audit Patterns](./system-design/security-compliance/compliance-audit-patterns.md)

- **Technical Leadership**
  - [Architecture Decision Records (ADRs)](./system-design/technical-leadership/architecture-decision-records.md)
  - [Technology Evaluation Frameworks](./system-design/technical-leadership/technology-evaluation-frameworks.md)
  - [Technical Debt Management](./system-design/technical-leadership/technical-debt-management.md)
  - [Migration Strategies and Planning](./system-design/technical-leadership/migration-strategies-planning.md)