---
name: hoc-statements
description: "Conventions for statements and control flow inside a function or method body, test files included. Use when writing or reviewing any body of code, a destructuring included. How a higher-order function is used belongs to the higher-order function convention; where an expression is wrapped across lines, to the coding style convention; a method's signature, to the methods convention."
---

# Shared: Statements

Summarizes conventions for statements and control flow. Applied across both functions and methods.

## Do not write the literal text `undefined` in production code

- Do not write the literal text `undefined` (as a literal or identifier) in real code files (production code).
- Reason: `undefined` is not a "value." If `undefined` is used intentionally as a value, it becomes impossible to distinguish between "a bug" and "intentional logic."
- When you intentionally want to express "no value," use `null` instead of `undefined`. Since `null` does not arise naturally in context, it can be used as an intentional value.
- How a JSDoc type expresses "no value" — `null` rather than `undefined`, and the exception a third-party module forces — belongs to `/hoc-jsdoc`.
- This convention is enforced by ESLint's `no-undefined` rule. However, in `eslint.config.js`, `no-undefined: 'off'` is set for `tests/**/*.js`, so writing `undefined` in test files is exempt from this prohibition.

### Cannot distinguish between a bug and an intentional value

Using `undefined` as an intentional value makes it impossible to distinguish between "a bug" and "intentional logic" in cases like the following.

```javascript
const object = {
  alpha: undefined, // NG: undefined is written as an intentional value
}

console.log(object.alpha) // undefined
console.log(object.beta) // undefined

console.log('alpha' in object) // true 🔥 (the key ends up existing)
console.log('beta' in object) // false

// ---------------

function extractAlpha (object) {
  object.alpha // <-- ❌️ Forgot to write return (a bug)
}

console.log(
  extractAlpha({})
) // undefined (a bug, but indistinguishable from a normal return value)
```

### Use `null` instead of `undefined`

Since `null` does not arise naturally depending on context, it can be used as an intentional value.

```javascript
// NG: writing undefined
function extractAlpha (object) {
  return object.alpha
}

// OK: resolve to null with ?? null (expresses an intentional "no value")
function extractAlpha (object) {
  return object.alpha
    ?? null
}

console.log(
  extractAlpha({})
) // null
```

## `let` is prohibited; declare every variable with `const`

- **`let` is not written.** Every variable is declared with `const`; `var` is already an error
  under lint (`no-var`).
- A `let` exists in order to be reassigned, and a reassigned binding is sequential processing
  held in one name: what the name means depends on how far down the function the reader has got.
  Every rule below this one — higher-order functions in place of loops, a conditional expression
  in place of a branch that assigns — exists so that a value is computed once and named once.
- **Lint enforces it.** `@openreachtech/eslint-config` carries `Never use let` among its
  `no-restricted-syntax` options, so a `let` errors whatever it is for. `prefer-const` would have
  caught only the ones never reassigned — that is, exactly the harmless ones — which is why the
  prohibition is written as syntax rather than left to it.
- Where a `let` looks unavoidable, what sits underneath it is one of these:

| The `let` | What replaces it |
| :-- | :-- |
| Accumulating through a loop | `map` / `filter` / `reduce` / `Array.from` |
| Assigned in each branch of an `if` | A conditional expression, or `??` |
| Assigned later, once a guard has passed | An early `return`, or a function that returns the value |

```javascript
// NG: one name, two meanings
let endpoint = config.graphqlEndpoint
if (config.prefix) {
  endpoint = `${config.prefix}${endpoint}`
}

// OK: computed once, named once
const endpoint = config.prefix
  ? `${config.prefix}${config.graphqlEndpoint}`
  : config.graphqlEndpoint
```

## An instance is never taken apart by destructuring

**The members of an instance are not destructured — not on the right of a destructuring
assignment, and not in a parameter.** An instance of a class, `this` included, is asked for what
it holds. Destructuring is left for data: a named-argument object, a payload being read.

```javascript
// NG: an instance taken apart on the right of an assignment
const {
  send,
} = this.mailClient

// NG: an instance taken apart in a parameter
.filter(({ account }) => account.isActive())

// OK: the instance asked, and answering for itself
.filter(it => it.hasActiveAccount())
```

- **A method taken out of its instance no longer has it.** Called as `send()`, the method runs
  with `this` unbound, and the first line of it that reaches `this` fails.
- **A member taken out states what the instance holds inside.** The local is a second name for one
  value from then on, and the code around it now depends on the instance's insides.
- **Where the body needs something behind the instance, the instance grows the member that
  answers.** `it.hasActiveAccount()` keeps what an account is to the item that holds it.
- **`this` is the case met most often**, and why a property of `this` is read where it is used
  belongs to `/hoc-scope`.
- **A class or a module is not an instance.** Taking a named export or a static member out of one
  is not what this rule refuses.

## Sequential processing is prohibited; write with higher-order functions

- Sequential processing (imperative loops) using `for` and the like is prohibited.
- Express iteration/transformation with higher-order functions (`map` / `filter` / `reduce` / `Array.from`, etc.).

Example:

```javascript
// NG: sequential processing
let result = ''
for (let i = 0; i < length; i++) {
  result += pick()
}

// OK: higher-order function
const result = Array.from({ length }, () => pick())
  .join('')
```

- **Achieving sequential processing via recursion in a method or function in order to evade this prohibition is also prohibited.** Replacing a loop with recursion is still sequential processing, and amounts to a workaround. Express iteration with higher-order functions.

```javascript
// NG: achieving sequential processing via recursion (a workaround)
function build ({ length, result = '' }) {
  if (length === 0) {
    return result
  }

  return build({
    length: length - 1,
    result: `${result}${pick()}`,
  })
}

// OK: higher-order function
const result = Array.from({ length }, () => pick())
  .join('')
```

## How a higher-order function is used

- What becomes of a higher-order function's return value, what `forEach()` may hold, what a
  callback holds, and how its parameters are named belong to `/hoc-higher-order-functions`. It is
  read here because the prohibition above is what sends iteration there.

## Conditional (ternary) expressions

### Basic policy

- The purpose of using a conditional expression is, in principle, limited to "branching between values of the same kind based on a condition." This is a means for using `const` instead of `let`.
- There are two cases for switching a value based on a condition:
  1. When assigning to a variable with `const`
  2. When passing a conditional expression to a `return` statement

### Prohibition rules

**(1) Prohibited when it embodies the meaning of a control statement**

```javascript
// NG: embodies the meaning of a control statement
return this.nextComposer
  ? Object.setPrototypeOf(
    this.integratedResolver,
    this.nextComposer.composeResolver()
  )
  : this.integratedResolver

// OK: control it with an early return
if (!this.nextComposer) {
  return this.integratedResolver
}

return Object.setPrototypeOf(
  this.integratedResolver,
  this.nextComposer.composeResolver()
)
```

**(2) Prohibited when the branched values are not of the same kind**

```javascript
// NG: branching to a default value (not the same kind)
return condition
  ? processValue()
  : defaultValue

// OK
if (!condition) {
  return defaultValue
}

return processValue()
```

```javascript
// NG: branching to objects of different shapes (not the same kind)
return condition
  ? {
    alpha: 100,
  }
  : {} // has no alpha property

// OK
if (!condition) {
  return {}
}

return {
  alpha: 100,
}
```

OK when the values are of the same kind:

```javascript
// OK
const STATUS = {
  OK: 0,
  ERROR: 1,
}

return condition
  ? STATUS.OK
  : STATUS.ERROR

// OK
return condition
  ? 100
  : 200
```

When you want to assign to `const` but the values are not of the same kind, extract it into a method and refactor with an early return:

```javascript
// NG
const value = this.condition
  ? this.processValue()
  : this.defaultValue

// OK
const value = this.generateValue()

// ...

generateValue () {
  if (!this.condition) {
    return this.defaultValue
  }

  return this.processValue()
}
```

## How the elements of an array are treated

- That every element is treated alike, that none is reached by subscript, and the exceptions for
  the first and the last element belong to `/hoc-higher-order-functions`.

## Do not casually add if statements

- Do not casually add `if` statements.
- For a repeatable (iterative, regular) structure, first consider whether "it can be written with a single piece of logic."
- In particular, branching only on the first element by checking `index` inside a higher-order function violates the principle of "treat all elements of an array equally in higher-order functions" (see `/hoc-higher-order-functions`).

```javascript
// NG: branching to special-case only the first element by checking index inside map
segments
  .map((it, index) => {
    if (index === 0) {
      return it.toLowerCase()
    }

    return this.capitalizeSegment({ segment: it })
  })
  .join('')

// OK: capitalize all elements equally, and lower-case the first character in a single batch step after joining
return segments
  .map(it => this.capitalizeSegment({ segment: it }))
  .join('')
  .replace(/^./u, it => it.toLowerCase())
```
