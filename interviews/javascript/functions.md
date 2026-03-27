# JavaScript Functions (In-Depth)

## 1. What is a Function

A function is a reusable block of code that executes when invoked.

Functions in JavaScript are **first-class citizens**, meaning:
- They can be assigned to variables
- Passed as arguments
- Returned from other functions

---

## 2. Function Declaration vs Function Expression

### Function Declaration

```js
function greet() {
  return "Hello"
}
````

Characteristics:

* Hoisted completely
* Can be called before definition

```js
greet() // works
function greet() {}
```

---

### Function Expression

```js
const greet = function() {
  return "Hello"
}
```

Characteristics:

* Not fully hoisted
* Stored as variable

```js
greet() // error
const greet = function(){}
```

---

## 3. Arrow Functions

```js
const add = (a, b) => a + b
```

### Key Differences

| Feature     | Arrow Function | Normal Function |
| ----------- | -------------- | --------------- |
| this        | lexical        | dynamic         |
| arguments   | not available  | available       |
| constructor | cannot use new | can use         |

---

### this Behavior

```js
const obj = {
  name: "Divy",
  normal: function() {
    console.log(this.name)
  },
  arrow: () => {
    console.log(this.name)
  }
}

obj.normal() // Divy
obj.arrow()  // undefined
```

Arrow function takes `this` from **parent scope**.

---

## 4. IIFE (Immediately Invoked Function Expression)

```js
(function(){
  console.log("IIFE")
})()
```

Purpose:

* Avoid global scope pollution
* Create private scope

---

## 5. First-Class Functions

Functions behave like values.

```js
function sayHi(){
  return "Hi"
}

function execute(fn){
  return fn()
}

execute(sayHi)
```

---

## 6. Higher-Order Functions

A function that:

* Takes another function as argument OR
* Returns a function

Example:

```js
function calculate(a, b, operation){
  return operation(a, b)
}

calculate(2, 3, (x,y)=>x+y)
```

---

## 7. Callback Functions

A function passed to another function and executed later.

```js
setTimeout(() => {
  console.log("Hello")
}, 1000)
```

Problem: Callback Hell

```js
a(() => {
  b(() => {
    c(() => {})
  })
})
```

---

## 8. Pure vs Impure Functions

### Pure Function

* Same input → same output
* No side effects

```js
function add(a,b){
  return a + b
}
```

---

### Impure Function

* Depends on external state

```js
let count = 0

function increment(){
  count++
}
```

---

## 9. Closures (Very Important)

Closure = function + its lexical environment

A function remembers variables from its outer scope even after that scope is gone.

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

fn() // 1
fn() // 2
```

---

### Memory Concept

```txt
outer execution finishes
BUT count is preserved in closure scope
```

Closure keeps variables alive.

---

### Use Cases

* Data hiding
* Private variables
* Caching
* Event handlers

---

## 10. Function Currying

Transform function:

```js
f(a, b, c)
```

Into:

```js
f(a)(b)(c)
```

---

### Example

```js
function curry(a){
  return function(b){
    return function(c){
      return a + b + c
    }
  }
}

curry(1)(2)(3)
```

---

## 11. Partial Application

Fix some arguments in advance.

```js
function multiply(a, b){
  return a * b
}

function double(x){
  return multiply(2, x)
}
```

Difference:

| Currying          | Partial             |
| ----------------- | ------------------- |
| one arg at a time | multiple args fixed |

---

## 12. Function Execution Context (Important)

Every function creates:

```txt
Execution Context
- Variable Environment
- Scope Chain
- this binding
```

---

## 13. Coding Implementations

---

### 1. compose()

Right to left execution.

```js
function compose(...fns){
  return function(x){
    return fns.reduceRight((acc, fn)=>fn(acc), x)
  }
}
```

Example:

```js
const add = x => x + 2
const multiply = x => x * 3

compose(add, multiply)(2)
// multiply → 6 → add → 8
```

---

### 2. pipe()

Left to right execution.

```js
function pipe(...fns){
  return function(x){
    return fns.reduce((acc, fn)=>fn(acc), x)
  }
}
```

---

### 3. Currying Implementation

```js
function curry(fn){
  return function curried(...args){
    if(args.length >= fn.length){
      return fn(...args)
    } else {
      return function(...next){
        return curried(...args, ...next)
      }
    }
  }
}
```

Usage:

```js
function sum(a,b,c){
  return a+b+c
}

const curriedSum = curry(sum)

curriedSum(1)(2)(3)
```

---

### 4. Memoization

```js
function memoize(fn){
  const cache = {}

  return function(...args){
    const key = JSON.stringify(args)

    if(cache[key]){
      return cache[key]
    }

    const result = fn(...args)
    cache[key] = result

    return result
  }
}
```

Example:

```js
const fib = memoize(function(n){
  if(n<=1) return n
  return fib(n-1) + fib(n-2)
})
```

---

## 14. Important Interview Insights

* Closures keep variables alive even after function ends
* Arrow functions don’t have their own `this`
* Functions are objects internally
* Higher-order functions are core to JS
* Currying and memoization are asked frequently

---

## 15. Common Interview Questions

1. What is closure? Explain with memory
2. Difference between arrow and normal function
3. What is currying?
4. Difference between curry and partial
5. Implement compose/pipe
6. What is pure function?
7. What is higher-order function?
8. Why arrow function doesn’t have `this`

```
