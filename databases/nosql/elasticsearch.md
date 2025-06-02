# Elasticsearch

## Overview
Elasticsearch is a distributed, RESTful search and analytics engine built on Apache Lucene. It's designed for horizontal scalability, reliability, and real-time search capabilities with near real-time indexing and searching.

## Core Concepts

### Cluster
A collection of one or more nodes that together holds your entire data and provides federated indexing and search capabilities.

### Node
A single server that is part of your cluster, stores your data, and participates in the cluster's indexing and search capabilities.

### Index
A collection of documents that have somewhat similar characteristics. An index is like a database in a relational database system.

### Document
The basic unit of information that can be indexed. Documents are expressed in JSON format.

### Shard
A subdivision of an index. Each shard is a fully functional and independent Lucene index that can be hosted on any node in the cluster.

### Replica
A copy of a shard. Replicas provide redundant copies of your data to protect against hardware failure and increase capacity to serve read requests.

## Architecture

```
Cluster: "my-cluster"
├── Node 1 (Master-eligible, Data)
│   ├── Index: "products"
│   │   ├── Shard 0 (Primary)
│   │   └── Shard 2 (Replica)
│   └── Index: "users"
│       └── Shard 1 (Primary)
├── Node 2 (Data)
│   ├── Index: "products"
│   │   ├── Shard 1 (Primary)
│   │   └── Shard 0 (Replica)
│   └── Index: "users"
│       └── Shard 0 (Replica)
└── Node 3 (Coordinating)
    └── Routes requests and aggregates results
```

## Usage Patterns

### Index Management

#### Create Index with Mapping
```json
PUT /products
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "custom_analyzer"
      },
      "description": {
        "type": "text",
        "analyzer": "standard"
      },
      "price": {
        "type": "double"
      },
      "category": {
        "type": "keyword"
      },
      "brand": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      },
      "created_date": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "location": {
        "type": "geo_point"
      },
      "specifications": {
        "type": "nested",
        "properties": {
          "key": {"type": "keyword"},
          "value": {"type": "text"}
        }
      }
    }
  }
}
```

#### Index Templates
```json
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "@timestamp": {"type": "date"},
        "level": {"type": "keyword"},
        "message": {"type": "text"},
        "source": {"type": "keyword"}
      }
    }
  }
}
```

### Document Operations

#### Index Documents
```json
# Index single document with auto-generated ID
POST /products/_doc
{
  "name": "iPhone 15 Pro",
  "brand": "Apple",
  "price": 999.99,
  "category": "Electronics",
  "description": "Latest iPhone with advanced features",
  "tags": ["smartphone", "iOS", "premium"],
  "created_date": "2024-06-02 10:30:00",
  "specifications": [
    {"key": "storage", "value": "256GB"},
    {"key": "color", "value": "Natural Titanium"}
  ]
}

# Index document with specific ID
PUT /products/_doc/1
{
  "name": "Samsung Galaxy S24",
  "brand": "Samsung",
  "price": 899.99,
  "category": "Electronics",
  "description": "Flagship Android smartphone",
  "tags": ["smartphone", "android", "premium"]
}

# Bulk indexing
POST /_bulk
{"index": {"_index": "products", "_id": "2"}}
{"name": "MacBook Pro", "brand": "Apple", "price": 1999.99, "category": "Computers"}
{"index": {"_index": "products", "_id": "3"}}
{"name": "Dell XPS 13", "brand": "Dell", "price": 1299.99, "category": "Computers"}
{"update": {"_index": "products", "_id": "1"}}
{"doc": {"price": 849.99}}
{"delete": {"_index": "products", "_id": "old_product"}}
```

### Search Operations

#### Basic Search
```json
# Match all documents
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

# Simple match query
GET /products/_search
{
  "query": {
    "match": {
      "description": "smartphone features"
    }
  }
}

# Multi-field search
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "Apple iPhone",
      "fields": ["name^2", "brand", "description"],
      "type": "best_fields"
    }
  }
}
```

#### Advanced Queries
```json
# Bool query with multiple conditions
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"category": "Electronics"}}
      ],
      "filter": [
        {"range": {"price": {"gte": 500, "lte": 1500}}},
        {"terms": {"brand": ["Apple", "Samsung"]}}
      ],
      "should": [
        {"match": {"tags": "premium"}}
      ],
      "must_not": [
        {"match": {"description": "refurbished"}}
      ]
    }
  }
}

# Fuzzy search for typos
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "ipone",
        "fuzziness": "AUTO"
      }
    }
  }
}

# Wildcard and regex search
GET /products/_search
{
  "query": {
    "wildcard": {
      "name": "iPhone*"
    }
  }
}

# Nested query
GET /products/_search
{
  "query": {
    "nested": {
      "path": "specifications",
      "query": {
        "bool": {
          "must": [
            {"match": {"specifications.key": "storage"}},
            {"match": {"specifications.value": "256GB"}}
          ]
        }
      }
    }
  }
}
```

#### Search with Aggregations
```json
GET /products/_search
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category",
        "size": 10
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "price_ranges": {
          "range": {
            "field": "price",
            "ranges": [
              {"to": 500},
              {"from": 500, "to": 1000},
              {"from": 1000}
            ]
          }
        }
      }
    },
    "brands": {
      "terms": {
        "field": "brand",
        "size": 5
      }
    },
    "price_histogram": {
      "histogram": {
        "field": "price",
        "interval": 200
      }
    },
    "price_stats": {
      "stats": {
        "field": "price"
      }
    }
  }
}
```

#### Geospatial Queries
```json
# Geo distance query
GET /products/_search
{
  "query": {
    "geo_distance": {
      "distance": "10km",
      "location": {
        "lat": 40.7128,
        "lon": -74.0060
      }
    }
  }
}

# Geo bounding box query
GET /products/_search
{
  "query": {
    "geo_bounding_box": {
      "location": {
        "top_left": {
          "lat": 40.8,
          "lon": -74.1
        },
        "bottom_right": {
          "lat": 40.7,
          "lon": -74.0
        }
      }
    }
  }
}
```

### Search Features

#### Highlighting
```json
GET /products/_search
{
  "query": {
    "match": {
      "description": "smartphone"
    }
  },
  "highlight": {
    "fields": {
      "description": {
        "pre_tags": ["<strong>"],
        "post_tags": ["</strong>"],
        "fragment_size": 100,
        "number_of_fragments": 3
      }
    }
  }
}
```

#### Suggestions
```json
# Term suggestions
GET /products/_search
{
  "suggest": {
    "text": "smartfone",
    "simple_phrase": {
      "phrase": {
        "field": "description",
        "size": 1,
        "gram_size": 3,
        "direct_generator": [{
          "field": "description",
          "suggest_mode": "always"
        }]
      }
    }
  }
}

# Completion suggestions (requires mapping)
GET /products/_search
{
  "suggest": {
    "product_suggest": {
      "prefix": "iph",
      "completion": {
        "field": "suggest",
        "size": 5
      }
    }
  }
}
```

#### Pagination and Sorting
```json
# Basic pagination
GET /products/_search
{
  "from": 20,
  "size": 10,
  "sort": [
    {"price": {"order": "desc"}},
    {"_score": {"order": "desc"}}
  ]
}

# Search after (for deep pagination)
GET /products/_search
{
  "size": 10,
  "sort": [
    {"price": "desc"},
    {"_id": "asc"}
  ],
  "search_after": [999.99, "product_123"]
}
```

## Use Cases

### Log and Event Data Analysis

#### ELK Stack (Elasticsearch, Logstash, Kibana)
- **Centralized Logging**: Collect logs from multiple sources
- **Real-time Analysis**: Monitor application performance and errors
- **Security Analytics**: Detect suspicious activities and threats
- **Operational Intelligence**: Track system metrics and KPIs

**Example Log Mapping**:
```json
PUT /logs-2024.06
{
  "mappings": {
    "properties": {
      "@timestamp": {"type": "date"},
      "level": {"type": "keyword"},
      "logger": {"type": "keyword"},
      "message": {"type": "text"},
      "thread": {"type": "keyword"},
      "class": {"type": "keyword"},
      "method": {"type": "keyword"},
      "exception": {
        "properties": {
          "class": {"type": "keyword"},
          "message": {"type": "text"},
          "stacktrace": {"type": "text"}
        }
      },
      "user_id": {"type": "keyword"},
      "request_id": {"type": "keyword"},
      "duration_ms": {"type": "long"}
    }
  }
}
```

### Full-Text Search

#### E-commerce Product Search
```json
# Product search with faceting
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "wireless headphones",
            "fields": ["name^3", "description^2", "brand"],
            "type": "cross_fields"
          }
        }
      ],
      "filter": [
        {"terms": {"availability": ["in_stock"]}},
        {"range": {"price": {"gte": 50, "lte": 300}}}
      ]
    }
  },
  "aggs": {
    "brands": {
      "terms": {"field": "brand", "size": 10}
    },
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          {"key": "Under $100", "to": 100},
          {"key": "$100-$200", "from": 100, "to": 200},
          {"key": "Over $200", "from": 200}
        ]
      }
    },
    "ratings": {
      "terms": {"field": "rating", "size": 5}
    }
  },
  "sort": [
    {"_score": {"order": "desc"}},
    {"popularity": {"order": "desc"}}
  ]
}
```

#### Content Management System
```json
# Article search with content analysis
GET /articles/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"content": "machine learning"}}
      ],
      "filter": [
        {"term": {"status": "published"}},
        {"range": {"publish_date": {"gte": "2024-01-01"}}}
      ]
    }
  },
  "highlight": {
    "fields": {
      "content": {
        "fragment_size": 200,
        "number_of_fragments": 3
      }
    }
  },
  "aggs": {
    "categories": {
      "terms": {"field": "category", "size": 10}
    },
    "authors": {
      "terms": {"field": "author.keyword", "size": 5}
    },
    "monthly_posts": {
      "date_histogram": {
        "field": "publish_date",
        "calendar_interval": "month"
      }
    }
  }
}
```

### Real-Time Analytics

#### User Behavior Analysis
```json
# Track user events
POST /user_events/_doc
{
  "@timestamp": "2024-06-02T10:30:00Z",
  "user_id": "user123",
  "session_id": "session456",
  "event_type": "page_view",
  "page_url": "/products/electronics",
  "user_agent": "Mozilla/5.0...",
  "ip_address": "192.168.1.100",
  "referrer": "https://google.com",
  "duration_ms": 5000,
  "location": {
    "lat": 40.7128,
    "lon": -74.0060
  }
}

# Analytics query
GET /user_events/_search
{
  "size": 0,
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1d"
      }
    }
  },
  "aggs": {
    "events_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "hour"
      }
    },
    "top_pages": {
      "terms": {
        "field": "page_url.keyword",
        "size": 10
      }
    },
    "unique_users": {
      "cardinality": {
        "field": "user_id"
      }
    },
    "avg_session_duration": {
      "avg": {
        "field": "duration_ms"
      }
    }
  }
}
```

### Geospatial Applications

#### Location-Based Services
```json
# Store location data
PUT /locations/_doc/1
{
  "name": "Coffee Shop Downtown",
  "location": {
    "lat": 40.7128,
    "lon": -74.0060
  },
  "category": "restaurant",
  "rating": 4.5,
  "price_range": "$$"
}

# Find nearby locations
GET /locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {"category": "restaurant"}
      },
      "filter": {
        "geo_distance": {
          "distance": "1km",
          "location": {
            "lat": 40.7127,
            "lon": -74.0059
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat": 40.7127,
          "lon": -74.0059
        },
        "order": "asc",
        "unit": "m"
      }
    }
  ]
}
```

## Pros

### Performance
- **Near Real-time Search**: Documents are searchable within seconds of indexing
- **Horizontal Scaling**: Add nodes to increase capacity and performance
- **Efficient Full-text Search**: Optimized inverted indexes for fast text search
- **Fast Aggregations**: Real-time analytics on large datasets
- **Parallel Processing**: Distributed queries across multiple shards
- **Caching**: Multiple levels of caching for improved response times

### Flexibility
- **Schema-free**: Dynamic mapping allows flexible document structures
- **Multiple Data Types**: Support for text, numbers, dates, geo-points, nested objects
- **Rich Query DSL**: Powerful query language for complex search requirements
- **Custom Analyzers**: Configure text analysis for specific use cases
- **Multi-field Mapping**: Index same field in different ways

### Ecosystem and Integration
- **ELK Stack**: Seamless integration with Logstash and Kibana
- **RESTful API**: Easy integration with any programming language
- **Client Libraries**: Official clients for Java, Python, JavaScript, .NET, Go, PHP
- **Plugins**: Extensive plugin ecosystem for additional functionality
- **Cloud Services**: Available as managed services (Elastic Cloud, AWS Elasticsearch)

### Features
- **Auto-complete**: Built-in suggestion and completion capabilities
- **Faceted Search**: Multi-dimensional filtering and navigation
- **Highlighting**: Show search term matches in results
- **Geospatial Search**: Location-based queries and filtering
- **Percolator**: Reverse search - find queries that match documents
- **Machine Learning**: Anomaly detection and forecasting (X-Pack)

### Scalability
- **Automatic Sharding**: Distribute data across multiple nodes
- **Replica Management**: Automatic replica creation and management
- **Rolling Upgrades**: Update cluster without downtime
- **Index Lifecycle Management**: Automate index operations over time

## Cons

### Complexity
- **Steep Learning Curve**: Complex concepts like sharding, mapping, analyzers
- **Cluster Management**: Requires understanding of distributed systems
- **Configuration Complexity**: Many settings to optimize for specific use cases
- **Troubleshooting Difficulty**: Complex to diagnose performance and consistency issues
- **Operational Overhead**: Requires dedicated DevOps knowledge

### Consistency and Reliability
- **Near Real-time (Not Real-time)**: Small delay between indexing and searchability
- **Eventually Consistent**: No guarantee of immediate consistency across replicas
- **No ACID Transactions**: Cannot guarantee atomicity across multiple operations
- **Data Loss Risk**: Potential for data loss during node failures if not properly configured
- **Split-brain Scenarios**: Cluster can split into multiple masters without proper configuration

### Resource Requirements
- **Memory Intensive**: Requires significant RAM for optimal performance
- **CPU Intensive**: Complex queries and aggregations consume substantial CPU
- **Storage Overhead**: Inverted indexes require additional storage space
- **Network Bandwidth**: Cluster communication can be bandwidth-intensive
- **JVM Limitations**: Subject to Java garbage collection pauses

### Limitations and Constraints
- **Not Suitable for Transactional Data**: Poor choice for OLTP applications
- **Limited Relational Support**: No native joins or foreign key constraints
- **Expensive for Simple Operations**: Overkill for basic key-value lookups
- **Document Size Limits**: Maximum document size of 2GB
- **Field Mapping Explosion**: Too many fields can cause performance issues

### Cost Considerations
- **Resource Intensive**: High infrastructure costs for large deployments
- **Licensing Costs**: X-Pack features require commercial license
- **Operational Costs**: Requires skilled personnel for management
- **Scaling Costs**: Linear cost increase with data growth

## Best Practices

### Index Design
- **Use Time-based Indices**: For time-series data (logs-2024.06.01)
- **Appropriate Shard Count**: Start with 1 shard per 10-50GB of data
- **Replica Strategy**: At least 1 replica for production environments
- **Index Templates**: Use templates for consistent mapping across indices
- **Alias Management**: Use aliases for zero-downtime reindexing

### Mapping Optimization
```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "status": {
        "type": "keyword"
      },
      "timestamp": {
        "type": "date"
      },
      "content": {
        "type": "text",
        "index": false  // Don't index if not searching
      }
    }
  }
}
```

### Query Optimization
- **Use Filters**: Filters are cached and faster than queries
- **Limit Result Size**: Use `size` parameter to limit results
- **Use `_source` Filtering**: Return only needed fields
- **Profile Queries**: Use `_profile` API to analyze query performance
- **Avoid Wildcard Queries**: Especially leading wildcards

### Performance Tuning
- **Monitor Cluster Health**: Regular health checks and monitoring
- **Optimize JVM Settings**: Proper heap size configuration
- **Use SSD Storage**: For better I/O performance
- **Network Optimization**: Dedicated network for cluster communication
- **Index Refresh Settings**: Adjust refresh intervals based on use case

### Security Best Practices
- **Enable Security**: Use X-Pack Security or Open Distro Security
- **Network Security**: Firewall rules and VPN access
- **Authentication**: Strong password policies and multi-factor authentication
- **Authorization**: Role-based access control
- **Encryption**: TLS for data in transit, encryption at rest

### Monitoring and Alerting
```json
# Cluster health monitoring
GET /_cluster/health

# Node statistics
GET /_nodes/stats

# Index statistics
GET /my-index/_stats

# Search slow log
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s",
  "index.search.slowlog.threshold.fetch.warn": "1s"
}
```

## Common Patterns and Anti-patterns

### Good Patterns
- **Use bulk operations** for indexing multiple documents
- **Implement proper error handling** with retry mechanisms
- **Use index templates** for consistent mapping
- **Monitor cluster metrics** proactively
- **Design queries for caching** (use filters over queries when possible)

### Anti-patterns to Avoid
- **Don't use Elasticsearch as primary database** for transactional data
- **Avoid deep pagination** with from/size (use search_after instead)
- **Don't create too many small indices** (index overhead)
- **Avoid mapping explosion** (too many fields)
- **Don't ignore cluster health** warnings

## Tools and Ecosystem

### Core Stack
- **Elasticsearch**: Search and analytics engine
- **Logstash**: Data processing pipeline
- **Kibana**: Data visualization and management
- **Beats**: Lightweight data shippers

### Management Tools
- **Elasticsearch Head**: Web-based cluster management
- **Cerebro**: Elasticsearch web admin tool
- **ElasticHQ**: Monitoring and management interface
- **Curator**: Index lifecycle management

### Development Tools
- **Dev Tools (Kibana)**: Query console and debugging
- **Elasticsearch SQL**: SQL interface for Elasticsearch
- **Canvas**: Data visualization and presentation
- **Graph**: Relationship analysis and visualization

### Monitoring Solutions
- **X-Pack Monitoring**: Built-in cluster monitoring
- **Elastic APM**: Application performance monitoring
- **Metricbeat**: System and service metrics collection
- **Prometheus Integration**: For custom monitoring setups

## Production Considerations

### Capacity Planning
- **Estimate Index Size**: Plan for data growth over time
- **Calculate Resource Needs**: CPU, memory, storage, network
- **Plan for Peak Loads**: Handle traffic spikes and seasonal variations
- **Disaster Recovery**: Backup and restore strategies

### High Availability Setup
```yaml
# Example cluster configuration
cluster.name: production-cluster
node.name: node-1
node.roles: [master, data, ingest]
network.host: 0.0.0.0
discovery.seed_hosts: ["node-1", "node-2", "node-3"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

### Backup and Recovery
```bash
# Create snapshot repository
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups/my_backup"
  }
}

# Create snapshot
PUT /_snapshot/my_backup/snapshot_1
{
  "indices": "index_1,index_2",
  "ignore_unavailable": true,
  "include_global_state": false
}

# Restore snapshot
POST /_snapshot/my_backup/snapshot_1/_restore
{
  "indices": "index_1,index_2",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

This comprehensive guide covers Elasticsearch's architecture, usage patterns, pros and cons, and best practices for production deployment. The platform excels at full-text search, real-time analytics, and log analysis, but requires careful planning and expertise to operate effectively at scale.