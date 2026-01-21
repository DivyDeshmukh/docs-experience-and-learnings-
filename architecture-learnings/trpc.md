# tRPC Summary Notes

## What tRPC Is

tRPC is a library for building APIs where backend functions are exposed as remote procedures that can be called directly from the frontend with full type safety. tRPC removes the need for separate API contract files or code generation by leveraging TypeScript’s type inference.

## How tRPC Works

1. Define a tRPC router on the backend with procedures that represent API endpoints.
2. Each procedure defines input and output types.
3. tRPC generates TypeScript types automatically from these definitions.
4. The frontend imports the types and client functions to call the backend procedures directly with type safety.
5. Behind the scenes, tRPC communicates over HTTP, but the client doesn't need to write REST endpoints or fetch logic manually.

## Core Concepts

### Router
A tRPC router is a collection of procedures representing API endpoints.

### Procedures
Procedures represent functions that can be invoked remotely. They can be `query` (read) or `mutation` (write), and can define validation logic and return types.

### Type Inference
TypeScript infers types automatically from backend definitions, and these inferred types are used by the frontend.

## Type Sharing Requirements

Type sharing only works when the backend and frontend can import the same type definitions. This is commonly done using:

- A monorepo with a shared workspace or package containing types.
- A published npm package containing shared types.
- Any setup that allows both frontend and backend to import the same TypeScript definitions.

tRPC does not require a monolithic application, only that types be shared.

## How tRPC Fits into API Contracts

Traditional API contracts are written manually (such as in markdown or Swagger), defining endpoints, request and response shapes, and error behavior. With tRPC, API contracts are defined in code:

- Input schema
- Procedure name and type
- Output schema
- Validation

These definitions are the contract. The frontend consumes them directly via shared types.

## tRPC vs Manual API Contracts

Traditional API development involves writing separate contract files and often generating code or types via code generation tools. tRPC removes this step by having the API contract defined in the code itself and automatically shared between backend and frontend.

## Does tRPC Replace Swagger

For internal applications where both frontend and backend are TypeScript, tRPC can act as the single source of truth for API contracts and remove the need for Swagger. However, for public APIs or when external clients that are not TypeScript need to consume the API, tools like OpenAPI/Swagger may still be required.

## Use Cases Where tRPC Is Ideal

- Full stack projects where both backend and frontend use TypeScript
- Projects where automatic type safety is desirable
- Projects where code generation is undesirable
- Internal APIs that do not require broad language support

## Use Cases Where tRPC Might Not Be Ideal

- APIs intended for use by clients implemented in many different languages
- Public APIs requiring language-agnostic documentation
- Situations where code generation is required for multiple client platforms

## Integrating tRPC with Frameworks

### Next.js

tRPC integrates seamlessly with Next.js API routes. Both server and client code live in the same codebase and share types directly.

### NestJS

tRPC can be mounted into NestJS with adapters or run in parallel. Nest services and controllers can be used within tRPC procedures.

### Express or Fastify

Adapters exist for tRPC to run on Express or Fastify servers.

## Type Sharing Without a Monolith

tRPC does not require a monolithic application. Instead, type sharing is possible through:

- Shared packages
- Shared workspaces
- npm packages with shared types

The frontend and backend both import shared types from a common location.

## Architectural Summary

tRPC is not a full framework like NestJS or Express. It is a type-safe transport layer that sits between frontend and backend. It is responsible for defining procedures, sharing types, and routing calls, but it does not replace business logic or architectural design.

## tRPC Compared to Other API Styles

| Feature | tRPC                        | REST + OpenAPI/Swagger  | GraphQL                  | gRPC                     |
|---------|-----------------------------|--------------------------|--------------------------|--------------------------|
| Type Safety | End-to-end native         | Partial with codegen     | Strong with codegen      | Strong with codegen     |
| Manual Contracts | Not needed            | Required                 | Schema first             | Schema first             |
| Language-Agnostic Clients | Hard         | Easy                     | Yes                      | Yes                      |
| Code Generation | Not needed             | Usually required         | Yes                      | Yes                      |
| Input Validation | Built-in with Zod       | Manual/Schema tools      | Schema based             | Schema based             |
| IDE Autocompletion | Native                 | Yes with type generation | Yes                      | Yes                      |
| External Non-TS Clients | Not ideal        | Best                     | Good                     | Best                     |

## Final Notes

tRPC allows teams building full stack TypeScript applications to define APIs in a type-safe manner without separate contract files, while maintaining IDE support and reducing duplicate definitions. To share types between backend and frontend, a shared code package or monorepo structure is needed, but a monolithic architectural pattern is not required. tRPC does not generate Swagger by default but can be used alongside documentation tools if required for external clients.
