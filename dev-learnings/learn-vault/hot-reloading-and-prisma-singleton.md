# Hot Reloading, Prisma Singleton, and Application Environments

## What is Hot Reloading?

Hot reloading, also known as Hot Module Replacement (HMR), is a development feature supported by many frameworks (including Next.js, NestJS, and React dev tools) that updates parts of a running application when source code changes, without restarting the entire server or page.

Hot reloading improves developer experience by:
- Applying code changes immediately in the dev environment,
- Preserving application state while modules reload,
- Avoiding full server restarts on every change.

When hot reloading runs, the Node.js module system may re-execute module files without restarting the entire Node process. This is important for modules that create singletons or global resources.

---

## Why We Use a Global Prisma Client Singleton

In traditional server code, creating a database client like PrismaClient each time a module loads can lead to too many database connections.

In development with hot reloading:
- Code files are reloaded frequently.
- A bare export like `export const prisma = new PrismaClient()` will create a new client instance on every reload.
- This can exhaust PostgreSQL connection limits (PostgreSQL often defaults to 100 max connections).

To prevent this, we leverage the Node.js `global` object to store a reference to the single Prisma client instance.

### How it works

```ts
declare global {
  var prisma: PrismaClient | undefined
}

export const prisma =
  global.prisma ??
  new PrismaClient({ log: ["query", "warn", "error"] })

if (process.env.NODE_ENV !== "production") {
  global.prisma = prisma
}
````

* `declare global` informs TypeScript that `global.prisma` may exist.
* We check `global.prisma` first to see if an instance exists.
* If not, we create one.
* In development, we assign it to the global object so subsequent reloads reuse it.
* In production, caching on global is optional; many environments instantiate fresh per process or per serverless execution.

This pattern prevents:

* Multiple Prisma clients being created on every hot reload,
* Database connection exhaustion,
* Unnecessary connection pools per reload.

---

## What Happens in Environments Without Hot Reloading

### Express (Node.js server)

In an Express application that is started with a normal process (e.g., `node server.js`), hot reloading is not enabled by default. Unless a development tool like `nodemon` is used to restart the server on file changes, the server process remains running.

In such environments:

* Your files load once at process startup,
* A plain `new PrismaClient()` at module top level only runs once,
* There is no need for a global singleton to avoid reload duplication,
* However, using a singleton pattern is still recommended for clarity and for serverless compatibility.

So in Express without HMR you could write:

```ts
export const prisma = new PrismaClient()
```

and it will only instantiate once per server process.

If you use a tool like `nodemon` to restart the server on file changes, it still restarts the entire Node process, so Prisma gets recreated cleanly on each restart. No global reuse is necessary in that scenario.

---

### React (Client-side)

React applications running purely in the browser (e.g., create-react-app, Vite, etc.) do not directly run Prisma clients because Prisma is a server-side ORM and cannot run in the browser.

So hot reloading in React only affects client code, not database access. Prisma singleton concerns do not apply on the client side.

---

## Hot Reloading in NestJS

NestJS supports hot reloading during development if it is configured with tools like `@nestjs/cli` and `webpack` or using the Nest CLI with `--watch`.

When hot reloading is enabled in NestJS:

* Nest reboots modules on change,
* The Node.js process can persist while modules reload,
* A Prisma singleton pattern is beneficial in development to prevent multiple connections,
* Without caching the instance globally, each reload module import could create new PrismaClient instances and exhaust connections.

NestJS does not provide global hot reload behavior by default; it uses underlying server reload mechanisms. But when NestJS is in watch mode, the same pattern of avoiding multiple PrismaClient instances should be used.

---

## Summary of Hot Reloading and Singleton Logic

| Environment        | Server restart behavior                | Singleton necessity                      |
| ------------------ | -------------------------------------- | ---------------------------------------- |
| Next.js Dev (HMR)  | Modules reload without process restart | Yes, singleton on global                 |
| Express (no HMR)   | Full restart only on server start      | Singleton optional                       |
| Express + nodemon  | Full process restart on code change    | Singleton optional                       |
| React (client)     | Hot reload client UI only              | Not applicable for Prisma                |
| NestJS Dev (watch) | Module reload on change                | Yes, singleton recommended               |
| Production         | No HMR, normal process                 | Singleton optional, based on environment |

---

## Best Practice Recommendation

* Always create a single Prisma client instance per running process.
* Use a global store in environments with hot module reload (Next.js dev, NestJS watch).
* In production, use the recommended connection pattern for your deployment environment (server, serverless, edge).
* Document your database patterns clearly so future developers understand why the singleton exists and when it is necessary.

```
