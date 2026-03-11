# Horizontal vs Vertical Scaling (System Design Notes)

## 1. What is Scaling?

**Scaling** means increasing the capacity of a system so it can handle **more users, requests, or data without slowing down**. ([Aerospike][1])

When traffic grows, systems must scale to maintain performance and reliability.

Example:

* If a website initially handles **1,000 users**
* Later it must handle **1 million users**
* The system must **scale**.

There are **two main ways** to scale a system:

1. Vertical Scaling (Scale Up)
2. Horizontal Scaling (Scale Out)

---

# 2. Vertical Scaling (Scale Up)

## Definition

Vertical scaling means **increasing the power of a single machine** by adding more resources such as:

* CPU
* RAM
* Storage

Instead of adding new servers, you **upgrade the existing server**. ([CloudZero][2])

---

## Diagram

```
Before Vertical Scaling

      Server
   +------------+
   | CPU: 4     |
   | RAM: 8GB   |
   | Disk: 200GB|
   +------------+


After Vertical Scaling

      Server
   +------------+
   | CPU: 16    |
   | RAM: 64GB  |
   | Disk: 2TB  |
   +------------+
```

Same machine, **just more powerful**.

---

## Example

Suppose you have an **e-commerce website** running on one server.

Initially:

```
Server
CPU: 4 cores
RAM: 8GB
```

Traffic increases.

You upgrade the server:

```
CPU: 16 cores
RAM: 64GB
```

Now the **same server handles more users**.

---

## Real-World Example

Database server upgrade.

Example:

Before:

```
Database Server
RAM: 16GB
```

After:

```
Database Server
RAM: 128GB
```

Now queries run faster.

Banks often use vertical scaling to increase database performance. ([Aerospike][1])

---

## Advantages

1. **Simple to implement**
2. No need to change architecture
3. No distributed system complexity
4. Low development effort

---

## Disadvantages

1. **Hardware limit**

   * CPU and RAM cannot grow forever
2. **Single point of failure**

   * If the server crashes → system goes down
3. **Downtime during upgrade**

---

# 3. Horizontal Scaling (Scale Out)

## Definition

Horizontal scaling means **adding more servers** and distributing the workload across them. ([nOps][3])

Instead of making one server stronger, you **add multiple servers**.

---

## Diagram

```
Before Horizontal Scaling

        Users
          |
       +------+
       |Server|
       +------+

After Horizontal Scaling

           Users
             |
        +-----------+
        |LoadBalancer|
        +-----------+
         /    |    \
   +------+ +------+ +------+
   |Server| |Server| |Server|
   +------+ +------+ +------+
```

Requests are distributed across servers.

---

## Example

A website initially runs on **1 server**.

When traffic grows:

Instead of upgrading the server, you add more servers.

```
Server 1
Server 2
Server 3
Server 4
```

A **load balancer** distributes requests.

Example:

```
User 1 → Server 1
User 2 → Server 2
User 3 → Server 3
User 4 → Server 4
```

---

## Real-World Example

Large platforms like:

* Netflix
* Amazon
* Instagram

Use **thousands of servers** to handle millions of users.

Each server handles part of the traffic.

---

## Advantages

1. **Almost unlimited scalability**
2. **Fault tolerance**

   * If one server fails, others continue working
3. **Better performance for large traffic**
4. **High availability**

Horizontal scaling can increase throughput by distributing work across machines. ([DataCamp][4])

---

## Disadvantages

1. **Complex architecture**
2. Requires **load balancers**
3. Requires **distributed systems**
4. Data consistency becomes difficult

---

# 4. Simple Analogy (Very Important for Interviews)

## Vertical Scaling

Imagine a **restaurant kitchen**.

Instead of adding more kitchens, you buy:

* bigger stove
* faster oven
* bigger fridge

Same kitchen → **more powerful equipment**

That is **Vertical Scaling**.

---

## Horizontal Scaling

Instead of upgrading one kitchen:

You open **multiple kitchens**.

Each kitchen handles part of the orders.

That is **Horizontal Scaling**.

---

# 5. Comparison Table

| Feature         | Vertical Scaling        | Horizontal Scaling  |
| --------------- | ----------------------- | ------------------- |
| Also called     | Scale Up                | Scale Out           |
| Method          | Upgrade existing server | Add more servers    |
| Complexity      | Simple                  | Complex             |
| Hardware limit  | Yes                     | Almost unlimited    |
| Fault tolerance | Low                     | High                |
| Architecture    | Monolithic systems      | Distributed systems |
| Example         | Upgrade RAM             | Add more servers    |

---

# 6. When to Use Each

## Use Vertical Scaling When

* Traffic is **predictable**
* System is **small**
* Simplicity is important
* Application cannot be distributed

Example:

* Small business website
* Single database server

---

## Use Horizontal Scaling When

* Traffic grows rapidly
* Millions of users
* High availability required
* Cloud or microservices architecture

Example:

* Social media
* Video streaming
* E-commerce platforms

---

# 7. Real Interview Insight

Most **modern cloud systems use both**.

Example flow:

```
Step 1 → Vertical scaling (quick fix)
Step 2 → Horizontal scaling (long-term solution)
```

Because vertical scaling has **hardware limits**, while horizontal scaling can grow by adding more nodes.
