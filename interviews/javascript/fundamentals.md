# JavaScript Fundamentals (Basics)

## 1. What is JavaScript

JavaScript is a high-level, interpreted, single-threaded programming language used to build interactive web applications.

Key characteristics:
- Dynamically typed
- Prototype-based
- Event-driven
- Supports asynchronous programming

---

## 2. Data Types in JavaScript

JavaScript has two categories of data types:

### Primitive Types

These store **actual values directly**.

- Number
- String
- Boolean
- Undefined
- Null
- Symbol
- BigInt

Example:

```js
let x = 10
let y = x
````

Memory model:

* `x` stores value `10`
* `y` gets a **copy of value**

```
x → 10
y → 10
```

Changing `y` does NOT affect `x`.

---

### Non-Primitive Types (Reference Types)

* Object
* Array
* Function
* Map
* Set

Example:

```js
let obj1 = {a:1}
let obj2 = obj1
```

Memory:

```
obj1 → reference → {a:1}
obj2 → same reference → {a:1}
```

Modifying one affects the other.

---

## 3. Primitive vs Non-Primitive (Core Difference)

| Feature    | Primitive  | Non-Primitive  |
| ---------- | ---------- | -------------- |
| Stored as  | Value      | Reference      |
| Memory     | Stack      | Heap           |
| Mutability | Immutable  | Mutable        |
| Copy       | Value copy | Reference copy |

---

## 4. Immutability

### Primitive Immutability

Primitive values cannot be changed.

```js
let str = "hello"
str[0] = "H"

console.log(str) // "hello"
```

New value is created instead:

```js
str = "Hello"
```

---

### Object Mutability

```js
let obj = {a:1}
obj.a = 2

console.log(obj) // {a:2}
```

Same memory is modified.

---

## 5. Type Coercion

JavaScript automatically converts types when needed.

### Implicit Coercion

```js
console.log("5" + 2)  // "52"
console.log("5" - 2)  // 3
console.log(true + 1) // 2
```

### Explicit Coercion

```js
Number("10")
String(10)
Boolean(0)
```

---

## 6. Equality (== vs ===)

### Loose Equality (==)

Performs type coercion.

```js
"5" == 5  // true
```

### Strict Equality (===)

No type conversion.

```js
"5" === 5 // false
```

---

## 7. Truthy and Falsy Values

### Falsy values:

```js
false
0
-0
0n
""
null
undefined
NaN
```

Everything else is truthy.

Example:

```js
if("hello") {
  console.log("truthy")
}
```

---

## 8. Undefined vs Null

| Feature | undefined                          | null              |
| ------- | ---------------------------------- | ----------------- |
| Meaning | variable declared but not assigned | intentional empty |
| Type    | undefined                          | object            |

Example:

```js
let x
console.log(x) // undefined

let y = null
```

---

## 9. NaN

NaN means "Not a Number".

```js
console.log("abc" - 1) // NaN
```

Special behavior:

```js
NaN === NaN // false
```

Correct way:

```js
Number.isNaN(NaN) // true
```

---

## 10. typeof Operator

Used to check type.

```js
typeof 10        // "number"
typeof "hello"   // "string"
typeof undefined // "undefined"
typeof null      // "object" (bug in JS)
```

---

## 11. instanceof

Checks prototype chain.

```js
[] instanceof Array  // true
{} instanceof Object // true
```

---

## 12. Variables (var, let, const)

### var

* Function scoped
* Hoisted
* Can be redeclared

```js
var x = 10
var x = 20
```

---

### let

* Block scoped
* Not redeclarable

```js
let x = 10
```

---

### const

* Block scoped
* Cannot be reassigned

```js
const x = 10
```

Important:

```js
const obj = {a:1}
obj.a = 2 // allowed
```

Because reference is constant, not content.

---

## 13. Hoisting

JavaScript moves declarations to the top.

### var

```js
console.log(x) // undefined
var x = 10
```

---

### let / const

```js
console.log(x) // ReferenceError
let x = 10
```

---

## 14. Temporal Dead Zone (TDZ)

The time between:

```js
variable declaration
AND
initialization
```

Example:

```js
console.log(x) // error
let x = 10
```

---

## 15. Spread Operator (...)

Expands elements.

```js
let arr = [1,2,3]
let newArr = [...arr, 4]
```

---

## 16. Rest Operator (...)

Collects values.

```js
function sum(...args){
  return args.reduce((a,b)=>a+b)
}
```

---

## 17. Optional Chaining (?.)

Safely access nested properties.

```js
user?.address?.city
```

---

## 18. Nullish Coalescing (??)

Returns right side only if left is null/undefined.

```js
let val = null ?? "default" // "default"
```

---

## 19. Key Interview Insights

* Primitives store values directly
* Objects store references
* `const` does not make objects immutable
* `null` is a bug (typeof null === "object")
* `==` vs `===` difference is critical
* Hoisting + TDZ is commonly asked

---

## 20. Common Interview Questions

1. Difference between `null` and `undefined`
2. Explain `==` vs `===`
3. Why `typeof null === "object"`
4. What is TDZ?
5. How `let` differs from `var`
6. What are truthy and falsy values
7. Explain primitive vs reference types
8. What is type coercion

```
And I’ll keep generating **clean GitHub-ready `.md` files like this**.
```
