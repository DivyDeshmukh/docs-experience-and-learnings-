# Pagination Best Practices Guide  
## Your Current Problem  
In page.tsx, you're doing prisma.user.findMany() with no limit --- this fetches every user from the database and sends it all to the client. With 50 users it's fine, with 10,000 it'll crush your server memory and slow the page to a crawl.

### Decision 1: Server-Side vs Client-Side Pagination  

Approach | How it works | Best for  
Server-side | Only fetch N rows from DB per request | Large datasets (100+ rows)  
Client-side | Fetch all data, slice in the browser | Small datasets (<100 rows)  

For your admin dashboard → Server-side pagination is the right choice because the users table will grow unbounded.

For the teams table? Probably fine with client-side (you'll rarely have 100+ teams), but server-side keeps it consistent.

### Decision 2: Offset-Based vs Cursor-Based  

## Offset-Based (simpler, use this)

Page 1: skip 0, take 10 → rows 1-10  
Page 2: skip 10, take 10 → rows 11-20  
Page 3: skip 20, take 10 → rows 21-30  

Prisma:

```js
prisma.user.findMany({
  skip: (page - 1) * limit,
  take: limit,
})
````

Pros: Simple, supports "jump to page 5", easy to understand
Cons: Performance degrades on very large offsets (skip 100,000 rows), can miss/duplicate rows if data changes between pages

## Cursor-Based (advanced)

Page 1: take 10 → rows 1-10, cursor = id of row 10
Page 2: take 10, cursor = last_id → rows 11-20, cursor = id of row 20

Prisma:

```js
prisma.user.findMany({
  take: limit,
  skip: 1,              // skip the cursor itself
  cursor: { id: lastId },
})
```

Pros: Consistent performance regardless of depth, no missed/duplicated rows
Cons: Can't "jump to page 5", only next/previous

For your admin dashboard → Offset-based is the right pick. Your dataset won't be millions of rows, and admins need to jump to specific pages.

### Decision 3: Where to Store Page State

You have 3 options in Next.js App Router:

## Option A: URL Search Params (Recommended)

/dashboard/admin?page=2&limit=10

Why this is best:

* Shareable/bookmarkable URLs
* Browser back/forward works naturally
* Server Components can read searchParams directly
* No client state management needed

In your server component:

```ts
// page.tsx receives searchParams automatically
const AdminPage = async ({ searchParams }: { 
  searchParams: Promise<{ page?: string; limit?: string }> 
}) => {
  const { page: pageStr, limit: limitStr } = await searchParams;
  const page = Math.max(1, parseInt(pageStr || "1"));
  const limit = Math.min(50, Math.max(1, parseInt(limitStr || "10")));
  // ...
};
```

## Option B: Client State (useState)

* Loses state on refresh, not shareable
* Requires converting your server component to client-side fetching
* Avoid this --- you'd lose the SSR benefits you already have

## Option C: React Context / Store

* Overkill for pagination
* Avoid this

---

## The Implementation Layers (What You'd Build)

Here's the architecture, layer by layer from bottom to top:

### Layer 1: Database Query (Prisma)

You need two queries --- the data + the total count:

```js
// In page.tsx or a service function
const [users, totalUsers] = await Promise.all([
  prisma.user.findMany({
    include: { team: true },
    orderBy: { createdAt: "desc" },
    skip: (page - 1) * limit,   // ← pagination
    take: limit,                 // ← pagination
  }),
  prisma.user.count(),           // ← for calculating total pages
]);
```

Why count() separately?
You need the total to render "Page 2 of 15" and know when to disable the "Next" button. Prisma doesn't return total count with findMany, so this is the standard pattern.

---

### Layer 2: Pagination Metadata

Calculate and pass this to the client:

```js
const totalPages = Math.ceil(totalUsers / limit);

const pagination = {
  page,
  limit,
  totalItems: totalUsers,
  totalPages,
  hasNextPage: page < totalPages,
  hasPrevPage: page > 1,
};
```

---

### Layer 3: Pass to Client Component

```jsx
<AdminDashboard 
  users={users}
  teams={teams}
  currentUser={user}
  pagination={pagination}   // ← new prop
/>
```

---

### Layer 4: Pagination UI Component

The client component uses Next.js `useRouter` + `useSearchParams` to navigate:

```js
"use client";
import { useRouter, useSearchParams } from "next/navigation";

const Pagination = ({ pagination }) => {
  const router = useRouter();
  const searchParams = useSearchParams();

  const goToPage = (newPage: number) => {
    const params = new URLSearchParams(searchParams.toString());
    params.set("page", String(newPage));
    router.push(`?${params.toString()}`);
    // This triggers a server re-render with new searchParams!
  };

  return (
    <div className="flex items-center gap-2">
      <button 
        onClick={() => goToPage(pagination.page - 1)}
        disabled={!pagination.hasPrevPage}
      >
        Previous
      </button>
      <span>Page {pagination.page} of {pagination.totalPages}</span>
      <button 
        onClick={() => goToPage(pagination.page + 1)} 
        disabled={!pagination.hasNextPage}
      >
        Next
      </button>
    </div>
  );
};
```

Key insight: When you call `router.push("?page=2")`, Next.js re-runs your server component with the new searchParams. So your Prisma query runs again with `skip: 10, take: 10` --- and the page updates. No API route needed!

---

## The Complete Data Flow

User clicks "Page 2"
→ router.push("?page=2")
→ Next.js re-renders the Server Component
→ searchParams = { page: "2" }
→ Prisma: findMany({ skip: 10, take: 10 })
→ Props flow to AdminDashboard
→ UI updates with rows 11-20

This is the best pattern for your stack because:

* You keep SSR (no loading spinners for page data)
* Direct Prisma access (no extra API route)
* URL reflects state (bookmarkable, back button works)
* Minimal code changes to your existing architecture

---

## Best Practices Checklist

* Always validate & clamp inputs --- page must be >= 1, limit should have a max (e.g. 50) to prevent someone requesting `?limit=999999`
* Always run count in parallel with findMany using `Promise.all` --- not sequentially
* Keep pagination state in the URL --- not in `useState`
* Add a key to your table based on page, so React cleanly re-renders
* Show total info --- "Showing 11-20 of 156 users" is better UX than just "Page 2"
* Handle edge cases --- what if someone navigates to `?page=999`? Redirect to last valid page or show empty state
* Index your orderBy columns in the database --- you already have `@@index([email])` etc., which is good. `createdAt` ordering is fine since Prisma's cuid() IDs are roughly time-ordered

```
