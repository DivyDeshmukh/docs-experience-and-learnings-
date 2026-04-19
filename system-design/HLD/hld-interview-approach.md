## Part 1: The HLD Thinking Framework

Before memorizing components, understand that every system you'll ever design answers **3 fundamental questions**:

```
1. How does data GET IN?        (ingress / write path)
2. Where does data LIVE?         (storage)
3. How does data GET OUT?        (egress / read path)
```

Everything else — caches, queues, load balancers — are **optimizations** on these three paths.

If you forget everything else, remember this: **design the happy path first, then optimize.**

---

## Part 2: The 7-Step HLD Framework

This is the exact sequence I want you to follow in every interview. Memorize it.Now let me break down what goes *inside* each step.

---

## Step 1: Clarify Requirements

Split into two buckets:

**Functional requirements** (what the system does)
- "Users can post photos"
- "Admins can ban users"  
- "Family gets notified within 10 seconds"

**Non-functional requirements** (how well it does it)
- **Availability** — can it be down? (99.9%? 99.99%?)
- **Consistency** — do all users see the same data immediately?
- **Latency** — how fast per request? (100ms? 1s?)
- **Scale** — how many users? reads? writes?
- **Durability** — can we lose data ever?

> Pro tip: Always ask "what's the **read-to-write ratio**?" Instagram is read-heavy (100:1). Payments are balanced (1:1). This single number changes your entire design.

---

## Step 2: Estimate Scale (Back-of-Envelope Math)

This is where most people freeze. Here's the cheat sheet:

**Know these numbers cold:**
```
1 million seconds  ≈ 11.5 days
1 day              = 86,400 seconds ≈ 100,000 seconds
1 month            ≈ 2.5 million seconds

Rough orders of magnitude:
- Read from memory:     100 nanoseconds
- Read from SSD:        100 microseconds  
- Read from network:    1-10 milliseconds
- Read from disk (HDD): 10 milliseconds
```

**Formula for QPS (Queries Per Second):**
```
QPS = (Daily Active Users × Actions per user per day) / 86,400

Example: 10M DAU, each makes 20 requests/day
QPS = (10,000,000 × 20) / 86,400 ≈ 2,300 QPS average
Peak QPS ≈ 3× average ≈ 7,000 QPS
```

**Storage estimate:**
```
Rows per day × bytes per row × retention days
Example: 1M events/day × 1KB × 365 days ≈ 365 GB/year
```

---

## Step 3: Define APIs

Before designing the system, define the **contract**. This reveals complexity.

Format to use:
```
POST /api/v1/users/{userId}/alerts
Request:  { event_type, timestamp, video_url }
Response: { alert_id, status }

GET /api/v1/users/{userId}/alerts?from=...&to=...
Response: { alerts: [...], pagination: {...} }
```

**Why this matters:** Writing APIs forces you to think about edge cases. "What if timestamp is missing?" "What if video_url is invalid?" Interviewers love when you think about this.

---

## Step 4: Design Data Model

Answer two questions:
1. **What entities exist?** (User, Event, Camera, Alert)
2. **What are their relationships?** (User has many Cameras, Camera produces many Events)

Then pick storage:

| If your data is... | Use |
|---|---|
| Structured + needs transactions (payments, orders) | **SQL** (Postgres, MySQL) |
| Semi-structured + read heavy (social feeds) | **NoSQL Document** (MongoDB, DynamoDB) |
| Key-based lookups (sessions, cache) | **Key-Value** (Redis, Memcached) |
| Time-series (metrics, events) | **Time-series DB** (InfluxDB, Timescale) |
| Full-text search | **Search** (Elasticsearch) |
| Graph relationships (friends, recommendations) | **Graph DB** (Neo4j) |
| Large files (video, images) | **Blob storage** (S3) |

---

## Step 5: The Component Toolkit (The Big One)

These are the **Lego blocks** you assemble into any system. Every HLD is some combination of these.Let me explain each layer in depth — **when to use what, and why.**

### 🔵 Entry Layer

**DNS** — Translates `api.intelense.com` → IP address. First point of contact.

**CDN (Content Delivery Network)** — Edge servers geographically close to users. Use when: serving static content (images, videos, JS files) or cacheable API responses. *Saves latency + bandwidth.*

**Load Balancer** — Distributes incoming requests across multiple servers. Two flavors:
- **Layer 4 (TCP/IP)**: Fast, routes by IP/port. Used by AWS NLB.
- **Layer 7 (HTTP)**: Smart, can route by URL/header. Used by AWS ALB, Nginx.

**API Gateway** — Single entry point for all APIs. Handles auth, rate limiting, request transformation. Use when: you have multiple microservices behind it.

**Reverse Proxy** — Sits in front of servers. Similar to API gateway but lower-level. Nginx, HAProxy are classic examples.

> 💡 In interviews: Say "I'll put a load balancer here to handle traffic across my app servers." Don't overthink which type unless asked.

---

### 🟢 Compute Layer

**Monolith** — One big app. Use when: small team, early stage, simple domain.

**Microservices** — Split by business capability (UserService, PaymentService). Use when: large teams, independent scaling needs, different tech per service.

**Serverless (Lambda, Cloud Functions)** — Pay per execution. Use when: sporadic workloads, event-driven tasks (image resize, webhook handler).

**Workers / Background Jobs** — Consume from queues, do slow work. Use when: tasks take >1 second (email sending, video processing, ML inference).

> 💡 Rule of thumb: "Anything a user is waiting for = synchronous (fast). Anything they aren't = async (use workers)."

---

### 🟣 Data Layer

I already gave you the table above. But here's the **deeper reasoning**:

**SQL** — Use when data integrity matters. ACID guarantees. Joins are cheap.
- Postgres = default choice for most apps
- MySQL = slight edge for read-heavy workloads

**NoSQL Document** — Schema flexibility. Horizontal scaling built-in.
- MongoDB = general purpose
- DynamoDB = AWS managed, predictable performance

**Key-Value stores** — Extremely fast. No complex queries.
- Redis = in-memory, supports data structures (lists, sets, sorted sets)
- DynamoDB = disk-based but fast

**Columnar / OLAP** — For analytics (reports, dashboards).
- Clickhouse, BigQuery, Redshift

> 💡 Don't say "I'll use MongoDB" without justification. Say "The data is semi-structured (variable fields per event) and needs horizontal scaling for 1M writes/day, so MongoDB or DynamoDB fits."

---

### 🟠 Caching Layer

**Where to cache** (cache hierarchy):
1. **Browser cache** — closest to user, no network hop
2. **CDN cache** — edge servers
3. **API gateway cache** — at entry
4. **Application cache** — in-memory (local)
5. **Distributed cache** — Redis, Memcached (shared across servers)
6. **Database query cache** — built into DB

**Cache strategies:**
- **Cache-aside** — App checks cache, misses → fetches from DB → writes to cache. Most common.
- **Write-through** — Write to cache AND DB simultaneously. Strong consistency.
- **Write-behind** — Write to cache, async flush to DB. Fast writes, risk of data loss.

**Eviction policies:** LRU (Least Recently Used), LFU, TTL (Time To Live).

> 💡 Interview trick: "I'd cache user profiles in Redis with 5-min TTL using cache-aside pattern. This reduces DB load by ~80% since profile reads dominate."

---

### 🟡 Async Layer

This is where most candidates lose marks by skipping it.

**Message Queue (SQS, RabbitMQ)** — Point-to-point. One producer → one consumer. Use for: task queues, retry logic.

**Pub-Sub (Kafka, Redis Pub/Sub)** — One producer → many consumers. Use for: event broadcasting, fan-out, analytics pipelines.

**When to use async:**
- Slow operations (video processing, ML inference, email sending)
- Decoupling services (OrderService doesn't need to know EmailService exists)
- Smoothing traffic spikes (queue absorbs the burst)
- Retries on failure (message stays in queue until processed)

> 💡 Intelense example: Fall detected → event published to Kafka → multiple consumers: NotificationService (alerts family), AnalyticsService (logs for ML training), StorageService (archives video clip). **Decoupled, resilient, scalable.**

---

### 🟢 Observability Layer

Interviewers ding candidates for ignoring this.

**Three pillars:**
- **Metrics** — numbers over time (QPS, latency p99, error rate). Tools: Prometheus, Datadog.
- **Logs** — text events. Tools: ELK stack, Splunk, CloudWatch.
- **Traces** — request path across services. Tools: Jaeger, OpenTelemetry.

**SLIs / SLOs / SLAs:**
- SLI = Service Level **Indicator** (what you measure — e.g., latency)
- SLO = Service Level **Objective** (your target — e.g., p99 < 200ms)
- SLA = Service Level **Agreement** (contractual — e.g., 99.9% uptime or refund)

> 💡 Mention this in interviews: "I'd emit metrics for alert latency, track p99 SLO of 10 seconds, with PagerDuty alerts on SLO breach."

---

## Step 6: Deep Dive

Interviewer will pick ONE component and say *"go deeper."* This is where LLD comes in — which we'll do next.

Common deep dive topics:
- "How does the ML service scale to 10k concurrent streams?"
- "Walk me through what happens when a database node fails"
- "How would you deduplicate fall events from the same camera?"

---

## Step 7: Bottlenecks & Scale

This is the "what breaks first?" game. Always mention these 5 scaling techniques:

1. **Vertical scaling** — bigger machines (simple but limits)
2. **Horizontal scaling** — more machines (needs stateless services)
3. **Caching** — reduce expensive operations
4. **Database sharding** — split data across DB instances (by user_id, geography)
5. **Read replicas** — separate read and write traffic

**CAP theorem** (you must know this):
- **Consistency** — all nodes see same data
- **Availability** — system always responds
- **Partition tolerance** — works despite network splits

You can only pick **2 out of 3** during a network partition. Most systems choose AP or CP.
- CP: Banking, payments (consistency critical)
- AP: Social feeds, recommendations (availability critical)

---

## The Cheat Sheet You Should Tattoo On Your Brain

Here's the mental checklist I want you to run through for *any* HLD problem:

```
CLARIFY    → Features? Scale? Constraints?
ESTIMATE   → QPS? Storage? Bandwidth?
APIs       → What endpoints? Request/response shape?
DATA       → Entities? SQL or NoSQL? Why?
DRAW       → Entry → Compute → Data → Cache → Async
DEEP DIVE  → Pick the hardest part, explain
SCALE      → Where does it break? How to fix?
```

Ready to dive into LLD of the ML service? Or any questions on the HLD toolkit first?
