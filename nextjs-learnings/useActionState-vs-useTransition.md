---

# 1. `useTransition` (React 18+)

## Purpose

Allows you to mark some state updates as **low priority** so the UI stays responsive.

Typical use:

* Navigation
* Filtering large lists
* Async UI updates
* Preventing UI freezing

---

## Basic example

```tsx
"use client";
import { useState, useTransition } from "react";

export default function Search() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(e) {
    const value = e.target.value;
    setQuery(value);

    startTransition(() => {
      // simulate heavy filtering
      const filtered = bigList.filter(item =>
        item.includes(value)
      );
      setResults(filtered);
    });
  }

  return (
    <>
      <input onChange={handleSearch} />

      {isPending && <p>Searching...</p>}

      {results.map(r => <p key={r}>{r}</p>)}
    </>
  );
}
```

---

## What happens internally

Without `useTransition`:

```
typing → heavy filtering → UI freezes
```

With it:

```
typing stays smooth
filter runs in background
```

---

## When to use `useTransition`

### Perfect for:

* Search/filter UI
* Route changes
* Tab switching
* Expensive renders

### Avoid for:

* Form submission state
* API mutation state
* Auth state

(That’s where `useActionState` shines.)

---

# 2. `useActionState` (React 19 / Next.js Server Actions)

## Purpose

Designed for:

* Forms
* Server actions
* Mutations
* Async workflows

It automatically manages:

* loading state
* result state
* error state

---

## Basic example (Next.js login)

```tsx
"use client";
import { useActionState } from "react";

async function login(prevState, formData) {
  const email = formData.get("email");
  const password = formData.get("password");

  if (email === "admin@test.com" && password === "123") {
    return { success: true };
  }

  return { error: "Invalid credentials" };
}

export default function LoginForm() {
  const [state, action, pending] =
    useActionState(login, {});

  return (
    <form action={action}>
      <input name="email" />
      <input name="password" type="password" />

      <button disabled={pending}>
        {pending ? "Logging in..." : "Login"}
      </button>

      {state.error && <p>{state.error}</p>}
    </form>
  );
}
```

---

## What `useActionState` gives you

Instead of manually doing:

```
useState loading
useState error
useEffect
form handlers
```

It bundles:

```
action state
form submission
pending flag
result storage
```

---

# Real Next.js Server Action example

## Server:

```ts
// app/actions.ts
"use server";

export async function createUser(prev, formData) {
  const name = formData.get("name");

  await db.user.create({ data: { name } });

  return { success: true };
}
```

## Client:

```tsx
"use client";
import { useActionState } from "react";
import { createUser } from "./actions";

export default function Form() {
  const [state, action, pending] =
    useActionState(createUser, {});

  return (
    <form action={action}>
      <input name="name" />

      <button disabled={pending}>
        Create User
      </button>
    </form>
  );
}
```
 This is the **modern Next.js mutation pattern**.

---

# Difference Between Them (Very Important)

| Feature            | useTransition                  | useActionState             |
| ------------------ | ------------------------------ | -------------------------- |
| Purpose            | Non-blocking UI updates        | Async form/server actions  |
| Controls           | Rendering priority             | Action state               |
| Returns            | `[isPending, startTransition]` | `[state, action, pending]` |
| Used for           | UI responsiveness              | Mutations/forms            |
| Server integration | No                             | Yes (Server Actions)       |

---

# How they work together

Example:

```
Form submit → useActionState
UI update after submit → useTransition
```

Common pattern:

```tsx
startTransition(() => router.push("/dashboard"));
```

after successful action.

---

# In YOUR Next.js RBAC/Auth setup

Best usage:

## Use `useActionState` for:

* login/logout forms
* profile updates
* CRUD operations

## Use `useTransition` for:

* route switching
* dashboard filtering
* expensive UI updates

That aligns perfectly with your architecture.

---

# Common mistakes (avoid these)

## Using `useTransition` for API loading state

Wrong abstraction.

---

## Using context instead of `useActionState` for forms

Creates unnecessary complexity.

---

## ❌ Blocking UI without transition

Makes apps feel laggy.

---

#  TL;DR

## `useTransition`

 Makes UI updates non-blocking.

## `useActionState`

 Handles async form/server actions cleanly.

Both are key to modern React + Next.js.

---
