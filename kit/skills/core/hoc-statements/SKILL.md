---
name: hoc-statements
description: "Conventions for statements and control flow. Covers prohibiting the literal `undefined` in production code, avoiding sequential processing in favor of higher-order functions, and policies on ternary expressions and if statements."
---

# Shared: Statements

Summarizes conventions for statements and control flow. Applied across both functions and methods.

## Do not write the literal text `undefined` in production code

- Do not write the literal text `undefined` (as a literal or identifier) in real code files (production code).
- Reason: `undefined` is not a "value." If `undefined` is used intentionally as a value, it becomes impossible to distinguish between "a bug" and "intentional logic."
- When you intentionally want to express "no value," use `null` instead of `undefined`. Since `null` does not arise naturally in context, it can be used as an intentional value.
- Do not write `undefined` in JSDoc either. When a type annotation needs to express "no value," use `null` instead of `undefined` (e.g. `@returns {string | null}`).
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

## Do not discard the return value of a higher-order function (except `Array#forEach()`)

- Do not discard the return value of a higher-order function (`map` / `filter` / `reduce`, etc.). If the return value is neither assigned to a variable nor used as a return value, that code is prohibited.
- If the sole purpose is a side effect on each element (such as logging), use `Array#forEach()`. `forEach` is a higher-order function that assumes an `undefined` return value, so it is exempt from this rule.
- In particular, using `reduce` solely for side effects, returning the accumulator unchanged and discarding the return value, is a hack to evade the prohibition on sequential processing, and is prohibited.

```javascript
// NG: using reduce solely for side effects and discarding the return value (a workaround to evade the sequential-processing prohibition)
array.reduce(
  (_, it) => {
    this.logger.log('value:', it)

    return _
  },
  null
)

// OK: express a side effect on each element with forEach
array.forEach(it => {
  this.logger.log(
    'value:',
    it
  )
})
```

### The usage of `Array#forEach()` is limited

- ESLint prohibits writing an assignment statement inside `Array#forEach()`.
- Also, `if` statements are prohibited inside higher-order functions (express branching with `filter` or a conditional expression).
- Therefore, what can be written with `forEach` is limited to **a pure side effect that involves no assignment or branching** (such as logging or calling an external API).
- **An implementation that processes all elements equally while building up a result by pushing elements into an array or `Map` defined with `let`/`const` in the outer scope violates the spirit of the sequential-processing prohibition and is not allowed.** `push()` / `set()` are not assignment statements, so they slip past ESLint, but in substance this is an imperative loop and amounts to a workaround. Express iteration that assembles a result with `map` / `filter` / `reduce` / `Array.from`, etc., and receive it as a return value.

```javascript
// NG: assembling a result by pushing into an outer array inside forEach (violates the spirit of the sequential-processing prohibition)
const results = []
array.forEach(it => {
  results.push(transform(it))
})

// NG: assembling by setting into an outer Map
const map = new Map()
array.forEach(it => {
  map.set(it.id, transform(it))
})

// OK: assemble with map and receive the return value
const results = array
  .map(it => transform(it))
```

## The callback passed to a higher-order function should basically be a single statement

- The body of a function (callback) passed as an argument to a higher-order function should ideally consist of **only a single statement**.
- Even when writing multiple statements, limit it to what can be expressed as a single method name (i.e., extracted as
  a single responsibility).
- When there are multiple responsibilities, make full use of `Array#filter()` / `Array#map()` to split each stage into a single responsibility.

```javascript
// NG: the callback mixes multiple responsibilities (filtering + transformation)
const names = users.map(it => {
  if (!it.enabled) {
    return null
  }

  return it.name.toUpperCase()
})

// OK: split into filtering with filter and transformation with map, making each stage a single responsibility
const names = users
  .filter(it => it.enabled)
  .map(it => it.name.toUpperCase())
```

## Do not casually extract the body of map() into a method

- Do not casually extract the body of the higher-order function passed to `Array#map()` into a method.
- If the reason is "I want to use an `if` statement but can't inside a higher-order function," first consider using `.filter().map()`.

```javascript
// NG: extracting the body of map into a method just to use if
items.map(it =>
  this.convertItem({ item: it }) // convertItem merely branches internally with if
)

// OK: separate the condition with filter, then map
items
  .filter(it => it.enabled)
  .map(it => it.value)
```

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

## Treat all elements of an array equally in higher-order functions

- When handling an array with a higher-order function such as `map` / `filter` / `forEach`, do not special-case a particular element (such as the first one) by checking `index`. Transform/process all elements equally with the same logic.
- Even when it looks like you want to change the result only for the first element, first consider whether "processing all elements equally and absorbing the difference in a batch step such as after joining" is possible (see "Do not casually add if statements" in this skill for a concrete example).
- Exception: when `reduce()` / `reduceRight()` omits the second argument (the initial value `initialValue`), **the first element of the array is used as the initial value of the accumulator**. In this case, since **the second element onward is folded equally**, excluding the first element which is assigned to the accumulator, this does not violate the principle.

```javascript
// OK: omitting initialValue -> first element becomes the accumulator, second element onward is folded equally
const total = numbers
  .reduce((total, it) => total + it)
```

## Do not access array elements by subscript (`[]`)

- Accessing an individual array element via the `[]` operator (`array[0]` / `array[i]`) is prohibited.
- Pulling out a specific element to handle it specially violates "Treat all elements of an array equally in higher-order functions" (above), and is a circumvention of the discipline that a collection is "always used all at once" (the class design principles convention / the property-definition convention).
- Handle every element together with `map` / `filter` / `reduce` / `for...of`, etc., without pulling out individual elements.

```javascript
// NG: accessing an individual element by subscript
const head = segments[0]

// OK: handle every element together with map / reduce, etc.
segments
  .map(it => this.normalize({ segment: it }))
```

### Exception: first / last element

- The **last element** may only be obtained via `Array#at(-1)` (destructuring cannot express the last element).
- The **first (leading) element(s)** are obtained via destructuring (`const [first] = array` / `const [first, second] = array`). Do not use `array[0]` / `Array#at(0)`.
- Reason: destructuring lets you specify a default value at the point of assignment (`const [first = fallback] = array`), so completion such as `?? null` becomes unnecessary. It can also take multiple leading elements declaratively in a single statement.

```javascript
// NG
const head = segments[0]
const last = segments[segments.length - 1]

// OK: destructuring for the leading element (with a default), at(-1) for the last
const [head = ''] = segments
const last = segments.at(-1)
```

## Do not casually add if statements

- Do not casually add `if` statements.
- For a repeatable (iterative, regular) structure, first consider whether "it can be written with a single piece of logic."
- In particular, branching only on the first element by checking `index` inside a higher-order function violates the principle of "treat all elements of an array equally in higher-order functions."

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
