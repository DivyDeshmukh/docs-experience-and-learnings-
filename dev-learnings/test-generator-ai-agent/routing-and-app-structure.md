# TestSmith AI – Routing & App Structure Decision

## What I Recommend for TestSmith AI — My Take

Given:

- This is primarily a tool for developers / teams (not a big marketing website yet).
- You plan auth + protected routes soon.
- Dashboard / internal features are central — marketing site isn’t a priority (yet).

I recommend going with **Option A now** — i.e. make `/` serve (or redirect to) the **Dashboard**.

Then later, once you build auth and maybe decide to expose a landing or marketing site, you can:

- Add a new route (e.g. `/landing` or `/home`) for that public content, or
- Move dashboard to `/app/dashboard` and reserve `/` for a public landing — but this refactor is straightforward if code is modular and routes are clean.

---

## Implementation Plan

- Create `app/page.tsx` → Dashboard page (or redirect to `/dashboard`).
- Add login-check logic:  
  If user is not logged in, redirect to `/login` or show a login modal / page.
- Use routing & layout structure that cleanly separates:
  - Public routes (login, maybe landing)
  - Protected routes (dashboard, generate-tests, etc.)
- If later you need a public landing page:
  - Create a new route outside the protected layout (e.g. `app/landing/page.tsx`)
  - Adjust layout/middleware accordingly.
