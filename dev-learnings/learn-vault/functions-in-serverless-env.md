## 1) What “function” means in serverless

In **serverless platforms** like **Vercel**, your backend code is deployed as **individual small server processes** that run *only when needed*.

These serverless functions are:

* **Not your whole app**
* **Not always running**
* **Not a permanent server process**

They are like tiny servers that live just long enough to handle one incoming request, then shut down.

### Example

If someone requests:

```
GET /api/resources
```

Vercel spins up a serverless function to:

* run your handler code
* query the database
* send the response
* then destroy the process

This is very different from a traditional server (like Express on a VM) that stays running all the time.

---

## 2) What “new function instance” means

When I say:

> “A new function instance may be created…”

In serverless, that means:

* A new isolated execution environment is started
* For your API route
* With its own memory and variables

Each of those environment instances is separate.

So if 10 people hit your API near the same time, Vercel might create **10 separate function instances** to handle them.

Each instance:

* loads your code
* imports your modules
* creates its own PrismaClient instance (unless pooled externally)
* runs the request
* then exits

This is why connection handling in serverless is different than in a traditional always-running backend.

---

## 3) Why this matters for database connections

### Traditional server

In something like Express or NestJS hosted on a VM (AWS EC2), there is:

```
One long-lived process
```

* Modules load once
* PrismaClient created once
* Database connections reused across all requests
* Efficient connection reuse by design

---

### Serverless (e.g., Vercel)

Every request might cause:

```
New process
  → load modules
  → create new PrismaClient
  → open database connections
  → handle request
  → shut down
```

This is exactly why connection pooling (like Supabase Transaction Pooler) is crucial:
**because the database can be hit by many short-lived connections, and the database has a finite limit.**

If your serverless functions each open their own direct connections, you can hit connection limits quickly.

---

## 4) Why prisma singleton is used in dev

In dev (Next.js dev server):

* Hot Reload means Next.js keeps the server process running
* Modules can reload without restarting Node

So your Prisma file might reload often.

If you didn’t cache the Prisma client globally, every reload would create a new PrismaClient and open new connections — *but keep the old ones alive*.

That eventually exhausts your database.

By storing the instance on `global` in dev:

```
global.prisma = prisma
```

You ensure:

* one instance per dev server process
* reuse on reload
* no connection overload

This pattern is specifically for **hot reload in dev**.

---

## 5) Why it’s different in serverless production

In production on Vercel:

* Functions are short-lived
* Each may create its own Prisma client instance
* You *cannot* reliably cache the instance in a global because the process ends after the request

So for production, Prisma alone does *not* solve pooling.
This is where **Supabase Transaction Pooler** comes in — you point `DATABASE_URL` at the pooler so that even many short-lived function invocations result in a **small number of actual database connections**.

---

## 6) Concrete Summary

| Hosting Type                                   | Server Lifecycle                 | Concerns                                   |
| ---------------------------------------------- | -------------------------------- | ------------------------------------------ |
| **Traditional server (Express, NestJS on VM)** | Long-lived                       | Reuse Prisma client easily                 |
| **Dev server with Hot Reload (Next.js)**       | Long process with module reloads | Use singleton to avoid duplicates          |
| **Serverless (Vercel)**                        | Short lived per request          | Use connection pooler or Prisma Accelerate |

---

## 7) Real Example in Next.js

### API route

```
app/api/users/route.ts
```

Every request to this route triggers a serverless function instance.

Each instance:

1. Loads modules
2. Imports `prisma`
3. Runs database calls
4. Ends

That’s why you use pooling at the database layer, not singleton caching at the code layer, to handle many invocations safely.

---

## Short Definition (for docs)

A **serverless function instance** is a short-lived backend runtime spawned by the platform (like Vercel) to handle *a single incoming request*.
Unlike a traditional server that stays running and handles multiple requests, serverless functions start, execute, and then shut down automatically — which affects how database connections must be managed.

---
