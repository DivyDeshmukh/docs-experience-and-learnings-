

# React & Next.js Rendering, State, Effects, Memoization — Deep Notes

---

## 1. Rendering vs Re-rendering (Core Concept)

### What “rendering” means in React

Rendering means:

> React **executes the component function** to calculate what the UI should look like.

```ts
function Component(props) {
  return <div>Hello</div>;
}
```

Rendering is **pure computation**:

* No DOM mutation yet
* No state changes
* Just JSX → Virtual DOM

---

### What “re-rendering” means

Re-rendering means:

> React executes the component function **again** because something changed.

Important:

* Re-render ≠ remount
* Re-render ≠ state reset
* Re-render ≠ DOM update (unless output changed)

---

## 2. What Causes a Component to Re-render

A component re-renders when **any of the following happen**:

1. **Its state updates**

   ```ts
   setCount(count + 1)
   ```

2. **Its props change**

   ```ts
   <Child value={x} />
   ```

3. **Its parent re-renders**

   * Child re-renders even if props are same (unless memoized)

4. **Context value changes**

   ```ts
   useContext(SomeContext)
   ```

5. **Force update**
   (rare, discouraged)

---

### What does NOT cause a re-render

* Mutating variables directly
* Mutating objects without changing reference
* Assigning same primitive value again

---

## 3. Parent Re-render → Child Re-render Confusion (Clarified)

### Important distinction

> Parent re-rendering causes **child function execution**, NOT child state updates.

When parent re-renders:

* Child component function runs again
* Child receives new props
* Child **state remains unchanged**

React does this intentionally to:

* Preserve user input
* Prevent accidental state loss

---

### Why child state does NOT update automatically

```ts
const [value, setValue] = useState(initialValue);
```

* `initialValue` is used **only on first mount**
* On re-renders, React ignores it
* State is owned by the component

So:

* Props changing ≠ state changing
* Rendering ≠ state mutation

---

## 4. Why Rendering Exists (Its Purpose)

Rendering exists to:

1. Recompute UI based on latest state & props
2. Build a new Virtual DOM tree
3. Diff against previous Virtual DOM
4. Apply **minimal** DOM updates

Rendering keeps UI in sync with data.

---

## 5. State Updates vs Rendering

### State update flow

```text
setState → schedule re-render → render → diff → DOM update → effects
```

* State updates **cause** re-renders
* Re-renders **do not cause** state updates

---

### Why React separates them

If React reset state on every render:

* User input would be lost
* Forms would be unusable
* Performance would degrade

Separation gives:

* Predictability
* Control
* Stability

---

## 6. useEffect and Dependency Arrays (Deep Understanding)

### useEffect execution rules

| Syntax               | Runs When          |
| -------------------- | ------------------ |
| `useEffect(fn)`      | After every render |
| `useEffect(fn, [])`  | Once (after mount) |
| `useEffect(fn, [a])` | When `a` changes   |

---

### Why effects can cause infinite loops

```ts
useEffect(() => {
  setCount(c => c + 1);
});
```

Flow:

1. Render
2. Effect runs
3. State updates
4. Re-render
5. Effect runs again
6. Infinite loop

Reason:

* Effect runs after every render
* Effect itself triggers render

---

### Correct use of effects

Effects are for:

* Side effects (API calls, subscriptions)
* Syncing props → state (intentionally)
* Running logic after DOM commit

Effects should NOT:

* Mutate state unconditionally
* Recompute values every render

---

## 7. Why `console.log` Shows Old State in useEffect

```ts
useEffect(() => {
  setCount(c => c + 1);
  console.log(count);
}, []);
```

Reason:

* State updates are batched
* Effect closure captures old value
* Updates apply after effect finishes

Correct way to observe updated state:

```ts
useEffect(() => {
  console.log(count);
}, [count]);
```

---

## 8. Prefilling Forms & Dependency Confusion

### Key rule

> Rendering does NOT sync props into state automatically.

To sync intentionally:

* Use `useEffect` with dependencies
* Or use `key` to remount

---

### When to include dependencies

If parent can change `id` and fetch new data:

```ts
useEffect(() => {
  setCode(initialCode);
}, [initialCode]);
```

Effect runs only when value changes.

---

### Using `key={id}`

```tsx
<CodeInput key={id} />
```

What happens:

* Old component unmounts
* New component mounts
* All internal state resets

Best for:

* Edit forms switching between entities
* Clean reset without manual syncing

---

## 9. useState, References, and Value Equality

### Primitive values

* Compared by value
* `"a" !== "b"` triggers dependency change

### Objects & arrays

* Compared by reference
* `{}` !== `{}` even if contents same

This matters for:

* `useEffect` dependencies
* `React.memo`
* `useMemo`

---

## 10. React.memo — Preventing Child Re-renders

### What React.memo does

```ts
const Child = React.memo(Component);
```

It tells React:

> Re-render only if props change (shallow compare).

---

### Use cases

* Lists
* Headers / Navbars
* Heavy UI components
* Static children inside dynamic parents

---

### When React.memo fails

```tsx
<Child config={{ enabled: true }} />
```

Because:

* New object created each render
* Reference changes
* Memo breaks

---

## 11. useMemo — What It Is and Why It Exists

### What useMemo does

```ts
const value = useMemo(() => computeExpensive(a), [a]);
```

It memoizes **computed values**, not components.

---

### What problem it solves

Without `useMemo`:

* Computation runs every render
* Even when inputs are same

With `useMemo`:

* Computation runs only when dependencies change

---

### Real-world use cases

1. Expensive calculations
2. Derived data from props/state
3. Stable object references for `React.memo`
4. Prevent unnecessary child re-renders

---

### Example: fixing React.memo breakage

```ts
const config = useMemo(() => ({ enabled }), [enabled]);
<Child config={config} />;
```

---

### What useMemo does NOT do

* It does NOT prevent re-render
* It does NOT store state
* It does NOT replace effects

It only memoizes return values.

---

## 12. Initial Rendering & Mounting (Next.js + React)

### Initial mount sequence

1. Component function runs
2. Hooks initialize state
3. JSX → Virtual DOM
4. Virtual DOM → Real DOM
5. Browser paints UI
6. `useEffect(() => {}, [])` runs

---

### Next.js rendering pipeline (simplified)

1. Browser requests route
2. Next.js returns HTML (SSR/SSG)
3. Browser loads JS bundle
4. React hydrates HTML
5. Event handlers attach
6. App becomes interactive

---

### Hydration

Hydration:

* Reuses server-rendered HTML
* Attaches React logic
* No DOM rebuild if markup matches

---

## 13. What Happens on Refresh

Full browser refresh:

* React state is lost
* Memory cleared
* Components remount from scratch

To persist:

* localStorage
* URL params
* Server data

---

## 14. Key Mental Model (Final)

* Render = compute UI
* Re-render = recompute UI
* State persists across renders
* State changes trigger renders
* Props do not mutate state
* Effects sync intentionally
* Keys control component identity
* Memoization optimizes performance

---

## 15. Final Summary Table

| Concept    | Purpose                  |
| ---------- | ------------------------ |
| Render     | Compute UI               |
| Re-render  | Recompute UI             |
| State      | Persistent data          |
| Props      | External inputs          |
| useEffect  | Side effects             |
| useMemo    | Memoize values           |
| React.memo | Memoize components       |
| key        | Reset component identity |
| Refresh    | Reset entire app         |

---
