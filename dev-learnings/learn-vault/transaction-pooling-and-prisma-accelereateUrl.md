# Transaction Pooler Overview

## What is a Transaction Pooler?

A **Transaction Pooler** is an intermediate layer between an application and the PostgreSQL database that manages and reuses a limited number of real database connections.

In serverless environments, short-lived function invocations can create many database connections very quickly. A pooler ensures that:

- The number of actual database connections stays within a safe limit.
- Connections are reused across requests.
- Excess requests wait for an available connection instead of opening new ones.

### Example Pooler Setup (Supabase)

Supabase provides a shared transaction pooler:

```

postgresql://postgres.<projectRef>:<PASSWORD>@aws-1-region.pooler.supabase.com:6543/postgres

```

This URL is used as `DATABASE_URL` in Prisma.  
The pooler accepts many client connections but only opens a smaller number of actual PostgreSQL connections (e.g., 15).

---

## Why Transaction Pooling Matters for Serverless

In a serverless platform like **Vercel**, every incoming API request may spin up an isolated function instance. Without pooling:

```

Request A → new function → new Prisma client → new DB connection
Request B → new function → new Prisma client → new DB connection
...

```

This rapidly exhausts the database connection limit.

With a transaction pooler:

- Vercel function instances still create Prisma clients.
- But they all connect to the **pooler**.
- The pooler provides a limited number of real DB connections (e.g., 15).
- Excess client requests wait for a free connection instead of creating new ones.

This prevents “too many connections” errors in production.

---

## How Supabase Transaction Pooler Works

### Key Terms

- **Pool Size**: Number of real PostgreSQL connections maintained (e.g., 15).
- **Max Client Connections**: Number of clients that can connect to the pooler concurrently (e.g., 200).

### Behavior

When client requests arrive:

1. First N clients (up to pool size) are assigned real DB connections.
2. Additional clients wait in the pooler queue.
3. When a connection completes a transaction, it is returned to the pool.
4. The next waiting client uses the freed connection.

If more than the max client connections arrive (e.g., 201+), the pooler rejects further connections.

### Example Scenario

- Pool size: 15
- Max clients: 200

200 concurrent requests:
- 15 get connections immediately
- 185 queue and wait
- As connections free up, waiting requests are served

Requests beyond 200 will be rejected.

---

## Prisma Setup with Transaction Pooler

### Environment Variables

```

DATABASE_URL="postgresql://...pooler.supabase.com:6543/postgres?pgbouncer=true"
DIRECT_URL="postgresql://postgres:<PASSWORD>@db.<projectRef>.supabase.co:5432/postgres"

````

- `DATABASE_URL`: Used by Prisma Client at runtime through pooler.
- `DIRECT_URL`: Used by Prisma CLI for migrations (direct DB access).

### Prisma Config

In `schema.prisma`:

```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}
````

This ensures migrations use direct connection (needed for schema changes), while runtime uses pooled connections.

---

# Comparing Transaction Pooler vs Prisma Accelerate (`accelerateUrl`)

## Transaction Pooler (e.g., Supabase)

* Hosted and managed by Supabase (you don’t self-host it).
* Acts like PgBouncer: pools PostgreSQL connections.
* Ideal for **serverless platforms** like Vercel.
* Handles pooling at the database level.
* You use its connection string as `DATABASE_URL`.
* Does not support `PREPARE` statements in some modes.

### Use Case

```
Next.js (Vercel serverless)
Supabase PostgreSQL
Supabase Transaction Pooler
Prisma Client (runtime)
```

### What it solves

* Limits real DB connections
* Prevents connection exhaustion
* Works with short-lived serverless functions

---

## Prisma Accelerate (`accelerateUrl`)

* A **Prisma product/service** (hosted by Prisma).
* Works as a global **managed proxy** and connection layer.
* Provides:

  * Connection pooling
  * Query caching
  * HTTP-based interface (can support environments where TCP is not possible, such as Edge functions)
* Requires you to set the `accelerateUrl` when creating `PrismaClient`.

### Where It Lives

* Hosted by Prisma (not Supabase).
* You do not self-host it unless specified by product options.

### Use Case

```
Edge Runtime (e.g., Cloudflare, Vercel Edge)
Global caching
Connection multiplexing
```

### What it solves

* Reduces database load even further.
* Helps with environments that cannot open raw TCP connections.

---

## Key Differences

| Feature          | Transaction Pooler               | Prisma Accelerate                   |
| ---------------- | -------------------------------- | ----------------------------------- |
| Who hosts        | Supabase                         | Prisma                              |
| Purpose          | Database connection pooling      | Managed proxy + pooling + caching   |
| How it’s used    | Use pooler URL as `DATABASE_URL` | Set `accelerateUrl` in PrismaClient |
| Best for         | Serverless (Vercel + serverless) | Edge functions + global performance |
| Caching          | No                               | Yes                                 |
| SQL capabilities | Direct pooled SQL                | Proxy layer over SQL                |

---

## Summary

* **Transaction Pooler** is a pooling layer between Prisma Client and PostgreSQL that limits real connections and queues excess requests.
* On serverless platforms, a pooler prevents connection exhaustion by reusing connections.
* **Prisma Accelerate** is a separate hosted product by Prisma that provides pooling and query caching with additional features, mainly useful for environments like Edge runtimes.
* In your stack (Next.js on Vercel + Supabase PostgreSQL + Supabase Transaction Pooler), you **do not need** `accelerateUrl`; you use the pooler connection string as `DATABASE_URL`.
* For migrations, use a direct Postgres URL via `DIRECT_URL`.

```
