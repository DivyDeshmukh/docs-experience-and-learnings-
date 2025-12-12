Complete Notes: Next.js Server vs Client Components + Rendering + Deployment + Serialization
🧩 1. Serialization Error in Next.js
Error
Only plain objects can be passed to Client Components from Server Components. Classes or other objects with methods are not supported.

Meaning

Server Components can only pass data that can be serialized (plain objects, primitives, arrays) to Client Components.

Functions, class instances, React component functions, and other non-JSON-serializable values cannot be passed as props from Server → Client.

📌 2. Why This Happens
Serialization

Serialization = converting values into a format safe to transfer (generally JSON).

JSON supports primitives, arrays, and plain objects.

It cannot represent functions or component definitions.

React components (function references) are non-serializable.

Example of invalid data
{ icon: Code2 } // ❌ React component function

Example of valid data
{ title: "Dashboard", count: 123 } // ✅ JSON-serializable

📌 3. Workarounds
Don’t pass component functions from server to client

Instead:

Pass strings or keys from server

Inside the client component, map string keys to actual component references

Example:

{ iconName: "Code2" }


Then in client:

const Icon = { Code2, AlertTriangle }[iconName];

🧠 4. Server Components vs Client Components
Default

Files in app/ are Server Components by default — run on the server.

They produce HTML and JSON descriptions for hydration.

Client Component

Mark explicitly with "use client" at the top.

Needed if:

You use state (useState,useEffect)

You interact with browser APIs (localStorage,window)

You handle events (onClick) or interactivity

📌 5. Server → Client Rules
Allowed	Not Allowed
Strings	Functions
Numbers	Component references
Booleans	Class instances
Plain objects	Anything with methods
Arrays of above	
📌 6. Why Serialization Matters
Concept

Server Components render HTML + JSON description of UI

JSON must be transferable over network

Functions and components can’t be turned into JSON

Mental Model
Server => HTML + JSON (data) => Browser

🧠 7. Component Children Passing
Allowed

Passing rendered JSX elements as children to a Client Component from server:

<ClientComp>
  <div>Hello</div>
</ClientComp>


Because JSX elements are plain data structures internally.

Not Allowed

Passing React component functions as children:

<ClientComp>
  <MyIcon /> // ❌ if MyIcon is a component function passed from server
</ClientComp>

📌 8. Client → Client Props Passing

If both parent and child are Client Components:

You can pass functions and component references freely

This is because both sides execute in the browser

Example:

<Button icon={MyIcon} /> // 🟢 both client

🧠 9. Server Component Receiving Props

Yes! Server components can receive props — but:

Props must be JSON-serializable if coming from server boundary

They can use props to render UI

📌 10. How Next.js Renders Pages
❖ Step 1 — Browser Request
GET /dashboard

❖ Step 2 — DNS Lookup

Browser resolves domain to IP via DNS.

❖ Step 3 — TCP/TLS Handshake

Establish network connection + encryption.

❖ Step 4 — Server Receives Request

Your VM runs Next.js server code.

❖ Step 5 — Server Renders HTML + JSON

Runs server components

Fetches data if needed

Generates HTML

Creates JSON describing the UI

❖ Step 6 — Response Sent

HTML + JSON sent to browser.

❖ Step 7 — Browser Displays HTML

User sees content even before JS runs.

❖ Step 8 — Hydration (if needed)

Browser downloads JS bundles for interactive parts and hydrates them.

🧠 11. Next.js vs CRA Rendering
CRA (Client-side rendering)

Server just serves static files

Browser executes all JS

React runs entirely in the browser

Interactivity and UI updates happen client-side

Next.js Server Rendering

Server renders initial HTML

Browser hydrates

Interactive parts run in browser

🧠 12. CSR vs SSR vs SSG vs ISR
CSR (Client-Side Rendering)

Browser fetches JS

React builds UI

No server UI rendering

SSR (Server-Side Rendering)

Server renders UI on each request

HTML sent to browser

Browser hydrates

SSG (Static Site Generation)

Build time rendering

Static HTML served

Browser hydrates

📌 13. Deployment Clarification
CRA Deployment

React code is compiled to plain JavaScript

Static files served by server/CDN

Browser runs all UI logic

Next.js Deployment

Server code runs on Node.js (or edge)

Some UI rendering may happen on server

Interactive parts run in browser

🧠 14. Server Execution Environment
Node.js

JavaScript runtime

Runs on server

Executes JS on server

Does not include browser APIs (e.g., DOM)

Browser

JS runs on browser’s JS engine (e.g., V8 in Chrome)

Has access to DOM, window, etc.

📌 15. Language Runtime Clarification

Node.js uses V8 engine to execute JS on server

Python uses Python interpreter (not JS)

Backend language runtime must be present for that language

🧠 16. Data Fetching in SSR / CSR

In SSR, server fetches data first → then renders HTML

In CSR, browser fetches data after loading JS

API calls from browser are executed in browser

📌 17. Mental Models
Server

➡ Produces HTML + JSON
➡ Can fetch secure data
➡ Runs pure JS (server logic)

Browser

➡ Renders UI
➡ Runs interactive JS
➡ Handles user events

📘 18. Key Analogies
SSR

Server = Kitchen
Prepares the food (HTML)
Sends food to table (browser)
Customer eats and adds seasoning (JS interactivity)

CSR

Server = Delivery
Sends recipe and ingredients
Customer cooks and eats

🧠 19. Flowchart Summary
Client Request
    ↓
DNS → Connect
    ↓
Server Receives Request
    ↓
Server Renders UI (SSR) + Data
    ↓
Send HTML + JSON
    ↓
Browser Renders
    ↓
If interactive:
Hydration → JS executes → UI is interactive

📌 20. Rules for Passing Values Between Components
From	To	Can Pass
Server	Client	Only JSON serializable
Client	Client	Anything
Server	Server	Anything
Client	Server	Only data via APIs
