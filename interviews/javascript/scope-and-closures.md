# Scope and Closures (In-Depth)

## 1. What is Scope

Scope defines **where a variable is accessible in code**.

Types of scope in JavaScript:

```txt
Global Scope
Function Scope
Block Scope
````

---

## 2. Global Scope

Variables declared outside any function/block.

```js
let x = 10

function test(){
  console.log(x)
}
```

Accessible everywhere.

---

## 3. Function Scope

Variables declared inside a function.

```js
function test(){
  let x = 10
}

console.log(x) // error
```

Accessible only inside the function.

---

## 4. Block Scope

Variables declared using `let` and `const` inside `{}`.

```js
if(true){
  let x = 10
}

console.log(x) // error
```

---

## 5. Lexical Scope (VERY IMPORTANT)

Lexical scope means:

> Scope is determined by **where the function is written in code**, not where it is called.

---

### Example

```js
function outer(){
  let x = 10

  function inner(){
    console.log(x)
  }

  inner()
}

outer()
```

Output:

```txt
10
```

Why?

Because `inner` is **lexically inside outer**, so it can access `x`.

---

### Key Rule

```txt
Inner function can access variables of outer function
Outer function CANNOT access variables of inner function
```

---

## 6. Scope Chain

When accessing a variable:

```txt
JS engine searches:
current scope → outer scope → global scope
```

---

### Example

```js
let a = 1

function outer(){
  let b = 2

  function inner(){
    let c = 3
    console.log(a, b, c)
  }

  inner()
}
```

Search flow:

```txt
c → inner
b → outer
a → global
```

---

## 7. What is Closure

Closure =

```txt
Function + its lexical environment
```

A closure is created when a function **remembers variables from its outer scope even after that scope is gone**.

---

## 8. Closure Example

```js
function outer(){
  let count = 0

  return function inner(){
    count++
    return count
  }
}

const fn = outer()

fn() // 1
fn() // 2
fn() // 3
```

---

## 9. How Closure Works Internally

### Step-by-step

```js
const fn = outer()
```

Execution:

1. `outer()` runs
2. `count = 0` created
3. `inner()` is returned
4. Normally `outer` should be destroyed

BUT:

```txt
inner still references count
```

So JS keeps `count` alive.

---

### Memory Model

```txt
Call Stack:
outer() → removed

Heap (Closure Environment):
count → preserved
```

So:

```txt
fn → reference to function + closure scope
```

---

## 10. Why Closure Exists

Because of **lexical scoping + function returning behavior**

JS ensures:

```txt
variables used by inner function are not garbage collected
```

---

## 11. Closure Visualization

```txt
outer()
  ↓
count = 0
  ↓
return inner
  ↓
outer removed from stack
  ↓
count still exists in memory
```

---

## 12. Real-World Use Cases

### 1. Data Privacy (Encapsulation)

```js
function createCounter(){
  let count = 0

  return {
    increment(){
      count++
    },
    get(){
      return count
    }
  }
}

const counter = createCounter()

counter.increment()
counter.increment()
console.log(counter.get()) // 2
```

`count` is private.

---

### 2. Function Factories

```js
function multiplyBy(x){
  return function(y){
    return x * y
  }
}

const double = multiplyBy(2)

double(5) // 10
```

---

### 3. Memoization

```js
function memo(){
  let cache = {}

  return function(n){
    if(cache[n]) return cache[n]

    cache[n] = n*n
    return cache[n]
  }
}
```

---

### 4. Event Handlers

```js
function setup(){
  let count = 0

  button.addEventListener("click", ()=>{
    count++
    console.log(count)
  })
}
```

---

## 13. Closure Interview Puzzles

---

### Puzzle 1

```js
for(var i=0; i<3; i++){
  setTimeout(()=>{
    console.log(i)
  }, 100)
}
```

Output:

```txt
3 3 3
```

---

### Why?

* `var` is function scoped
* Only ONE `i` exists
* Loop finishes → `i = 3`
* All callbacks use same `i`

---

### Fix using let

```js
for(let i=0; i<3; i++){
  setTimeout(()=>{
    console.log(i)
  }, 100)
}
```

Output:

```txt
0 1 2
```

Because:

```txt
let creates new binding per iteration
```

---

### Fix using closure

```js
for(var i=0; i<3; i++){
  (function(i){
    setTimeout(()=>{
      console.log(i)
    },100)
  })(i)
}
```

---

## 14. Another Puzzle

```js
function outer(){
  let x = 10

  return function(){
    console.log(x)
  }
}

const fn = outer()

x = 20

fn()
```

Output:

```txt
10
```

Because closure captures **lexical scope**, not global variable.

---

## 15. Closure + Memory Leak (Important)

Closures can cause memory leaks if not handled.

Example:

```js
function create(){
  let largeData = new Array(1000000)

  return function(){
    console.log(largeData.length)
  }
}
```

Even if unused:

```txt
largeData remains in memory
```

---

## 16. Garbage Collection with Closures

Variables are removed ONLY when:

```txt
no reference exists
```

If closure exists:

```txt
memory is retained
```

---

## 17. Execution Context + Closure

Each function creates:

```txt
Execution Context
- Variable Environment
- Scope Chain
- this
```

Closure links to:

```txt
Lexical Environment
```

---

## 18. Key Interview Insights

* Closures keep variables alive after function execution
* Based on lexical scope, not dynamic scope
* Each function remembers its creation environment
* `let` creates block-level closures
* Common source of bugs in loops

---

## 19. Common Interview Questions

1. What is lexical scope?
2. What is closure?
3. How closure works internally?
4. Explain closure with memory diagram
5. Why closures are useful?
6. Difference between var and let in closures
7. Closure in loops problem
8. How closures cause memory leaks?

# Lexical Scope and Lexical Environment (Deep Dive)

## 1. What is Lexical Scope

Lexical scope means:

Scope is determined by **where the function is written (defined)**, not where it is called.

---

### Example

```js
function outer(){
  let x = 10

  function inner(){
    console.log(x)
  }

  inner()
}

outer()
````

Output:

```
10
```

---

### Why?

Because:

```
inner is defined inside outer
→ it gets access to outer's variables
```

---

### Important Rule

```
Scope depends on code structure (lexical structure), not runtime call stack
```

---

### Counter Example (Important)

```js
function outer(){
  let x = 10

  return function inner(){
    console.log(x)
  }
}

function test(){
  let x = 20
  const fn = outer()
  fn()
}

test()
```

Output:

```
10
```

Even though `test` has `x = 20`, inner does NOT use it.

Because:

```
inner was defined inside outer → lexical scope fixed
```

---

## 2. What is Lexical Environment (VERY IMPORTANT)

Lexical Environment is an internal structure created by JavaScript engine.

It stores:

```
1. Variables (environment record)
2. Reference to outer lexical environment
```

---

### Structure

```
LexicalEnvironment = {
  EnvironmentRecord: { variables },
  Outer: reference to parent scope
}
```

---

## 3. Example with Lexical Environment

```js
let a = 1

function outer(){
  let b = 2

  function inner(){
    let c = 3
    console.log(a, b, c)
  }

  inner()
}

outer()
```

---

### Memory Representation

#### Global Environment

```
Global Lexical Environment
a → 1
outer → function
Outer → null
```

---

#### outer() Execution

```
outer Lexical Environment
b → 2
inner → function
Outer → Global
```

---

#### inner() Execution

```
inner Lexical Environment
c → 3
Outer → outer environment
```

---

### Variable Lookup Flow

Inside `inner()`:

```
find c → current scope
find b → outer scope
find a → global scope
```

This is called:

```
Scope Chain
```

---

## 4. How Lexical Environment is Created

Whenever:

```
function is defined → lexical environment is created
function is executed → execution context is created
```

---

## 5. Connection Between Lexical Scope and Lexical Environment

| Concept             | Meaning                 |
| ------------------- | ----------------------- |
| Lexical Scope       | rules of access         |
| Lexical Environment | actual memory structure |

---

## 6. Closures and Lexical Environment

Closure works because:

```
function keeps reference to its lexical environment
```

---

### Example

```js
function outer(){
  let count = 0

  return function inner(){
    count++
    return count
  }
}

const fn = outer()
```

---

### What happens internally

1. `outer()` runs
2. `count = 0` created in outer environment
3. `inner` is returned

Normally:

```
outer environment should be destroyed
```

BUT:

```
inner references it
```

So:

```
outer lexical environment is preserved in heap
```

---

### Memory Representation

```
Call Stack:
outer → removed

Heap:
Closure Environment
count → 0
```

---

### Key Insight

```
Closure = function + reference to lexical environment
```

---

## 7. Why Closure Works

Because JS engine:

```
does NOT garbage collect variables
that are still referenced by inner functions
```

---

## 8. Important Interview Concept

Closure does NOT store variables.

It stores:

```
reference to lexical environment
```

---

## 9. Common Misconception

Wrong:

```
closure copies variables
```

Correct:

```
closure references same variable in memory
```

---

### Proof

```js
function outer(){
  let x = 10

  return function(){
    console.log(x)
  }
}

const fn = outer()
x = 20
fn()
```

Output:

```
10
```

Because:

```
closure refers to outer's x, not global x
```

---

## 10. Advanced Closure Flow

```js
function outer(){
  let x = 10

  return function inner(){
    x++
    return x
  }
}

const fn = outer()
fn() // 11
fn() // 12
```

Why value updates?

Because:

```
same lexical environment is reused
```

---

## 11. Visual Diagram

```
outer()
  ↓
creates lexical environment
  ↓
returns inner
  ↓
inner keeps reference to that environment
  ↓
environment stays in memory
```

---

## 12. Closure + Garbage Collection

Garbage collection removes variables only if:

```
no references exist
```

If closure exists:

```
memory stays alive
```

---

## 13. Key Interview Insights

* Lexical scope is decided at definition time
* Lexical environment stores variables + outer reference
* Closures keep lexical environments alive
* Variables are not copied, they are referenced
* Scope chain is based on lexical environment links

---

## 14. Common Interview Questions

1. What is lexical scope?
2. What is lexical environment?
3. Difference between scope and lexical environment?
4. How closure works internally?
5. Does closure copy variables or reference them?
6. What is scope chain?
7. Why closures don’t lose variables after function ends?
