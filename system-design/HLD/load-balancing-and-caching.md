# Load Balancing, Hashing, and Consistent Hashing (System Design Notes)

## 1. Why Load Balancing Is Needed

When a system has only **one server**, all user requests go to that single machine.

Example:

```
Users → Server
```

If traffic grows (thousands or millions of users):

* CPU overload
* Memory exhaustion
* Slow responses
* Server crashes

### Solution: Multiple Servers

```
Users → Load Balancer → S1
                        S2
                        S3
                        S4
```

A **load balancer** distributes incoming requests across multiple servers so that no single server is overloaded.

### Goals of Load Balancing

* Distribute traffic evenly
* Improve scalability
* Improve reliability
* Reduce latency

---

# 2. Basic Load Balancing Strategies

## Round Robin

Requests are sent sequentially to each server.

```
User1 → S1
User2 → S2
User3 → S3
User4 → S4
User5 → S1
```

### Advantage

Simple and evenly distributes requests.

### Problem

Does not guarantee the **same user goes to the same server**.

---

## Least Connections

Requests go to the server with the fewest active connections.

Used in:

* Nginx
* HAProxy

Still does not guarantee user consistency.

---

# 3. When Simple Load Balancing Fails

Many applications store **temporary data locally on servers**.

Examples:

* user sessions
* authentication tokens
* cached data
* recommendation results

Example flow:

```
User123 → Server2
```

Server2 caches user data.

If the next request goes to Server3:

```
User123 → Server3
```

Problems:

* Cache miss
* Recalculation required
* Slower response

### Desired behavior

```
Same user → Same server
```

This is called **sticky routing** or **session affinity**.

---

# 4. Using Hashing for Sticky Routing

A common solution is to hash the user ID.

```
server = hash(user_id) % number_of_servers
```

Example with 4 servers:

```
server = hash(user_id) % 4
```

Example:

```
hash(user123) = 17
17 % 4 = 1
```

So:

```
User123 → Server1
```

Every request from that user goes to **Server1**.

Benefits:

* Cache reuse
* Session persistence
* Deterministic routing

---

# 5. Problem When Scaling Servers

Suppose we start with **4 servers**.

```
server = hash(user_id) % 4
```

Example users:

| User | Hash | Server     |
| ---- | ---- | ---------- |
| A    | 10   | 10 % 4 = 2 |
| B    | 21   | 21 % 4 = 1 |
| C    | 34   | 34 % 4 = 2 |
| D    | 47   | 47 % 4 = 3 |
| E    | 59   | 59 % 4 = 3 |

Now we add a **new server**.

```
server = hash(user_id) % 5
```

Recalculate:

| User | Hash | Old Server | New Server |
| ---- | ---- | ---------- | ---------- |
| A    | 10   | S2         | S0         |
| B    | 21   | S1         | S1         |
| C    | 34   | S2         | S4         |
| D    | 47   | S3         | S2         |
| E    | 59   | S3         | S4         |

Most users move to different servers.

### Consequences

* Cache invalidation
* Increased database load
* Performance degradation

In large systems **80–90% of keys may move**.

---

# 6. Consistent Hashing

To solve this problem, we use **consistent hashing**.

Instead of mapping keys directly to servers using `% N`, we map them to a **hash ring**.

Hash space example:

```
0 → 360
```

(Real systems use something like `0 → 2^32`)

Both **servers and users** are hashed onto this ring.

---

# 7. Placing Servers on the Hash Ring

Example server IDs:

```
server1
server2
server3
server4
```

Hash them:

```
hash("server1") % 360 = 100
hash("server2") % 360 = 150
hash("server3") % 360 = 220
hash("server4") % 360 = 300
```

Server positions:

```
S1 → 100
S2 → 150
S3 → 220
S4 → 300
```

---

# 8. Hashing Users

Example users:

```
userA
userB
userC
```

Hash them:

```
hash("userA") % 360 = 120
hash("userB") % 360 = 160
hash("userC") % 360 = 330
```

User positions:

```
UserA → 120
UserB → 160
UserC → 330
```

---

# 9. Assignment Rule

Rule:

**A key is assigned to the first server clockwise on the ring.**

Server positions:

```
S1 → 100
S2 → 150
S3 → 220
S4 → 300
```

### Example

UserA = 120

Next clockwise server:

```
150 → S2
```

So:

```
UserA → S2
```

UserB = 160

```
160 → next server 220
```

```
UserB → S3
```

UserC = 330

```
wrap around → 100
```

```
UserC → S1
```

---

# 10. Server Responsibility Range

Each server handles the range **between the previous server and itself**.

Example:

```
S1 → 100
S2 → 150
S3 → 220
S4 → 300
```

Ranges:

| Server | Range       |
| ------ | ----------- |
| S1     | (300 → 100] |
| S2     | (100 → 150] |
| S3     | (150 → 220] |
| S4     | (220 → 300] |

---

# 11. Adding a New Server

Suppose we add:

```
S5 → 130
```

New order:

```
S1(100) → S5(130) → S2(150)
```

Before:

```
S2 handled (100 → 150]
```

After:

```
S5 handles (100 → 130]
S2 handles (130 → 150]
```

Only users in **100–130 move**.

All others remain unchanged.

This minimizes data movement.

---

# 12. Why Consistent Hashing Works Better

If we have **N servers**, each server owns roughly:

```
1/N of the hash space
```

When a new server joins:

Only about

```
1/N keys move
```

Instead of **almost all keys moving**.

---

# 13. Problem with Basic Consistent Hashing

Servers may receive **uneven load**.

Example:

```
S1 → small range
S2 → huge range
```

S2 gets much more traffic.

---

# 14. Virtual Nodes (VNodes)

Solution: each server is represented by **multiple positions on the ring**.

Example:

```
Server1 → S1A(50), S1B(170), S1C(290)
Server2 → S2A(80), S2B(210), S2C(340)
Server3 → S3A(120), S3B(250), S3C(310)
```

Now the ring is divided into many smaller segments.

Benefits:

* balanced load
* better distribution
* smoother scaling

This is used in:

* Cassandra
* DynamoDB
* Redis Cluster

---

# 15. Real World Example: Distributed Cache

Large systems use many cache servers.

Architecture:

```
Users
   ↓
Load Balancer
   ↓
Application Servers
   ↓
Distributed Cache Cluster
   ↓
Database
```

Cache servers:

```
Cache1
Cache2
Cache3
Cache4
```

To determine which cache server stores data:

```
hash(key) → position on ring
```

Example:

```
hash("user:123") → Cache2
```

Benefits:

* even cache distribution
* minimal data movement
* scalable architecture

Used in:

* Memcached clusters
* Redis clusters

---

# 16. CDN Example

Content Delivery Networks like:

* Netflix
* Cloudflare
* Akamai

Use hashing to distribute cached content.

Example:

```
hash("movie_123") → EdgeServer7
```

If a new edge server is added:

Only a small portion of content moves.

---

# 17. Why Hashing Is Used

Hashing provides **deterministic mapping**.

```
same input → same output
```

Example:

```
hash(user123) → 845234
```

This helps in:

* routing
* caching
* distributed storage

---

# 18. Hashing for Fast Data Access

Hashing is also used in data structures.

Example: **HashMap**

```
index = hash(key)
```

This enables:

```
O(1) average lookup time
```

Used in:

* HashMap
* Redis
* dictionaries

---

# 19. Full Request Flow Example

User request:

```
GET /profile/123
```

System flow:

```
hash("profile:123")
      ↓
position on consistent hash ring
      ↓
Cache Server 5
```

Then:

```
Cache hit → return data
Cache miss → fetch from DB → store in cache
```

---

# 20. Final Key Takeaways

### Load Balancing

Distributes traffic across servers.

### Hashing

Maps keys deterministically to servers.

### Consistent Hashing

Minimizes data movement when servers change.

### Virtual Nodes

Improve load distribution across servers.

---

# One Line Summary

Consistent hashing distributes requests or data across servers in a way that **minimizes remapping when servers are added or removed**, enabling scalable distributed systems.
