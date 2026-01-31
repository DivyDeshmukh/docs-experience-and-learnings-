# Understanding the Pattern

Think of `asyncHandler` as a **function factory** — it creates and returns a new function.

---

## Step 1: What is a Higher-Order Function?

```javascript
// Simple example first:
function makeGreeter(greeting) {
  return function(name) {  // ← Returns a NEW function
    return `${greeting}, ${name}!`
  }
}

// Usage:
const sayHello = makeGreeter("Hello")  // Returns a function
sayHello("John")  // "Hello, John!"
````

**What happened?**

1. `makeGreeter("Hello")` creates a function that remembers `"Hello"`
2. That function is stored in `sayHello`
3. When we call `sayHello("John")`, it uses the remembered `"Hello"`

`asyncHandler` works the same way!

---

## Step 2: Let's Trace asyncHandler Execution

### Code:

```ts
export function asyncHandler(fn: AsyncHandlerFunction) {
  return async (req: Request, context?: any) => {
    try {
      return await fn(req, context)
    } catch (error) {
      // handle error
    }
  }
}
```

### Usage:

```ts
const myHandler = async (req: Request) => {
  const body = await req.json()
  return NextResponse.json({ data: body })
}

export const POST = asyncHandler(myHandler)
```

---

## Step 3: What Happens Line-by-Line

---

### Phase 1: Module Load (Before Any Request)

```ts
// Step 1: Define myHandler
const myHandler = async (req: Request) => { ... }

// Step 2: Call asyncHandler(myHandler)
asyncHandler(myHandler)

// Step 3: Inside asyncHandler
export function asyncHandler(fn) {  // fn = myHandler
  // Step 4: Return this new function
  return async (req, context) => {
    try {
      return await fn(req, context)  // fn still points to myHandler
    } catch (error) {
      // ...
    }
  }
}

// Step 5: The returned function is assigned to POST
export const POST = <the returned function>
```

### Result

`POST` is now a function that looks like this:

```ts
POST = async (req, context) => {
  try {
    return await myHandler(req, context)
  } catch (error) {
    // handle error
  }
}
```

---

### Phase 2: When HTTP Request Arrives

```ts
// User makes POST request to /api/resources

// Next.js calls:
POST(request, context)
```

Which is actually:

```ts
async (req, context) => {
  try {
    return await myHandler(req, context)
  } catch (error) {
    // handle error
  }
}
```

---

## Step 4: Why Write `req` and `context` in Inner Function?

### The Question

```ts
export function asyncHandler(fn) {
  return async (req, context) => {  // ← Why write these here?
    return await fn(req, context)   // ← And here?
  }
}
```

### The Answer: To Pass Them Through!

```ts
// Next.js calls your route handler like this:
POST(incomingRequest, routeContext)

// Your POST is:
async (req, context) => {  // ← Receives incomingRequest and routeContext
  return await fn(req, context)  // ← Passes them to your actual handler
}

// So your actual handler receives them:
const myHandler = async (req, context) => {  // ← Gets the values!
  const body = await req.json()  // ← Can use req
  const { id } = context.params  // ← Can use context
}
```

---

## Step 5: Visual Flow Diagram

```
HTTP Request arrives
    ↓
Next.js calls: POST(request, context)
    ↓
POST is: async (req, context) => { ... }
    ↓
Receives: req = request, context = context
    ↓
Calls: await fn(req, context)
    ↓
fn is: myHandler
    ↓
myHandler receives: req and context
    ↓
myHandler executes with those values
    ↓
Returns result
    ↓
Caught by try-catch if error
    ↓
Response sent back
```
