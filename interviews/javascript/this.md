# The `this` Keyword (In-Depth)

## 1. What is `this`

`this` is a special keyword in JavaScript that refers to **the object that is executing the current function**.

Important:

```txt
`this` is determined at runtime (not at definition time)
````

---

## 2. Key Rule (Most Important)

```txt
`this` depends on HOW a function is called, not where it is defined
```

---

## 3. Execution Context and `this`

Every function execution creates:

```txt
Execution Context
- Variable Environment
- Scope Chain
- this binding
```

`this` is decided during **execution context creation**.

---

## 4. Global Context

### Browser

```js id="q3t7jo"
console.log(this)
```

Output:

```txt id="3v9g4u"
window object
```

---

### Node.js

```js id="6c5a2p"
console.log(this)
```

Output:

```txt id="h2f1lw"
{}
```

---

## 5. Function Invocation (Normal Function)

```js id="r5t2ec"
function test(){
  console.log(this)
}

test()
```

### Behavior

* Non-strict mode → global object
* Strict mode → undefined

```js id="9evu8k"
"use strict"

function test(){
  console.log(this)
}

test() // undefined
```

---

## 6. Method Invocation (Object Method)

```js id="2k9t1z"
const obj = {
  name: "Divy",
  greet(){
    console.log(this.name)
  }
}

obj.greet()
```

Output:

```txt id="b5px4h"
Divy
```

Rule:

```txt
If function is called using object, `this` = that object
```

---

## 7. Losing `this` (Important Bug)

```js id="l1zp7k"
const obj = {
  name: "Divy",
  greet(){
    console.log(this.name)
  }
}

const fn = obj.greet
fn()
```

Output:

```txt id="8o0l3w"
undefined
```

Because:

```txt
function is called without object → `this` lost
```

---

## 8. Arrow Functions and `this`

Arrow functions do NOT have their own `this`.

They take `this` from **lexical scope**.

---

### Example

```js id="d83e2k"
const obj = {
  name: "Divy",
  arrow: () => {
    console.log(this.name)
  }
}

obj.arrow()
```

Output:

```txt id="1d83bf"
undefined
```

Because:

```txt
arrow function takes `this` from global scope
```

---

### Correct Usage

```js id="z5v2mt"
const obj = {
  name: "Divy",
  normal(){
    const arrow = () => {
      console.log(this.name)
    }
    arrow()
  }
}

obj.normal()
```

Output:

```txt id="c9n8jq"
Divy
```

---

## 9. Constructor Function (`new` keyword)

```js id="k8m7xq"
function Person(name){
  this.name = name
}

const p = new Person("Divy")
```

Here:

```txt
`this` → newly created object
```

---

## 10. Explicit Binding (call, apply, bind)

---

### call()

```js id="9avq2o"
function greet(){
  console.log(this.name)
}

greet.call({name: "Divy"})
```

---

### apply()

```js id="1c7k9x"
greet.apply({name: "Divy"})
```

---

### bind()

```js id="3qz6bc"
const bound = greet.bind({name: "Divy"})
bound()
```

---

## 11. Rules Priority (Important)

Order of precedence:

```txt
1. new binding
2. explicit binding (call/apply/bind)
3. implicit binding (object method)
4. default binding (global/undefined)
```

---

## 12. this in Event Handlers

```js id="8r2t5y"
button.addEventListener("click", function(){
  console.log(this)
})
```

Output:

```txt id="g5x9kt"
button element
```

Because:

```txt
browser sets `this` to element
```

---

## 13. this in Arrow Event Handler

```js id="7n1m8c"
button.addEventListener("click", () => {
  console.log(this)
})
```

Output:

```txt id="0z2x3y"
global object
```

---

## 14. this inside setTimeout

```js id="0c4z7m"
const obj = {
  name: "Divy",
  greet(){
    setTimeout(function(){
      console.log(this.name)
    },1000)
  }
}

obj.greet()
```

Output:

```txt id="b9z6kh"
undefined
```

Fix using arrow:

```js id="9l2m7q"
setTimeout(()=>{
  console.log(this.name)
},1000)
```

---

## 15. this vs Lexical Scope

| Feature       | this                | lexical scope |
| ------------- | ------------------- | ------------- |
| Determined by | how function called | where defined |
| Dynamic       | yes                 | no            |

---

## 16. Common Interview Tricky Example

```js id="2y6p7r"
const obj = {
  name: "Divy",
  greet(){
    return function(){
      console.log(this.name)
    }
  }
}

obj.greet()()
```

Output:

```txt id="f4m1c8"
undefined
```

---

### Fix

```js id="3v8x1k"
return () => {
  console.log(this.name)
}
```

---

## 17. Another Tricky Example

```js id="8p4t1z"
const obj = {
  name: "Divy",
  greet: function(){
    console.log(this.name)
  }
}

const another = {
  name: "Rahul"
}

another.greet = obj.greet
another.greet()
```

Output:

```txt id="6x8z9n"
Rahul
```

Because:

```txt
this depends on calling object
```

---

## 18. Internal Working (Deep)

When function is called:

```txt
Execution Context Created
↓
this binding assigned
↓
function runs
```

Example:

```txt
obj.greet()

this = obj
```

---

## 19. Important Edge Case

```js id="5l8x3c"
const obj = {
  name: "Divy",
  greet: () => {
    console.log(this.name)
  }
}

obj.greet()
```

Output:

```txt id="2x9z7k"
undefined
```

Because:

```txt
arrow function has no own this
```

---

## 20. Key Interview Insights

* `this` depends on call site
* Arrow functions inherit `this`
* `bind` permanently binds `this`
* `new` overrides all bindings
* `this` can be lost easily
* Event handlers behave differently

---

## 21. Common Interview Questions

1. What is `this` in JavaScript?
2. How is `this` determined?
3. Difference between arrow and normal function `this`
4. What happens when `this` is lost?
5. Explain call/apply/bind
6. What is precedence of `this` binding?
7. Why arrow function doesn't have `this`?
8. this inside setTimeout?

```
