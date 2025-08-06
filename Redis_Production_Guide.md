# Redis Production Setup Guide

This document provides a **comprehensive Redis production setup**, including memory management, eviction policies, persistence, security, monitoring, and deployment via Docker.

---

## Table of Contents

1. [Overview](#overview)
2. [Use Cases](#use-cases)
3. [Memory Management](#memory-management)
4. [Eviction Policies](#eviction-policies)
5. [Persistence Options](#persistence-options)
6. [Security Best Practices](#security-best-practices)
7. [Production Configuration (`redis.conf`)](#redisconf-production-settings)
8. [Docker Deployment](#docker-deployment)
9. [Monitoring Redis](#monitoring-redis)
10. [Scaling Considerations](#scaling-considerations)
11. [Tips & Best Practices](#tips--best-practices)

---

## Overview

**Redis** (Remote Dictionary Server) is an open-source, in-memory key-value store. It’s widely used for caching, session management, queues, pub/sub systems, and real-time analytics.

---

## Use Cases

* **Caching API responses**
* **Session store** for web apps
* **Rate limiting** using counters + TTL
* **Message queues** using Lists
* **Leaderboard ranking** via Sorted Sets
* **Pub/Sub** and **Streams** for eventing

---

## Memory Management

In production, Redis can **consume all available memory** if not limited. To avoid this:

```conf
maxmemory 4gb  # Or 75-80% of system RAM
```

---

## Eviction Policies

Choose a policy to decide what to evict **when memory is full**.

| Policy         | Behavior                             | Best For                     |
| -------------- | ------------------------------------ | ---------------------------- |
| `noeviction`   | Rejects writes, keeps all data       | Critical data                |
| `allkeys-lru`  | Removes least recently used key      | Full caching system          |
| `volatile-lru` | LRU eviction only for keys with TTL  | Mixed critical/cache data    |
| `volatile-ttl` | Removes TTL keys with nearest expiry | Session or TTL-based caching |

```conf
maxmemory-policy allkeys-lru
```

---

## Persistence Options

### AOF (Append Only File)

* Logs every write operation
* Safer for critical data
* Recommended: `appendfsync everysec`

### RDB (Snapshotting)

* Periodically saves the entire dataset
* Faster, smaller, but less durable

### Recommended Combined Setup:

```conf
appendonly yes
appendfsync everysec
save 900 1
save 300 10
save 60 10000
```

---

## Security Best Practices

* Set a strong password:

  ```conf
  requirepass "YourStrongPasswordHere"
  ```

* Bind to specific IP or use firewall:

  ```conf
  bind 127.0.0.1
  ```

* Enable protected mode:

  ```conf
  protected-mode yes
  ```

* Use TLS in production (Redis v6+)

---

## `redis.conf` Production Settings

```conf
# MEMORY

maxmemory 4gb                        # Max memory Redis can use
maxmemory-policy allkeys-lru        # Evict least recently used keys when memory is full

# PERSISTENCE


appendonly yes                      # Enable AOF persistence (logs every write)
appendfsync everysec                # Sync AOF to disk every second (good balance)
save 900 1                          # RDB snapshot if 1 change in 900 sec
save 300 10                         # RDB snapshot if 10 changes in 300 sec
save 60 10000                       # RDB snapshot if 10K changes in 60 sec

# SECURITY

requirepass "YourStrongPasswordHere" # Password required to connect to Redis
protected-mode yes                   # Protect from unsafe access (enabled by default)

# NETWORK

bind 0.0.0.0                         # Listen on all interfaces (secure with firewall)
port 6379                            # Redis default port
tcp-keepalive 60                     # Keep TCP connection alive (prevents stale clients)
timeout 0                            # Disable idle client timeout

# LOGGING

logfile "/var/log/redis/redis-server.log"  # Path to Redis log file
loglevel notice                             # Log level (debug | verbose | notice | warning)

# OTHER

dynamic-hz yes                      # Auto adjust internal housekeeping frequency
no-appendfsync-on-rewrite no        # Ensure AOF is synced during background rewrite too
```

---

## Docker Deployment

### `docker-compose.yml`

```yaml
version: "3"

services:
  redis:
    image: redis:7.2
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - redis-data:/data
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    restart: always

volumes:
  redis-data:
```

### Folder Structure

```
redis-prod/
├── redis.conf
└── docker-compose.yml
```

---

## Monitoring Redis

### Prometheus Exporter

```bash
docker run -d -p 9121:9121 \
  oliver006/redis_exporter \
  --redis.addr=redis://:<password>@localhost:6379
```

### Metrics to Monitor

| Metric                  | Description                  |
| ----------------------- | ---------------------------- |
| `used_memory`           | RAM used                     |
| `connected_clients`     | Number of clients connected  |
| `evicted_keys`          | Keys removed due to eviction |
| `keyspace_hits/misses`  | Cache efficiency             |
| `aof_last_write_status` | AOF persistence health       |

Use **Grafana** + Prometheus for alerting dashboards.

---

## Scaling Considerations

* Use **Redis Sentinel** for automatic failover
* Use **Redis Cluster** for sharding large datasets
* Use **ElastiCache/Cloud Redis** for fully managed setup

---

## Tips & Best Practices

* Don’t mix critical and cache data in the same instance
* Enable `tcp-keepalive` for stability
* Always monitor memory and eviction stats
* Design keys with TTL if eviction is enabled
* Use Redis namespaces (`user:1234`, `session:xyz`) for logical grouping
* Use ACLs (Redis 6+) to restrict command access

---

## Conclusion

Redis is powerful and fast, but production setups require:

* Memory limits and eviction tuning
* Strong security practices
* Proper persistence (or explicit cache-only mode)
* Monitoring and failover planning
