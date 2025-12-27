# 1. Next.js “window is not defined” Error (Very Brief Note)

## What Causes It
Next.js uses **Server Components by default**.  
On the server, there is **no `window` object** because `window` exists only in the browser.

When code that references `window` runs during SSR, you get:

```

ReferenceError: window is not defined

````

## Why It Happens
Server components are executed on the server before sending HTML to the browser.  
Browser globals like `window`, `document`, and `localStorage` do **not exist on the server**.

## Fixes

### 1. Move code to a Client Component
If you need browser APIs, mark the component with:

```tsx
"use client";
````

### 2. Guard against server environment

Only access `window` inside client-safe checks:

```js
if (typeof window !== "undefined") {
  // safe to use window
}
```

or inside effects:

```tsx
useEffect(() => {
  // runs only in browser
  console.log(window.location.href);
}, []);
```

## Summary

* `window` is **undefined on the server** → Next.js SSR errors.
* Use `"use client"` or `typeof window !== "undefined"` guard.
* Wrap browser API code in `useEffect` when appropriate.

```
---
