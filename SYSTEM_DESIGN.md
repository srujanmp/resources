# SYSTEM DESIGN COMPLETE GUIDE
### Scratch to Advanced · Interview + Real-World Engineering
> Covers FAANG / Top-Tier Technical Interviews · Real-World Distributed Systems Engineering · Beginner-Friendly with Deep Dives

---

# Chapter 1: What Is System Design?

System design is the process of defining the architecture, components, modules, interfaces, and data flow of a system to satisfy specified requirements. In simpler terms: given a problem like "Build Twitter" or "Design a URL shortener", how do you structure the entire software system behind it?

> 💡 **Analogy:** Think of it like designing a city. You don't just build one big building — you plan roads (network), power grids (infrastructure), neighborhoods (services), and communication systems (APIs) that work together seamlessly at massive scale.

---

## Why System Design Matters

- It's a core part of senior engineering interviews at Google, Meta, Amazon, Apple, Netflix, and every major tech company
- Real software serves millions of users — design decisions directly impact cost, reliability, and speed
- Bad design leads to outages, data loss, and poor user experience
- Good design is what separates a senior engineer from a junior one

---

## The Two Pillars of System Design

### 🏗️ Architecture & Components
- How to split your system into pieces
- What each piece does
- How pieces communicate
- Where data lives and how it flows

### 📈 Scalability & Reliability
- How the system handles 1M users vs 10 users
- What happens when a server crashes
- How fast the system responds
- How to keep data consistent

---

## Key Vocabulary You'll Use Constantly

| Term | Plain English Meaning | Example |
|---|---|---|
| Scalability | System can handle growing load | Goes from 100 to 10M users smoothly |
| Latency | Time to complete one request | Your API responds in 50ms |
| Throughput | Requests handled per second | Server handles 10,000 req/sec |
| Availability | System uptime percentage | 99.99% = ~52 min downtime/year |
| Consistency | All nodes see same data | Read after write returns latest data |
| Partition Tolerance | Works despite network failures | Two DCs stay up if link breaks |
| Fault Tolerance | Handles component failures | One server dies, others take over |
| Redundancy | Duplicate components for safety | 2 databases, 3 app servers |

---

# Chapter 2: Networking Fundamentals

Before designing systems, you need to understand how computers talk to each other over a network. Everything in distributed systems is built on these basics.

---

## The OSI Model (Simplified)

> ⚡ **Note:** Don't memorize all 7 layers — focus on the ones that matter for system design: Transport (TCP/UDP) and Application (HTTP, DNS, etc.).

| Layer | What It Does | Protocols / Examples |
|---|---|---|
| 7 - Application | What the user/app sends | HTTP, HTTPS, DNS, FTP, SMTP |
| 4 - Transport | Reliable delivery of data | TCP (reliable), UDP (fast, unreliable) |
| 3 - Network | Routing between machines | IP addresses, routers |
| 2 - Data Link | Node-to-node communication | MAC addresses, switches |
| 1 - Physical | Actual bits on wire/air | Ethernet cables, WiFi signals |

---

## TCP vs UDP — Know This Cold

### TCP — Reliable, Ordered
- 3-way handshake (SYN, SYN-ACK, ACK)
- Guarantees delivery and order
- Retransmits lost packets
- Higher overhead, ~20ms extra latency
- *Use when: banking, file transfer, web pages*

### UDP — Fast, Unreliable
- No connection setup
- Fire-and-forget — no delivery guarantee
- Packets may arrive out of order or not at all
- Very low latency overhead
- *Use when: live video, gaming, DNS lookups*

---

## HTTP / HTTPS — The Language of the Web

HTTP (HyperText Transfer Protocol) is how browsers and servers talk. HTTPS is HTTP with TLS encryption. Most system design APIs use HTTP/HTTPS.

| HTTP Method | Purpose | Idempotent? | Use Case |
|---|---|---|---|
| GET | Read a resource | Yes | Fetch user profile |
| POST | Create a resource | No | Create new order |
| PUT | Replace entire resource | Yes | Update full user object |
| PATCH | Partially update resource | No | Change only email field |
| DELETE | Remove a resource | Yes | Delete a tweet |

---

## DNS — The Internet's Phone Book

DNS (Domain Name System) translates human-readable domain names (google.com) into IP addresses (142.250.80.46) that computers use to connect.

> 📖 **DNS resolution flow:** Browser → Local cache → ISP resolver → Root server → TLD server (.com) → Authoritative server → IP address returned. This takes milliseconds and is cached at every step.

---

## Load Balancers — Traffic Cops at Scale

A load balancer sits in front of your servers and distributes incoming requests across multiple backend servers. It's one of the first things you add when scaling beyond a single server.

| Algorithm | How It Works | Best For |
|---|---|---|
| Round Robin | Send request 1 to server 1, request 2 to server 2, cycle... | Equally powerful servers |
| Least Connections | Send to the server with fewest active connections | Long-running requests (WebSockets) |
| IP Hash | Route same client IP to same server every time | Session persistence (stateful apps) |
| Weighted Round Robin | More powerful servers get proportionally more traffic | Mixed hardware capacity |
| Random | Pick a server at random | Simple, often good enough |

> 💡 **Layer 4 vs Layer 7 Load Balancing:** L4 routes based on TCP/IP (fast but dumb). L7 routes based on HTTP content like URL paths, headers, cookies (slower but smart — can route /api to one fleet and /static to another).

---

# Chapter 3: Databases — Storing and Retrieving Data

Your choice of database is one of the most critical system design decisions. The wrong choice can make your system unmaintainable, slow, or impossible to scale.

---

## Relational (SQL) Databases

Store data in tables with rows and columns. Data has a strict schema. Use SQL to query. Tables relate to each other via foreign keys. ACID transactions guarantee correctness.

> 🔒 **ACID:** Atomicity (all or nothing), Consistency (always valid state), Isolation (transactions don't interfere), Durability (committed data survives crashes). This is the gold standard for correctness.

| Database | Known For | Best Use Cases |
|---|---|---|
| PostgreSQL | Full-featured, JSONB support, open-source | Most applications — strong default choice |
| MySQL | Widely deployed, fast reads | Web apps, simple transactional workloads |
| SQLite | Serverless, single file | Embedded apps, local dev, mobile |
| Aurora (AWS) | MySQL/Postgres-compatible, managed, auto-scales | High-traffic cloud apps needing managed ops |

---

## NoSQL Databases — When SQL Isn't Enough

NoSQL databases sacrifice some ACID guarantees for horizontal scalability, flexible schemas, or specialized data access patterns. The name is misleading — it really means "not only SQL".

| Type | Structure | Databases | Use When |
|---|---|---|---|
| Document | JSON/BSON documents | MongoDB, Firestore, CouchDB | Flexible schemas, nested objects, catalogs |
| Key-Value | Hash map: key → value | Redis, DynamoDB, Memcached | Caching, sessions, shopping carts, counters |
| Wide-Column | Tables with dynamic columns | Cassandra, HBase, Bigtable | Time-series, IoT, write-heavy, massive scale |
| Graph | Nodes and edges | Neo4j, Amazon Neptune | Social networks, recommendations, fraud detection |
| Time-Series | Timestamped data points | InfluxDB, TimescaleDB, Prometheus | Metrics, monitoring, sensor data |
| Search | Inverted index | Elasticsearch, OpenSearch, Solr | Full-text search, log analytics |

---

## SQL vs NoSQL — The Decision Framework

### ✅ Choose SQL when:
- Data has clear relationships (joins needed)
- ACID transactions are required (finance, inventory)
- Schema is well-defined and stable
- Team has SQL expertise
- Data volume is manageable (< few TB)
- You need complex queries and reporting

### ✅ Choose NoSQL when:
- Schema is dynamic or evolving fast
- Extreme horizontal scalability needed
- Very high write throughput (logs, events, IoT)
- Data is denormalized and self-contained
- Access pattern is simple (get by key)
- Geographic distribution required

---

## Database Scaling Strategies

### Vertical Scaling (Scale Up)
Add more CPU, RAM, or storage to your existing database server. Simplest solution — but has a ceiling, and expensive hardware has diminishing returns.

### Read Replicas
Create copies of your database (replicas) that serve read traffic. Writes still go to the primary (master) database, which replicates data to replicas. Most apps are read-heavy (80%+ reads), so this dramatically reduces load on the primary.

> ⚠️ **Trade-off:** Replication lag. Data written to the primary takes milliseconds to appear on replicas. If a user writes data and immediately reads it from a replica, they might get stale data.

### Sharding (Horizontal Partitioning)
Split your data across multiple database servers (shards). Each shard holds a subset of the data. For example, users with ID 1-1M go to shard 1, users 1M-2M go to shard 2.

| Sharding Strategy | How It Works | Pros | Cons |
|---|---|---|---|
| Range-based | Shard by value range (A-M, N-Z) | Simple, easy range queries | Hotspots if data isn't uniform |
| Hash-based | Hash(key) % num_shards | Even distribution | Range queries span shards |
| Geographic | Shard by region (US, EU, Asia) | Low latency for local users | Cross-region queries are slow |
| Directory-based | Lookup table maps key → shard | Most flexible | Lookup table is a bottleneck/SPOF |

> 🚨 **Warning:** Sharding adds enormous complexity: cross-shard transactions, joins, and re-sharding when you grow. Exhaust all other scaling options (caching, read replicas, better queries) before sharding.

---

## Indexing — Making Queries Fast

An index is a data structure (usually a B-Tree or hash) that allows the database to find rows without scanning the entire table. Think of it like a book's index vs reading the entire book.

- **Primary index:** automatically created on the primary key
- **Secondary index:** manually created on other frequently-queried columns
- **Composite index:** covers multiple columns — order matters! `(col_a, col_b)` helps queries on `col_a` or `(col_a AND col_b)` but NOT `col_b` alone
- **Covering index:** includes all columns needed by a query, so the DB never touches the main table

> 💡 **The Rule of Thumb:** Index columns used in WHERE, JOIN, and ORDER BY clauses. Every index speeds up reads but slows down writes (index must be updated). Don't index everything — be surgical.

---

## CAP Theorem — The Fundamental Trade-off

CAP Theorem states that a distributed system can only guarantee 2 of these 3 properties at any time:

| Property | What It Means | Example Violation |
|---|---|---|
| Consistency (C) | Every read gets the most recent write | Two nodes show different data |
| Availability (A) | Every request gets a response (may be stale) | System returns error during partition |
| Partition Tolerance (P) | System works despite network partitions | Network split causes data divergence |

> 🚨 **Critical:** Network partitions WILL happen in any real distributed system. So you must choose between CP (consistent but may be unavailable) or AP (available but may show stale data). You cannot avoid this choice.

| System Type | Guarantees | Example Databases |
|---|---|---|
| CP | Consistent + Partition Tolerant. May refuse requests. | HBase, Zookeeper, etcd, MongoDB (configurable) |
| AP | Available + Partition Tolerant. May return stale data. | Cassandra, CouchDB, DynamoDB (eventually consistent) |
| CA (not possible) | Consistent + Available. Assumes no partitions. | Only single-node systems — not truly distributed |

---

# Chapter 4: Caching — Speed at Scale

Caching stores frequently accessed data in fast, temporary storage (usually RAM) so future requests can be served much faster. It's the single most impactful optimization in system design.

> ⚡ **The cache hierarchy from fastest to slowest:** L1 CPU cache (nanoseconds) → L2 cache → L3 cache → RAM (microseconds) → SSD (~100µs) → HDD (~10ms) → Network (~100ms). Caching at any level is a win.

---

## Where to Cache

| Cache Layer | Where | Speed | Examples |
|---|---|---|---|
| Client-side | Browser / mobile device | Instant | Browser cache, localStorage, ServiceWorker |
| CDN | Edge servers worldwide | ~10ms (nearest PoP) | CloudFront, Fastly, Cloudflare |
| Reverse Proxy | In front of your servers | ~1ms (same DC) | Nginx, Varnish, Squid |
| Application | In your server's memory | Microseconds | In-process cache, Spring Cache |
| Database Query Cache | Inside the DB engine | ~1ms | MySQL query cache (deprecated) |
| Distributed Cache | Shared cache cluster | ~1-5ms (network hop) | Redis, Memcached |

---

## Cache Eviction Policies

When a cache is full and a new item needs to be stored, the cache must decide what to remove (evict). The policy determines this.

| Policy | What It Removes | Best For |
|---|---|---|
| LRU (Least Recently Used) | Item not accessed for the longest time | General purpose — most common choice |
| LFU (Least Frequently Used) | Item accessed fewest times overall | Long-lived caches with popularity skew |
| FIFO (First In, First Out) | Oldest item (inserted first) | Simple cases; not great for hot data |
| TTL (Time To Live) | Items older than a set expiry time | Data that goes stale after a fixed time |
| MRU (Most Recently Used) | Most recently used item | Rarely useful — niche cases |

---

## Cache Reading Strategies

### Cache-Aside (Lazy Loading)
Application checks cache first. On miss, reads from DB and populates cache. Most common pattern.

1. Request arrives — check cache
2. Cache HIT → return immediately
3. Cache MISS → fetch from DB → write to cache → return

> 💡 **Pros:** Only requested data is cached (no wasted memory). Cache failure doesn't break the app. **Cons:** First request is always slow (cold start). Thundering herd problem on cache miss under load.

### Read-Through Cache
Cache sits between application and DB. On miss, the cache itself fetches from DB. Application always talks to cache only.

> 💡 **Pros:** Simpler application code — always just query cache. **Cons:** Cache and DB must be tightly integrated. Still slow on first request.

---

## Cache Writing Strategies

| Strategy | How It Works | Pros | Cons |
|---|---|---|---|
| Write-Through | Write to cache AND DB synchronously | No stale data, consistent | Higher write latency |
| Write-Back (Write-Behind) | Write to cache only; async flush to DB | Very fast writes, batching possible | Risk of data loss if cache dies |
| Write-Around | Write directly to DB, bypassing cache | No cache pollution for rarely-read data | First read is always a miss |

---

## Cache Invalidation — The Hard Problem

> "There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton

How do you keep cache in sync with the source of truth?

- **TTL-based expiry:** Cache items expire after N seconds. Simple but may serve stale data up to N seconds
- **Event-driven invalidation:** When DB changes, publish event → cache subscriber deletes or updates the key. More complex but more accurate
- **Write-through:** Update cache on every write. Simple but every write has double the latency
- **Cache-busting:** Change the cache key when data changes (v1_user_123 → v2_user_123)

> 🚨 **The Cache Stampede Problem:** When a cached item expires, many simultaneous requests all miss and all hit the database at once. Solutions: probabilistic early expiration, request coalescing (mutex/lock per key), background refresh before expiry.

---

## Redis — The Swiss Army Knife of Caching

Redis (Remote Dictionary Server) is the most popular distributed cache. It stores data in RAM and supports rich data structures beyond simple key-value pairs.

| Data Structure | Operations | System Design Use Cases |
|---|---|---|
| String | GET, SET, INCR, DECR | Caching HTML fragments, rate limiting counters |
| Hash | HGET, HSET, HGETALL | User sessions, object caching |
| List | LPUSH, RPUSH, LRANGE | Activity feeds, queues (simple use cases) |
| Set | SADD, SMEMBERS, SINTERSTORE | Unique visitors, tags, friend lists |
| Sorted Set | ZADD, ZRANGE, ZRANGEBYSCORE | Leaderboards, time-series, priority queues |
| Pub/Sub | PUBLISH, SUBSCRIBE | Real-time notifications, fan-out messaging |
| Streams | XADD, XREAD, XGROUP | Event sourcing, message queues (durable) |

---

# Chapter 5: APIs — The Contracts Between Services

An API (Application Programming Interface) defines how components of a system communicate. In system design, APIs are the contracts that services expose to each other and to clients.

---

## REST APIs — The Industry Standard

REST (Representational State Transfer) is an architectural style for designing web APIs using HTTP. It's the most common API style in production systems.

**REST Constraints:**
- **Stateless:** Server stores no session state between requests — client sends everything needed
- **Uniform Interface:** Resources are identified by URLs; standard HTTP methods for operations
- **Client-Server Separation:** Frontend and backend are independent, evolve separately
- **Cacheable:** Responses explicitly marked as cacheable or not
- **Layered System:** Client can't tell if it's talking to the real server or a proxy

**REST API Design Best Practices:**
- Use nouns for resources, not verbs: `/users` (good) vs `/getUser` (bad)
- Use HTTP methods to express actions: `GET /users` (list), `POST /users` (create), `GET /users/123` (get), `PUT /users/123` (update), `DELETE /users/123` (delete)
- Use HTTP status codes correctly: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 500 Server Error
- Version your API: `/v1/users`, `/v2/users` — allows non-breaking changes
- Use pagination for list endpoints: `?page=2&limit=50` or cursor-based

---

## GraphQL — Query What You Need

GraphQL is a query language for APIs where the client specifies exactly what data it wants. Instead of multiple REST endpoints, there's a single endpoint where you send a query document.

### ✅ GraphQL Advantages
- No over-fetching (get exactly what you need)
- No under-fetching (one request for nested data)
- Strong typing with schema
- Self-documenting via introspection
- Great for complex, nested data (social graphs)

### ⚠️ GraphQL Disadvantages
- Complex caching (no URL-based cache keys)
- N+1 query problem (mitigated by DataLoader)
- Harder rate limiting per query complexity
- Steeper learning curve vs REST
- Overkill for simple CRUD APIs

---

## gRPC — High-Performance Service Communication

gRPC is Google's RPC framework using Protocol Buffers (protobuf) for serialization and HTTP/2 for transport. It's significantly faster than REST+JSON for internal service-to-service communication.

| Feature | REST + JSON | gRPC + Protobuf |
|---|---|---|
| Protocol | HTTP/1.1 (usually) | HTTP/2 |
| Data Format | JSON (text, human-readable) | Protobuf (binary, compact) |
| Performance | Baseline | ~5-10x faster serialization |
| Streaming | WebSockets or SSE (add-on) | Built-in bidirectional streaming |
| Type Safety | Optional (OpenAPI/Swagger) | Enforced by schema |
| Browser Support | Native | Needs grpc-web proxy |
| Best For | External-facing APIs, simple services | Internal microservice communication |

---

## WebSockets — Real-Time Bidirectional Communication

HTTP is request-response: client asks, server answers. WebSockets establish a persistent, bidirectional connection where either side can send messages anytime.

> 💡 **WebSocket handshake** starts as an HTTP request with an Upgrade header. Once the server accepts, the connection upgrades to a WebSocket. From this point, both sides can push data freely without waiting for a request.

- **Use cases:** live chat, multiplayer games, collaborative editors (Google Docs), stock tickers, live sports scores, real-time notifications
- **Challenge:** WebSocket connections are stateful — a user is "stuck" to one server. Need sticky sessions (IP hash load balancing) or a shared message broker (Redis Pub/Sub)
- **At scale:** A single server can handle ~65,000 concurrent WebSocket connections (one per port). With 10M users, you need ~150+ servers just for WebSocket handling

---

## API Gateway — The Front Door

An API gateway is a server that acts as the single entry point for all client requests. It routes requests to appropriate microservices, handles cross-cutting concerns, and shields internal architecture from clients.

| Responsibility | What It Does |
|---|---|
| Request Routing | Routes /users to user-service, /orders to order-service |
| Authentication | Validates JWT tokens, API keys — services don't need to |
| Rate Limiting | Caps requests per user/IP to prevent abuse |
| SSL Termination | Handles HTTPS; downstream services use plain HTTP |
| Request/Response Transform | Protocol translation, payload transformation |
| Circuit Breaking | Stops calling a failing downstream service |
| Caching | Caches common responses at the edge |
| Observability | Centralized logging, tracing, metrics collection |

---

# Chapter 6: Message Queues & Event-Driven Architecture

Message queues allow services to communicate asynchronously — instead of waiting for an immediate response, a service puts a message in a queue and the consumer processes it when ready. This is foundational for scalable, resilient systems.

---

## Why Message Queues?

### The Problem (Synchronous)
- User uploads video → service immediately transcodes
- Transcoding takes 5 minutes
- HTTP request times out (typically 30s-2min)
- If transcoder crashes, video is lost
- Spike in uploads overwhelms transcoder

### The Solution (Async Queue)
- User uploads video → message put in queue (instant)
- User gets "202 Accepted, processing..."
- Transcoder picks up jobs at its own pace
- If transcoder crashes, message stays in queue
- Scale up consumers for spikes

---

## Key Concepts

| Concept | Meaning |
|---|---|
| Producer | Service that creates and sends messages to the queue |
| Consumer | Service that reads and processes messages from the queue |
| Queue | Ordered buffer holding messages until consumed (point-to-point) |
| Topic / Exchange | Named channel — many consumers can subscribe (pub/sub fan-out) |
| ACK (Acknowledgment) | Consumer signals message was processed; queue deletes it |
| Dead Letter Queue (DLQ) | Where messages go after failing to process N times |
| At-Least-Once Delivery | Message delivered at least once (may be duplicate) |
| Exactly-Once Delivery | No duplicates, no loss — hardest to guarantee |

---

## Apache Kafka — The Event Streaming Platform

Kafka is the dominant solution for high-throughput event streaming. Unlike traditional queues (message deleted after consumption), Kafka retains messages for a configurable retention period, allowing multiple consumers to replay the same stream independently.

**Kafka Core Concepts:**
- **Topic:** Named stream of records (like a database table in a queue)
- **Partition:** A topic is split into ordered partitions — enables parallelism. Messages in a partition are strictly ordered.
- **Offset:** Position of a message within a partition. Consumers track their offset.
- **Consumer Group:** Multiple consumers sharing a topic's partitions. Each partition is read by one consumer in the group.
- **Broker:** A Kafka server. Multiple brokers form a cluster.
- **Replication Factor:** Each partition is replicated to N brokers for fault tolerance.

> ⚡ **Kafka's superpower:** Consumers track their own offset. A new analytics service can replay the entire event history from offset 0. You can add consumers without affecting producers or other consumers — total decoupling.

| Tool | Type | Throughput | Retention | Use When |
|---|---|---|---|---|
| Kafka | Log/stream | Millions/sec | Days to forever | Event sourcing, audit logs, analytics pipelines, high throughput |
| RabbitMQ | Message broker | Thousands/sec | Until consumed | Task queues, work distribution, complex routing |
| AWS SQS | Managed queue | High | 14 days max | AWS-native apps, simple job queues, auto-scaling workers |
| AWS SNS | Pub/Sub | High | No storage | Fan-out notifications, trigger multiple subscribers |
| Redis Streams | In-memory stream | Very high | Configurable | Low-latency streaming, real-time apps, small scale |

---

## Event-Driven Architecture Patterns

### Publish-Subscribe (Pub/Sub)
Producers publish events to a topic without knowing who (if anyone) is listening. Subscribers receive all events on a topic they've subscribed to. Total decoupling between producers and consumers.

> 💡 **Example:** "OrderPlaced" event published → inventory service decrements stock, email service sends confirmation, analytics service records sale. All independently, without the order service knowing about them.

### Event Sourcing
Instead of storing the current state of an entity, store the sequence of events that led to it. The current state is derived by replaying all events.

- **Example:** Bank account balance = sum of all debit/credit events since account creation
- **Benefits:** Complete audit trail, ability to replay, temporal queries ("what was balance on Jan 1?"), easy debugging
- **Drawbacks:** Event store grows forever, replaying many events is slow (mitigated by snapshots)

### CQRS (Command Query Responsibility Segregation)
Separate read and write paths of an application. Commands (writes) update the write model. Queries (reads) use a separate, optimized read model. Often combined with Event Sourcing.

> 💡 CQRS lets you optimize reads and writes independently. Your write database can be fully normalized PostgreSQL for consistency. Your read database can be denormalized Elasticsearch for fast full-text search. They stay in sync via events.

---

# Chapter 7: Microservices vs Monoliths

One of the most debated architectural decisions: should you build one big application (monolith) or many small services (microservices)? The honest answer: it depends, and monoliths are more often the right start.

---

## The Monolith

A monolith is a single deployable unit where all functionality lives in one codebase. A request flows through one process rather than multiple network hops.

### ✅ Monolith Strengths
- Simple to develop, test, and deploy
- No network latency between components
- Easy transactions (in-process ACID)
- Simpler debugging (one log stream, one stack trace)
- Good for small teams (< 15 engineers)
- Fast iteration early in product lifecycle

### ⚠️ Monolith Weaknesses
- Entire app re-deploys for small changes
- One bug can crash everything
- Hard to scale specific parts independently
- Tech stack locked — all in one language/framework
- Large codebase becomes slow to build/test
- Tight coupling makes changes risky

---

## Microservices Architecture

Microservices split an application into small, independently deployable services, each responsible for a single business domain. Each service has its own database (the database-per-service pattern).

### ✅ Microservices Strengths
- Independent deployment — change one, deploy one
- Scale each service based on its load
- Independent tech stacks per service
- Fault isolation — one service fails, others don't
- Smaller codebases — easier for teams to own
- Clear domain boundaries (Conway's Law)

### ⚠️ Microservices Weaknesses
- Network calls instead of function calls (latency + failure)
- Distributed transactions are very hard
- Service discovery, load balancing complexity
- Debugging across services is painful
- Much more infrastructure to manage
- Premature microservices are the #1 architecture mistake

> 💡 **Martin Fowler's Rule:** "Don't start with microservices. Start with a monolith, understand the domain boundaries well, then extract services only when you have a clear scaling or team independence need." Netflix, Amazon, Uber all started as monoliths.

---

## Service Communication in Microservices

| Pattern | How | Use When |
|---|---|---|
| Synchronous REST/gRPC | Direct HTTP call — waits for response | When you need an immediate answer (user auth check) |
| Async Messaging | Put message in queue, proceed | When you don't need immediate response (send email) |
| Event-Driven | Publish event; others react | Loose coupling, multiple consumers, audit trail |
| Service Mesh (Envoy/Istio) | Sidecar proxy handles all network concerns | Large-scale, need observability + traffic control |

---

## Service Discovery

With many services, how does Service A know where Service B is? Service Discovery solves this.

- **Client-side discovery:** Service queries a service registry (Consul, Eureka) to find healthy instances, then calls directly
- **Server-side discovery:** Service sends request to load balancer (AWS ALB, Kubernetes Service), which queries the registry and routes
- **DNS-based:** Services register with DNS; caller just uses service name (Kubernetes does this natively)

---

## The Saga Pattern — Distributed Transactions

Traditional ACID transactions don't work across microservices (each service has its own DB). The Saga pattern coordinates a sequence of local transactions, with compensating transactions to undo steps if something fails.

> 💡 **Example:** Book hotel (saga step 1) → Book flight (step 2) → Charge credit card (step 3). If step 3 fails, compensating transactions cancel the flight booking and hotel reservation.

| Saga Style | How It Works | Pros | Cons |
|---|---|---|---|
| Choreography | Each service publishes events; next service reacts | No central coordinator, decoupled | Hard to trace/debug, complex failure logic |
| Orchestration | Central orchestrator (workflow engine) directs steps | Easy to visualize, clear control flow | Orchestrator is a bottleneck/SPOF risk |

---

# Chapter 8: Scalability Patterns

Scalability is the ability of a system to handle growing load gracefully. This chapter covers the patterns every system designer must know to make systems that scale from thousands to millions of requests.

---

## The 3 Dimensions of Scaling (AKF Scale Cube)

| Axis | Name | Technique | Example |
|---|---|---|---|
| X-Axis | Horizontal Duplication | Run N identical copies behind a load balancer | 10 app servers behind an ALB |
| Y-Axis | Functional Decomposition | Split by domain/function (microservices) | User service, Order service, Payment service |
| Z-Axis | Data Partitioning | Each instance handles a subset of data (sharding) | User IDs 0-1M on server A, 1M-2M on server B |

---

## Rate Limiting

Rate limiting caps how many requests a client can make in a given time window. This prevents abuse, DDoS attacks, and ensures fair usage.

| Algorithm | How It Works | Pros | Cons |
|---|---|---|---|
| Fixed Window Counter | Count requests per minute window; reset at boundary | Simple, memory efficient | Burst allowed at window edge (2x burst) |
| Sliding Window Log | Record timestamp of each request; count within window | Accurate | High memory (stores all timestamps) |
| Sliding Window Counter | Weighted sum of current + previous window counts | Memory efficient, smooth | Slightly approximate |
| Token Bucket | Bucket fills with tokens at rate R; each request consumes 1 | Allows controlled bursting | Harder to implement correctly |
| Leaky Bucket | Queue holds requests; processes at fixed rate | Smooth output rate | Drops requests under burst |

> 💡 **Where to rate limit:** API Gateway (per client/IP/API key), Application level (per user), Database level (connection pool limits). Store counters in Redis for distributed rate limiting — it's atomic with INCR and EXPIRE commands.

---

## Content Delivery Networks (CDN)

A CDN is a globally distributed network of servers (Points of Presence — PoPs) that cache static content close to users. Instead of every request traveling to your origin server, users get content from a nearby edge node.

- Static assets: images, CSS, JS, fonts, videos → always use a CDN
- Dynamic content: some CDNs can cache API responses with smart invalidation
- Edge compute: CDNs like Cloudflare Workers, Lambda@Edge can run code at the edge
- DDoS protection: CDN absorbs attack traffic before it hits your origin

| CDN Provider | Known For | Best For |
|---|---|---|
| Cloudflare | DDoS protection, Workers (edge compute), free tier | Security + performance for most apps |
| AWS CloudFront | Deep AWS integration, Lambda@Edge | AWS-native architectures |
| Fastly | Instant purging, streaming, VCL programmability | Media companies, large-scale streaming |
| Akamai | Largest PoP network globally, enterprise reliability | Enterprise, high-stakes global delivery |

---

## Consistent Hashing

Consistent hashing solves the problem of distributing data evenly across servers in a way that minimizes redistribution when servers are added or removed.

**The problem with naive hashing:** With 3 servers, key maps to `server_index = hash(key) % 3`. If you add a 4th server, almost every key remaps — massive cache misses and DB load.

**Consistent hashing solution:** Arrange servers on a "ring" (0 to 2^32). Hash each server to a point on the ring. For each key, hash it and travel clockwise to find the next server. Adding/removing a server only affects keys between it and the previous server (1/N of all keys).

> 💡 **Virtual nodes:** To avoid uneven distribution, each physical server is mapped to multiple virtual nodes on the ring (e.g., 150 virtual nodes per server). This ensures better load distribution even with heterogeneous hardware.

**Used in:** Cassandra (data partitioning), DynamoDB (storage nodes), Memcached (ketama), load balancers (consistent IP routing), distributed caches

---

## Circuit Breaker Pattern

When Service A calls Service B which is failing or slow, Service A shouldn't keep trying indefinitely — it will exhaust its own resources waiting. The Circuit Breaker prevents this.

| State | Behavior | Transition |
|---|---|---|
| Closed (Normal) | Requests flow through normally; failures tracked | Too many failures → Open |
| Open (Broken) | Requests immediately fail with error; B is not called | After timeout → Half-Open |
| Half-Open (Testing) | Let a few requests through to test if B recovered | Success → Closed; Failure → Open |

> 💡 The Circuit Breaker gives failing services time to recover rather than being hammered with traffic while down. Combine with retries (with exponential backoff + jitter) for resilient service communication. Libraries: Hystrix (Netflix), Resilience4j, Polly (.NET).

---

## Bulkhead Pattern

Inspired by ship bulkheads (watertight compartments), this pattern isolates components so failures don't cascade. Allocate separate resource pools (thread pools, connection pools, memory) per downstream service.

> 🚨 **Example:** Without bulkheads, a slow payment service exhausts all 200 threads in your app server, making the entire app unresponsive — even /health and /login. With bulkheads, payment service gets 30 threads, order service gets 30 threads — other features stay alive.

---

# Chapter 9: Storage Systems — Beyond Databases

Modern systems use multiple storage types beyond traditional databases. Understanding when to use each type is critical for designing cost-effective, performant systems.

---

## Object Storage

Object storage stores unstructured data as "blobs" (binary large objects) — files, images, videos, backups. Objects are stored flat (no folder hierarchy), identified by a key, accessible via HTTP.

- AWS S3, Google Cloud Storage, Azure Blob Storage are the dominant offerings
- Essentially infinite capacity, pays per GB stored and per GB transferred
- High durability (S3 guarantees 99.999999999% — eleven nines — durability)
- Not a filesystem — no in-place updates. Must read-modify-write or use versioning
- Use for: user uploads, static websites, ML training data, backups, logs archives, media streaming

---

## Block Storage vs File Storage vs Object Storage

| Type | What It Looks Like | When To Use | Examples |
|---|---|---|---|
| Block Storage | Raw disk — OS sees it as an attached drive | Database files, OS disks, low-latency random I/O | AWS EBS, GCP Persistent Disk |
| File Storage (NFS) | Filesystem with folders and files; shared across servers | Shared file access across multiple servers, legacy apps | AWS EFS, Azure Files, NFS |
| Object Storage | Flat namespace of key-value blobs; HTTP access | Unstructured data at massive scale, media, backups | AWS S3, GCS, Azure Blob |

---

## Search Infrastructure

Databases are bad at full-text search (`LIKE '%keyword%'` is a full table scan). Dedicated search engines use inverted indexes — a mapping from every word to all documents containing it.

**Elasticsearch and OpenSearch Architecture:**
- **Index:** Like a database. Contains documents (JSON objects)
- **Shard:** Index split into N shards, each a Lucene index. Horizontal scaling
- **Replica shard:** Copies of primary shards for fault tolerance and read throughput
- **Node → Cluster:** Multiple nodes form a cluster. Master node manages cluster state

> ⚠️ **Important:** Elasticsearch is not a primary database — it's an eventually consistent search index. Pattern: Write to PostgreSQL as source of truth → event triggers Elasticsearch index update → search queries hit Elasticsearch only.

---

## Data Warehouses & Analytics

OLTP databases (PostgreSQL, MySQL) are optimized for many small reads/writes. OLAP — data warehouses — are optimized for complex aggregate queries over huge datasets.

| Aspect | OLTP (PostgreSQL) | OLAP (Snowflake/Redshift/BigQuery) |
|---|---|---|
| Query Pattern | GET user WHERE id=123 (point lookup) | SELECT COUNT(*) ... GROUP BY ... (aggregate) |
| Data Volume | GB to low TB | TB to PB |
| Write Pattern | Continuous small writes | Batch loads (hourly/daily ETL) |
| Row vs Column | Row-oriented storage | Column-oriented (compression, faster scans) |
| Users | App servers, real-time | Data analysts, BI tools |

---

## The Data Pipeline

Getting data from operational databases to analytics: ETL (Extract, Transform, Load) or its modern variant ELT (Extract, Load, Transform — transform in the warehouse).

- **Batch ETL:** Run nightly jobs (Apache Spark, AWS Glue, dbt) — simple but 24h stale
- **Streaming ETL:** Kafka → Flink/Spark Streaming → warehouse — near real-time analytics
- **Change Data Capture (CDC):** Debezium reads DB transaction logs to capture every insert/update/delete without polling — efficient and low-overhead
- **Data Lake:** S3/GCS as raw storage. Warehouse queries on top (query in place with Athena/BigQuery)

---

# Chapter 10: Reliability & High Availability

Building systems that stay up when things go wrong is one of the most important (and hardest) engineering challenges. Everything fails eventually — hardware, networks, software, even AWS regions.

---

## Availability Nines — What They Actually Mean

| SLA | Downtime per Year | Downtime per Month | Feasibility |
|---|---|---|---|
| 99% (2 nines) | 3.65 days | 7.3 hours | Easy — single server |
| 99.9% (3 nines) | 8.7 hours | 43.8 minutes | Standard — basic redundancy |
| 99.99% (4 nines) | 52.6 minutes | 4.4 minutes | Hard — requires serious HA work |
| 99.999% (5 nines) | 5.26 minutes | 26 seconds | Extremely hard — major investment |
| 99.9999% (6 nines) | 31.5 seconds | 2.6 seconds | Near impossible without massive spend |

> 🚨 **The 9s compound in multi-component systems.** If Service A is 99.9% available and Service B is 99.9% available and your app requires both, combined availability is 99.9% × 99.9% = 99.8%. Add more dependencies and availability drops fast.

---

## Redundancy Patterns

### Active-Active
All instances are live and serving traffic simultaneously. Load is balanced across all nodes. If one fails, others absorb its traffic. Best for high availability and horizontal scaling.

### Active-Passive (Primary-Standby)
One primary serves all traffic. One or more standbys are ready but idle. On primary failure, standby is promoted. Failover takes time (minutes). Simpler than active-active.

### Multi-Region Active-Active
Services run in multiple geographic regions simultaneously. Users route to nearest region. Requires data replication across regions — the hardest HA configuration.

> 🚨 Multi-region introduces the hardest problem in distributed systems: cross-region data consistency. You must choose between availability (serve requests even if regions disagree on data) and consistency (wait for regions to agree, adding latency).

---

## Health Checks and Auto-Healing

- **Liveness probe:** Is the service process alive? (Is it running at all?) → If fails, restart the container
- **Readiness probe:** Is the service ready to serve traffic? (DB connected? Cache warmed?) → If fails, remove from load balancer without restarting
- **Startup probe:** For slow-starting apps — give extra time before liveness checks kick in
- **Auto Scaling Groups (AWS ASG):** Automatically launch new instances to replace failed ones
- **Kubernetes:** Pod anti-affinity rules spread replicas across nodes/AZs; ReplicaSet ensures N healthy pods always

---

## Disaster Recovery

| Strategy | RPO | RTO | Cost | How |
|---|---|---|---|---|
| Backup & Restore | Hours | Hours/Days | Low | Daily backups to S3, restore manually |
| Pilot Light | Minutes | ~1 hour | Low-Med | Minimal prod replica + data replication; scale up on disaster |
| Warm Standby | Seconds | Minutes | Medium | Scaled-down copy of full prod; scale up on disaster |
| Multi-Site Active-Active | Near zero | Near zero | Very High | Full prod in multiple regions simultaneously |

> 💡 **RPO (Recovery Point Objective):** How much data loss is acceptable? RPO = 1 hour means you can lose up to 1 hour of data. **RTO (Recovery Time Objective):** How long can the system be down? RTO = 30min means service must be restored within 30 minutes.

---

## Chaos Engineering

Netflix pioneered the idea of deliberately breaking production systems to find weaknesses before they cause incidents. The Simian Army (Chaos Monkey, Chaos Kong) randomly kills instances, degrades networks, and terminates entire AZs in production.

- **Chaos Monkey:** Randomly terminates EC2 instances in production during business hours
- **Chaos Kong:** Simulates failure of an entire AWS Availability Zone
- **Latency Monkey:** Injects artificial network latency and packet loss
- **Start small:** Chaos in dev/staging first → canary in production → broad production
- **Tools:** Netflix Chaos Monkey, Gremlin, AWS Fault Injection Simulator (FIS), LitmusChaos (Kubernetes)

---

# Chapter 11: Observability — Knowing What's Happening

You can't fix what you can't see. Observability is the ability to understand the internal state of a system from its external outputs. The three pillars: Metrics, Logs, and Traces.

---

## The Three Pillars of Observability

### Metrics — The Numbers
Metrics are time-series numerical measurements. They're cheap to store, fast to query, and great for dashboards and alerting.

- **System metrics:** CPU usage, memory, disk I/O, network bytes
- **Application metrics:** Requests per second, error rate, response time (p50, p95, p99), queue depth
- **Business metrics:** Active users, orders per minute, revenue per hour, signup conversion rate

> 💡 **The RED Method for services:** Rate (requests/sec), Errors (error rate %), Duration (latency). **The USE Method for infrastructure:** Utilization (% busy), Saturation (queue length), Errors (error count). Apply these to every service and resource.

### Logs — The Story
Logs are time-stamped records of discrete events. They tell you exactly what happened and are invaluable for debugging specific issues.

- **Structured logging:** Emit JSON (not free-text) so logs can be queried programmatically
- **Log levels:** DEBUG (verbose dev info), INFO (normal events), WARN (concerning but non-fatal), ERROR (failures that need attention), FATAL (crash-level)
- **Correlation ID / Trace ID:** Attach a unique ID to each request; pass it through all services so you can find all logs for one user's request across your entire system
- **Log shipping:** Apps write to stdout/files → Fluentd/Logstash/Filebeat collects → Elasticsearch/S3 for storage/querying

### Traces — The Journey
Distributed tracing tracks a single request as it travels through multiple services. Each service emits "spans" (start time, end time, service name, tags). Spans are assembled into a trace — a visual waterfall of the request's entire journey.

> 💡 **Traces answer:** "Why was this user's checkout slow?" Trace shows: API gateway 2ms, order-service 5ms, payment-service 450ms (← bottleneck!), email-service 1ms. Without tracing, finding this in a microservices system takes hours of log grep.

| Tool | Pillar | Open Source? | Popular With |
|---|---|---|---|
| Prometheus | Metrics | Yes | Kubernetes, cloud-native |
| Grafana | Dashboards (metrics+logs+traces) | Yes | Universal visualization |
| Elasticsearch + Kibana | Logs + Search | Yes (ELK/OpenSearch) | Log analytics at scale |
| Jaeger / Zipkin | Distributed Tracing | Yes | Microservices, K8s |
| OpenTelemetry | Standards (metrics+logs+traces) | Yes (CNCF) | Vendor-neutral instrumentation |
| Datadog | All three (commercial) | No | Enterprises, full-stack |
| New Relic / Dynatrace | All three (commercial) | No | APM, enterprise monitoring |

---

## SLOs, SLAs, and Error Budgets

| Term | Definition | Example |
|---|---|---|
| SLI (Service Level Indicator) | A metric that measures service behavior | 99.5% of requests return HTTP 200 in < 200ms |
| SLO (Service Level Objective) | Internal target for an SLI | Maintain 99.9% of requests succeed per month |
| SLA (Service Level Agreement) | Contractual commitment with consequences | We guarantee 99.9% uptime; refund if we fail |
| Error Budget | Allowed failures within SLO | 0.1% failure rate allowed = 43.8 min downtime/month |

> 💡 Error budgets change the conversation from "developers vs ops" to "shared responsibility". If error budget is healthy → ship fast. If error budget is burning → freeze deployments, fix reliability. This is the core of Google's SRE model.

---

# Chapter 12: Security in System Design

Security is not an afterthought — it must be designed in from the start. This chapter covers the security concepts that come up in both real-world design and system design interviews.

---

## Authentication vs Authorization

### Authentication (AuthN) — "Who are you?" — verifying identity
- Username + password
- OAuth 2.0 (Login with Google/GitHub)
- Multi-Factor Authentication (MFA)
- Certificate-based auth (mTLS)
- Biometric (fingerprint, Face ID)

### Authorization (AuthZ) — "What can you do?" — verifying permissions
- RBAC (Role-Based Access Control)
- ABAC (Attribute-Based Access Control)
- ACLs (Access Control Lists)
- OAuth scopes (read:profile, write:posts)
- Policy engines (OPA, Casbin)

---

## JWT — JSON Web Tokens

JWTs are a compact, URL-safe way to transmit authentication information as a JSON object that is digitally signed. The server can verify the token without a database lookup.

- **Structure:** `header.payload.signature` — three base64url-encoded parts separated by dots
- **Header:** algorithm used (RS256, HS256) and token type
- **Payload (claims):** user ID, email, roles, expiration time (exp), issued at (iat)
- **Signature:** HMAC(header.payload, secret) or RSA/ECDSA signature — proves token wasn't tampered with

> 💡 **JWT vs Sessions:** Sessions store state server-side (session ID in cookie → server looks up data). JWTs are stateless — all data is in the token. JWT advantage: horizontally scalable (any server can verify). JWT disadvantage: can't revoke a token before expiry without a blacklist.

---

## OAuth 2.0 and OIDC

OAuth 2.0 is an authorization framework that lets users grant third-party applications limited access to their account without sharing their password. OpenID Connect (OIDC) adds an identity layer on top of OAuth 2.0.

1. User clicks "Login with Google"
2. Your app redirects to Google with client_id, requested scopes, and redirect_uri
3. User authenticates with Google and approves permissions
4. Google redirects back with an authorization code
5. Your server exchanges code for access_token + id_token
6. Access token used to call Google APIs; id_token contains user identity

---

## Encryption

| Type | What It Does | Algorithms | Use Cases |
|---|---|---|---|
| Symmetric | Same key encrypts and decrypts | AES-256, AES-128 | Encrypting data at rest, bulk data encryption |
| Asymmetric | Public key encrypts, private key decrypts | RSA-2048, ECDSA, Ed25519 | TLS handshake, digital signatures, key exchange |
| Hashing | One-way — cannot be reversed | SHA-256, bcrypt, Argon2 | Password storage, data integrity checks |
| TLS/HTTPS | Encrypts data in transit | TLS 1.3 (modern) | All external HTTP traffic, API calls |

> 🚨 **Never store plaintext passwords.** Use bcrypt, scrypt, or Argon2 — they are slow by design (prevents brute-force) and include a salt (prevents rainbow table attacks). PBKDF2 is acceptable. SHA-256 is NOT appropriate for passwords.

---

## HTTPS and TLS Termination

- TLS 1.3 is the current standard — enforce it, disable older versions
- **TLS termination:** Decrypt HTTPS at the load balancer/API gateway. Downstream services use HTTP (less overhead). Fine if internal network is trusted.
- **mTLS (Mutual TLS):** Both client and server authenticate with certificates. Used for service-to-service auth in zero-trust networks. Kubernetes service meshes (Istio) implement this automatically.

---

## Common Vulnerabilities — OWASP Top 10

| Vulnerability | What It Is | Prevention |
|---|---|---|
| SQL Injection | Malicious SQL in user input | Parameterized queries / prepared statements |
| XSS (Cross-Site Scripting) | Inject scripts into web pages viewed by others | Escape output, Content Security Policy (CSP) |
| CSRF (Cross-Site Request Forgery) | Trick user's browser into making unwanted requests | CSRF tokens, SameSite cookies |
| Broken Authentication | Weak session tokens, no brute-force protection | MFA, rate limiting, strong JWT, account lockout |
| Sensitive Data Exposure | Unencrypted sensitive data | Encrypt at rest and in transit, mask PII in logs |
| Insecure Deserialization | Untrusted data deserialized to execute code | Validate/sanitize all deserialized data |

---

# Chapter 13: System Design Interview Framework

The system design interview is an open-ended problem-solving session. There's no single right answer. Interviewers evaluate your thought process, communication, knowledge breadth, and ability to make and justify trade-offs.

> 🚨 **Top Mistake:** Jumping straight to a solution without clarifying requirements. In real engineering and interviews, you must understand the problem space before designing. A design without requirements is meaningless.

---

## The RADIO Framework (45-minute interview)

| Phase | Time | What You Do |
|---|---|---|
| R — Requirements | 5 min | Clarify scope, scale, constraints. Ask questions. Define what's in/out of scope. |
| A — Architecture | 10 min | High-level design. Draw the major components and how they connect. |
| D — Data Model | 5 min | Define entities, relationships, choose databases, design key schemas. |
| I — Interface (APIs) | 5 min | Define core APIs — endpoints, inputs, outputs. |
| O — Optimization | 20 min | Deep dive into bottlenecks. Scale each component. Handle failures. Add caching, CDN, sharding, etc. |

---

## Phase 1: Requirements Clarification — Questions to Ask

### Functional Requirements (What should the system do?)
- What are the core features? (Don't design everything — pick the most important 3-5)
- What is explicitly OUT of scope? (Confirm with interviewer)
- What does the user journey look like? (Walk through a common use case)

### Non-Functional Requirements (How should it perform?)
- **Scale:** How many users? DAU (Daily Active Users) / MAU? Read vs write ratio?
- **Latency:** What's acceptable response time? 100ms? 1 second? Real-time?
- **Availability:** What's the SLA? Can the system afford brief downtime?
- **Consistency:** Do users need to see data immediately after writing it?
- **Durability:** What happens if data is lost? Can we afford eventual consistency?

---

## Phase 2: Capacity Estimation (Back-of-Envelope)

Show you can reason quantitatively about scale. Use round numbers — precision is not the point. The goal is to identify scale bottlenecks.

**Key Numbers to Memorize:**
- Characters in a tweet: 280 | Average user photo: 200KB | Average user video (1 min): 50MB
- Seconds per day: 86,400 | Seconds per month: 2.6M | Seconds per year: 31.5M
- 1 million requests/day = ~12 req/sec | 1 billion req/day = ~12,000 req/sec
- HDD read: ~100MB/s | SSD read: ~500MB/s | RAM: ~10GB/s | Network LAN: ~1Gbps
- 99th percentile latency rule: often 10-50x the median latency

**Example estimation — Twitter storage:**
- 100M tweets/day × 280 bytes = 28 GB of text/day
- 100M tweets/day × 20% have images × 200KB = 4 TB of images/day
- Over 10 years: ~15 PB just for images (need object storage like S3)

---

## Phase 3: High-Level Design

Draw the system architecture at a high level. Start with the simplest possible design, then evolve it. Always draw left-to-right: Client → Load Balancer → App Servers → Database.

- **Client:** Web browser, mobile app, third-party service
- **DNS + CDN:** Static assets, edge caching
- **API Gateway / Load Balancer:** Entry point, routing, auth, rate limiting
- **App Servers:** Business logic, stateless (horizontal scaling)
- **Primary Database:** Source of truth for critical data
- **Cache:** Redis/Memcached for hot data
- **Object Storage:** S3 for files, media
- **Message Queue:** Kafka/SQS for async processing
- **Background Workers:** Process queue messages

---

## Phase 4: Deep Dives — Typical Interview Topics

| Topic | Common Interview Questions |
|---|---|
| Database Design | How do you design the schema? How do you handle scale? SQL or NoSQL? Why? |
| Caching Strategy | What do you cache? What's the eviction policy? How do you handle cache misses under load? |
| Scalability | How does this work with 10x traffic? Which component is the bottleneck? How do you shard? |
| Failure Handling | What happens if the database goes down? If the cache fails? If a server crashes? |
| Consistency | Is eventual consistency OK? How do you handle read-after-write consistency? |
| Security | How do you authenticate users? How do you prevent abuse? How is data encrypted? |

---

## How to Think About Trade-offs (The Interview Differentiator)

Great answers don't just present a solution — they acknowledge alternatives and explain why one was chosen over another. Use the phrase: *"I'm choosing X over Y because [requirement], which means we trade off [downside]."*

| Decision | Option A | Option B | How to Decide |
|---|---|---|---|
| Storage | SQL (consistent, relational) | NoSQL (scalable, flexible) | Is data relational? Need ACID? → SQL. Massive scale, flexible schema? → NoSQL |
| Communication | Synchronous (REST/gRPC) | Asynchronous (Queue) | Need immediate response? → Sync. Can process later? → Async for resilience |
| Caching | Read-through | Cache-aside | Want simpler app code? → Read-through. Want cache failure tolerance? → Cache-aside |
| Architecture | Monolith | Microservices | Small team, early stage? → Monolith. Large team, different scale needs? → Microservices |

---

# Chapter 14: Classic System Design Problems

These are the most commonly asked system design problems. Each solution demonstrates a combination of concepts from previous chapters.

---

## Design a URL Shortener (e.g., bit.ly)

### Requirements
- Given a long URL, return a short code (e.g., bit.ly/XyZ9k)
- Given the short code, redirect to the original URL
- Scale: 100M URL shortenings/day, 10B redirects/day (100:1 read/write ratio)

### Key Design Decisions
- **Short code generation:** hash(long_url) → take first 7 chars of MD5/SHA256. Problem: collisions. Better: auto-increment ID → Base62 encode (a-z, A-Z, 0-9 = 62 chars) → 7 chars = 62^7 = 3.5 trillion unique URLs
- **Storage:** Simple key-value. `shortCode → {longURL, createdAt, userId, expiresAt}`. Redis for hot codes (cache-aside). PostgreSQL for persistence.
- **Redirects:** 301 (Permanent) caches in browser — saves server load. 302 (Temporary) passes traffic through your server — enables analytics tracking. Choose based on whether you need click analytics.
- **Scale:** Read-heavy (10B/day = 115K req/sec). Multiple app servers + Redis cluster + read replicas. CDN can cache redirects at the edge.

> 💡 **Distributed ID generation:** At scale, multiple app servers generating IDs creates race conditions. Solutions: Redis INCR (atomic), UUID (128-bit, no coordination needed), Twitter Snowflake IDs (64-bit, time-sortable, machine ID embedded).

---

## Design a Key-Value Store (e.g., Redis / DynamoDB)

### Requirements
- GET(key) → value, PUT(key, value), DELETE(key)
- 100K reads/sec, 10K writes/sec, 1TB data, sub-10ms p99 latency

### Key Design Decisions
- **Data structures:** Hash table for O(1) lookups. But main bottleneck is storage — data must fit in RAM or spill to disk.
- **Persistence:** WAL (Write-Ahead Log) — write to append-only log before memory. On crash, replay log. Periodic snapshots (RDB in Redis) for fast recovery.
- **Replication:** Leader-Follower replication. Writes to leader only → async replicated to followers → reads can go to followers (eventual consistency).
- **Distribution:** Consistent hashing to shard data across N nodes. Virtual nodes for even distribution. Gossip protocol for cluster membership.
- **Conflict resolution:** Vector clocks or last-write-wins (LWW) with timestamps.

---

## Design a Rate Limiter

### Requirements
- Limit each user to N requests per second/minute/hour
- Works across multiple app servers (distributed)
- Low overhead — must be fast (adds latency to every request)

### Solution: Token Bucket with Redis
- For each user, store a Redis key: `rate_limit:user:12345 = {tokens: N, last_refill: timestamp}`
- On each request: use Lua script (atomic) to check and deduct tokens, refill based on elapsed time
- Lua script runs atomically on Redis — no race conditions with multiple app servers
- Return `X-RateLimit-Remaining` and `X-RateLimit-Reset` headers so clients can back off gracefully
- For API key rate limiting: use sorted sets — ZADD key timestamp member, ZREMRANGEBYSCORE removes old entries, ZCARD counts requests in window

---

## Design a Notification System

### Requirements
- Send push notifications, emails, SMS to users
- Support for iOS, Android, web push
- 1M notifications/minute, deliver within 10 seconds

### Architecture
- **Producer:** Any service publishes to Kafka topic "notifications" with `{userId, type, template, data, priority}`
- **Fanout worker:** Reads from Kafka → looks up user preferences (unsubscribed?, preferred channel?) → dispatches to channel-specific queues
- **Push notification worker:** Calls APNs (Apple) or FCM (Google Firebase) APIs. These are the actual delivery mechanisms.
- **Email worker:** Calls SendGrid / SES API with rendered template
- **SMS worker:** Calls Twilio / AWS SNS
- **Delivery tracking:** Store notification status in DB. Retry with exponential backoff on failure. Dead letter queue for permanently failed messages.

---

## Design a Chat System (e.g., WhatsApp)

### Requirements
- 1:1 messaging and group chats (up to 500 members)
- Message delivery receipts: sent / delivered / read
- Online presence (who's online)
- 50M DAU, 100B messages/day

### Architecture
- **Connection layer:** WebSocket servers — each user maintains a persistent WebSocket connection. Users are "sticky" to a server via consistent hashing on user ID.
- **Message flow:** User A sends message → WebSocket server A → Message service writes to DB + Kafka → Fanout service → if User B online on server B: push via WebSocket; if offline: store for later delivery
- **Message storage:** Cassandra (wide-column) — designed for write-heavy, append-only workloads. Key: `(chat_id, message_timestamp)` → scales to billions of messages.
- **Presence:** Use Redis. When user connects, `SETEX user:presence:12345 30 'online'` (expires in 30s). User sends heartbeat every 15s to refresh. Absence of refresh = offline.
- **Group messages:** Fan-out on write (each member gets a copy) for small groups. Fan-out on read (shared message, each user fetches) for large groups / celebrities.

---

# Chapter 15: Advanced Topics

These are the topics that differentiate senior from staff/principal-level system design thinking. They come up in senior interviews and are critical for large-scale production systems.

---

## Distributed Consensus — Raft and Paxos

How do multiple servers agree on a single value when they can't communicate perfectly? This is the core problem of distributed systems. Raft is the more understandable modern algorithm; Paxos is the classic but complex one.

**Raft algorithm overview:**
1. **Leader election:** Servers start as followers. If no heartbeat from leader within timeout, start election. Candidate requests votes. First to get majority wins.
2. **Log replication:** Client writes go to leader only. Leader appends to log, sends AppendEntries RPC to all followers. Once majority acknowledge, entry is committed.
3. **Safety:** A committed entry is guaranteed to survive leader failures as long as majority of servers are alive.

> 💡 **Used in:** etcd (Kubernetes config store), CockroachDB (distributed SQL), Consul (service discovery), TiDB. Raft guarantees strong consistency (CP in CAP) at the cost of write latency (must wait for majority to acknowledge).

---

## Bloom Filters

A Bloom filter is a probabilistic data structure that can tell you: "This element is DEFINITELY NOT in the set" or "This element is PROBABLY in the set" — using a tiny amount of memory compared to storing all elements.

- A Bloom filter with 10M elements uses only ~10MB vs 80MB for a hash set
- False positives possible (may say "in set" when it's not) — false negatives impossible
- Cannot delete elements (unless using Counting Bloom Filter)

> 💡 **Use cases:** Check if username is taken before DB query (reduce DB hits by 90%+). Check if URL was already crawled. Check if cache key exists before cache lookup. Cassandra uses Bloom filters to avoid disk reads for non-existent keys.

---

## Count-Min Sketch

A probabilistic data structure for counting frequency of elements in a stream using fixed memory, regardless of the number of distinct elements. Provides approximate counts with guaranteed error bounds.

- **Use cases:** Top-K trending topics, heavy hitter detection, approximate word frequency counts in huge logs
- **Memory:** O(width × depth) regardless of number of distinct elements — constant memory for streaming data
- **Error:** With width=1000 and depth=5, error is bounded by (total_count / 1000) with 96.8% probability

---

## Leader Election in Distributed Systems

Many distributed systems need exactly one leader at a time (to coordinate writes, to run a cron job, to manage a shard). Leader election ensures this even as servers fail.

- **Zookeeper:** Use ephemeral nodes. Leader creates `/leader` node. On crash, ephemeral node auto-deletes. Other servers watch for deletion and run election.
- **Redis with TTL:** Leader sets a key with short TTL and keeps refreshing. If leader fails, TTL expires, another node acquires the key. Redis Redlock for more robustness.
- **etcd / Consul:** Built-in leader election primitives based on Raft consensus. Most robust choice for production.

---

## Time in Distributed Systems

Distributed systems can't rely on wall-clock time being synchronized across servers. NTP synchronization is approximate (±1-50ms). This causes subtle bugs in ordering events.

| Approach | How It Works | Use Case |
|---|---|---|
| Logical Clocks (Lamport) | Counter per node; increment on send, max(counter)+1 on receive | Ordering events; not global time |
| Vector Clocks | Array of counters per node; tracks causal relationships | Conflict detection (DynamoDB uses this) |
| Hybrid Logical Clocks (HLC) | Physical time + logical counter; advances to max+1 on receive | CockroachDB — causally consistent, close to wall time |
| TrueTime (Google Spanner) | Hardware atomic clocks + GPS; returns time interval [earliest, latest] | External consistency (linearizability) across datacenters |

---

## Designing for Global Scale — Multi-Region

Serving users globally with low latency and high availability requires running in multiple geographic regions simultaneously.

| Strategy | Data Model | Complexity | Latency | Use When |
|---|---|---|---|---|
| Read-local, Write-global | One primary region; replicate reads worldwide | Low-Medium | Reads: local. Writes: high | Mostly-read data (product catalog) |
| Active-Active (Multi-master) | Each region can write; sync via CRDTs or events | Very High | Local for all ops | Social feeds, user-generated content |
| Data Residency / Geofencing | User data stored only in their legal region | High | Varies | GDPR, financial regulations |

> 💡 **CRDTs (Conflict-Free Replicated Data Types):** Mathematical data structures that can be merged without conflicts. A counter that only goes up (G-Counter), a last-write-wins register, a set you can only add to — these can be replicated to all regions and merged without coordination. Used in collaborative editing (Google Docs uses operational transforms, a related idea).

---

# Chapter 16: Study Roadmap & Resources

System design mastery requires consistent practice, not just reading. Here is a structured path from beginner to interview-ready to production-expert.

---

## Beginner (Weeks 1–2): Build the Foundation
- Master networking basics: HTTP, DNS, TCP/UDP, load balancers
- Understand SQL vs NoSQL trade-offs deeply
- Learn caching — Redis data structures, eviction policies, cache invalidation
- Practice: Design a URL shortener end-to-end
- Draw every system — even simple ones — with pen and paper or Excalidraw

## Intermediate (Weeks 3–4): Distributed Systems
- Study CAP theorem, BASE vs ACID, eventual consistency
- Learn message queues: Kafka architecture, Pub/Sub, event-driven patterns
- Understand microservices: communication patterns, service discovery, Saga pattern
- Study read replicas, sharding, consistent hashing
- Practice: Design a notification system, a chat app, Twitter feed

## Advanced (Weeks 5–6): Production Patterns
- Observability: Metrics (Prometheus), tracing (Jaeger/OpenTelemetry), structured logging
- Reliability: Circuit breakers, bulkheads, chaos engineering, multi-region HA
- Security: JWT, OAuth 2.0, mTLS, OWASP top 10, encryption patterns
- Advanced data structures: Bloom filters, Count-Min Sketch, HyperLogLog
- Study real architectures: How Netflix, Uber, Airbnb built their systems (their engineering blogs)

## Interview-Specific Practice (Week 7+)
- Do 2 mock interviews per week with a partner or on Pramp / Interviewing.io
- Time yourself — 45 minutes per problem. Communicate constantly.
- Practice the RADIO framework until it's instinct
- **Common problems to master:** URL shortener, Twitter/news feed, WhatsApp, Netflix, Uber/Lyft, Google Maps, Rate limiter, Search autocomplete, Web crawler, Distributed cache

---

## Essential Reading

| Resource | Type | What You'll Learn |
|---|---|---|
| Designing Data-Intensive Applications (DDIA) — Martin Kleppmann | Book | The bible of distributed systems. Read cover to cover. |
| System Design Interview Vol 1 & 2 — Alex Xu | Book | Common interview problems solved step by step |
| The System Design Primer — GitHub | Free Online | Comprehensive overview with diagrams |
| High Scalability Blog (highscalability.com) | Blog | Real-world architecture case studies |
| AWS / GCP Architecture Center | Documentation | Reference architectures for common systems |
| Netflix, Uber, Airbnb Engineering Blogs | Blog | Actual engineering decisions and why they were made |
| ByteByteGo Newsletter (Alex Xu) | Newsletter | Weekly system design concepts with great visuals |

---

## Quick Reference: When to Use What

| Need | Reach For |
|---|---|
| Cache hot data | Redis (in-memory KV) |
| Full-text search | Elasticsearch / OpenSearch |
| High-throughput event streaming | Apache Kafka |
| Simple job queue | AWS SQS / RabbitMQ |
| Store files and media | AWS S3 / GCS (object storage) |
| Distributed coordination | etcd / Zookeeper |
| Feature flags, A/B testing | LaunchDarkly / custom Redis flags |
| Rate limiting | Redis (token bucket / sliding window) |
| Service mesh / mTLS | Istio / Linkerd |
| Container orchestration | Kubernetes |
| Columnar analytics | Snowflake / BigQuery / Redshift |
| Time-series metrics | Prometheus / InfluxDB / TimescaleDB |

---

> 🚀 **Remember:** System design is about trade-offs, not perfect solutions. Every choice has costs. The engineer who understands these trade-offs and can articulate them clearly is the one who gets hired and promoted. Good luck!

---

*System Design Complete Guide · From Scratch to Advanced · Interview + Real-World Engineering*
