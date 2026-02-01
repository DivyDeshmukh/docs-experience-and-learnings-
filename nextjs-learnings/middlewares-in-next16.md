# Middleware in Next.js 16 — Complete Notes

These notes explain Next.js 16 middleware (App Router), including rules, patterns, folder scopes, matching, and recommended implementation strategies. They summarize best practices and everything discussed about organizing and using middleware in large full-stack applications.

---

## 1. What is Middleware in Next.js

Middleware in Next.js provides a layer of code that runs **before requests** reach route handlers (API or pages). It allows you to:

- inspect incoming requests
- enforce authentication / authorization
- redirect or rewrite requests
- modify headers
- apply route protection logic

Middleware runs on the **Edge Runtime** by default, which is a lightweight environment optimized for performance and proximity to users; it is stateless and does not include Node.js built-in APIs. :contentReference[oaicite:1]{index=1}

In Next.js 16, the term `middleware.ts` is being replaced by `proxy.ts` as the official name for this interception file, but `middleware.ts` still works and shares the same behavior. :contentReference[oaicite:2]{index=2}

---

## 2. Where Middleware Lives

### Global Middleware

To intercept requests application-wide, create a file at the **project root**:

```

middleware.ts

```

or inside `src/` if your project uses a `src/` structure. This file runs for every request by default (or only on matched routes if configured). :contentReference[oaicite:3]{index=3}

Only **one** `middleware.ts` file per project is recognized by the Next.js runtime. The middleware function exported there is the entrypoint for all interception logic. :contentReference[oaicite:4]{index=4}

### Folder-Scoped Middleware

Next.js supports **folder scoped middleware** in the App Router:

```

app/protected/middleware.ts

```

Middleware in this location runs only for requests whose path falls under that folder (e.g., `/protected/*`). This lets you isolate middleware logic to only those routes without having to match and conditionally run within the global middleware. :contentReference[oaicite:5]{index=5}

Only one middleware file per folder segment is allowed — Next.js does not automatically merge multiple files from the same segment.

---

## 3. Rules About Middleware Files

- Middleware must be named `middleware.ts` (or `.js`) at the root or inside a route segment.
- Only **one** global middleware file is allowed per project root or per scoped folder.
- You cannot have multiple `middleware.ts` in the same folder acting independently — only the one in the folder or its closest parent applies.
- Folder scope means middleware runs only for that subtree’s routes. :contentReference[oaicite:6]{index=6}

**Example**:

```

app/api/public
app/api/protected
middleware.ts      # runs for any URL under /api/protected/*

````

Requests to `/api/public/*` will not trigger `app/api/protected/middleware.ts`.

---

## 4. How Middleware Runs (Request Lifecycle)

When a request arrives:

1. Next.js matches the request against middleware config (root or scoped).
2. Middleware executes before any route handler (`route.ts`).
3. Middleware can:
   - continue the request
   - rewrite or redirect
   - block the request
4. If middleware doesn’t send a response, the request is passed to the route handler.

Middleware runs **before** route handlers and can short-circuit routing decisions.

---

## 5. Running Middleware Only on Some Routes

By default, a global `middleware.ts` runs on all routes. To restrict it:

### Using `matcher`

You can export a `config` object specifying paths where middleware should run:

```ts
export const config = {
  matcher: [
    '/api/protected/:path*',
    '/dashboard/:path*',
  ],
};
````

Only matched routes will execute middleware. ([Stack Overflow][2])

---

## 6. Middleware Limitations

Middleware runs in an Edge Runtime:

* No Node.js core modules (`fs`, `path`, etc.)
* Cannot parse request body
* Cannot keep state between requests
* Must be fast and lightweight

Because of these limitations, middleware should be used only for pre-routing decisions such as token or header validation, not heavy computations or database access. ([contentful.com][3])

---

## 7. Typical Middleware Use Cases

Middleware should be used to:

* authenticate and authorize requests before handlers
* enforce role permissions
* redirect unauthenticated users
* attach minimal context (e.g., user info in headers)
* block or rewrite requests

Middleware is not intended to return data responses for normal API behavior — route handlers should do that.

---

## 8. How Many Middleware Files Can Exist

You can have:

✔ One global middleware file at root (`middleware.ts`)
✔ One middleware file per folder segment in which you want scoped logic

Only one middleware file is recognized per scope. Multiple files in the same scope won’t be combined or executed automatically. ([Clerk][4])

Example:

```
app/
  middleware.ts      # global
  dashboard/
    middleware.ts    # scoped to /dashboard
```

Requests to `/dashboard/*` will run `dashboard/middleware.ts`; others run `middleware.ts` or neither depending on matcher.

---

## 9. Folder Scoped vs Global Middleware

### Global Middleware

* Single entrypoint
* You must match routes manually inside if you have varied logic
* Useful for common guards (e.g., auth checks across many routes)

### Folder Scoped Middleware

* Automatic scoping to that folder segment
* Keeps logical separation clean
* Reduces need for route matching logic inside `middleware.ts`

Both approaches can be combined: scoped folders handle specific guard logic, while a global middleware handles broader concerns.

---

## 10. How to Implement Multiple Guard Logic

Because Next.js only runs a single middleware file per scope, you should *compose* middleware logic rather than rely on the framework to chain them automatically. Community recommendations clarify:

> Next.js only supports one middleware file and function per project. If you need multiple middleware behaviors, you must call them manually within that file. ([Clerk][4])

This means building a dispatcher that calls multiple guard functions in sequence.

---

## 11. Dispatcher Pattern for Multiple Guards

Define guard helpers in a `lib/middleware` folder and call them in your main `middleware.ts` file:

```
lib/middleware/
  auth.guard.ts
  role.guard.ts
  upload.guard.ts
```

Then inside `middleware.ts`:

```ts
const handlers = [authGuard, roleGuard('ADMIN')];
for (const handler of handlers) {
  const res = await handler(request);
  if (res instanceof NextResponse) return res;
}
```

This lets you run multiple guards in one middleware entrypoint.

---

## 12. Folder Scoped Middleware Example

```
app/
  api/
    protected/
      middleware.ts   # scoped guard
      users/
      auth/
```

Middleware in `protected/middleware.ts` runs before any route under `/api/protected/*`. No matching logic required — Next.js applies it based on folder hierarchy automatically.

---

## 13. Global Middleware with Matcher Example

```
middleware.ts

export const config = {
  matcher: ['/api/protected/:path*', '/dashboard/:path*'],
};

export function middleware(request) {
  // your logic here
}
```

Only matched paths will run this middleware.

---

## 14. Order of Middleware Execution

* Scoped middleware in deeper folder takes precedence
* Global middleware runs if no scoped middleware intercepts or if conditions overlap

Next.js will run at most one middleware per request scope (root or folder). Manual dispatch logic inside can branch as needed.

---

## 15. Best Practices for Middleware Logic

* Keep middleware fast: do not parse body or make heavy calls
* Use middleware only for pre-routing decisions (auth, redirects)
* Route handlers should handle business logic and responses
* Break guard logic into helper functions for reusability
* Use scoped middleware where possible for logical grouping

---

## 16. When to Use Global vs Scoped

Use **scoped middleware** when:

* guards apply only to a specific subtree
* you want automatic scoping without route matching

Use **global middleware** when:

* guards affect many disparate routes
* you want centralized logic

Often, use both: global for common checks, scoped for deeper concerns.

---

## 17. Example Folder Structures

### Minimal public/protected

```
app/
  api/
    public/
      auth/
        login/
        register/
    protected/
      middleware.ts
      users/
      auth/
```

Protected users and auth routes will run the scoped middleware automatically.

### Dispatcher pattern

```
middleware.ts
lib/middleware/
  auth.guard.ts
  role.guard.ts
  upload.guard.ts
```

Middleware code dispatches to guards based on request paths.

---

## 18. Summary

* Next.js supports **one middleware file per scope** (global or folder). ([Clerk][4])
* Global middleware must be recognized at the root or configured with `matcher`. ([Next.js][1])
* Folder scoped middleware runs automatically for that folder segment. ([Clerk][4])
* To run multiple guard functions, you must combine them in a single middleware function.
* Middleware is for pre-routing logic; business logic belongs to route handlers.

```
