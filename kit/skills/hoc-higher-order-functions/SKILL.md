---
name: hoc-higher-order-functions
description: "How higher-order functions such as `map()`, `filter()`, `reduce()` and `forEach()` are used: what becomes of their return value, what a callback holds and is named, how elements are treated, and what `reduce()` may not be bent into. Use when writing or reviewing a callback passed to an array method, or when about to reach an element by index. Loops themselves belong to the statements convention; a collection a property holds, to the property-definition convention."
---

# Higher-Order Functions

How a higher-order function is used once iteration has been written with one. That loops are not
written at all is settled by the statements convention; what follows is what the higher-order
form may and may not be.

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

## `Array#forEach()` takes no assignment

- ESLint prohibits writing an assignment statement inside `Array#forEach()`. **The reason is to
  keep `forEach()` from standing in for `for` / `while`**: a loop that assembles a result needs
  somewhere to assign it, and with no assignment there is nothing for the callback to build.
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

## Treat all elements of an array equally in higher-order functions

- When handling an array with a higher-order function such as `map` / `filter` / `forEach`, do not special-case a particular element (such as the first one) by checking `index`. Transform/process all elements equally with the same logic.
- Even when it looks like you want to change the result only for the first element, first consider whether "processing all elements equally and absorbing the difference in a batch step such as after joining" is possible (see "Do not casually add if statements" in the statements convention for a concrete example).

## Do not access array elements by subscript (`[]`)

- Accessing an individual array element via the `[]` operator (`array[0]` / `array[i]`) is prohibited.
- Pulling out a specific element to handle it specially violates "Treat all elements of an array equally in higher-order functions" (above), and is a circumvention of the discipline that a collection is "always used all at once" (the property-definition convention).
- Handle every element together with a higher-order function — `map` / `filter` / `reduce` / `Array.from`, etc. — without pulling out individual elements.

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

## `reduce()` and `reduceRight()`

**The accumulator is built out of what the elements hold.** `reduce()` can be made to return
anything, which is why it is the method most often bent into something other than a fold: a
count, a side effect, a loop. Each of those is written with what was meant for it instead.

- **A callback that never reads its element is not folding the collection.** It returns the
  same thing whatever the elements are, so what it computes is something the collection already
  has — its length, written longhand. A count is answered by the class holding the collection
  (see the property-definition convention), and a name that promises a boolean does not return
  one.
- **Using `reduce()` only for its side effects is a loop in disguise**, covered by the return
  value section above: return the accumulator unchanged and discard the result, and what is left
  is `for` without the keyword.

```javascript
// NG: the callback never reads its element — this is the length, written longhand
hasValue () {
  return this.values.reduce(accumulator => accumulator + 1, 0)
}

// OK: the accumulator is built from what each element holds
const total = prices
  .reduce((total, it) => total + it.amount, 0)
```

- When `reduce()` / `reduceRight()` omits the second argument (the initial value `initialValue`), **the first element of the array is used as the initial value of the accumulator**. In this case, since **the second element onward is folded equally**, excluding the first element which is assigned to the accumulator, this does not violate the principle.

```javascript
// OK: omitting initialValue -> first element becomes the accumulator, second element onward is folded equally
const total = numbers
  .reduce((total, it) => total + it)
```

## Naming the callback parameters

- For a function passed to a higher-order function, use `it` as the parameter name for receiving each item.
- If a higher-order function is called inside another higher-order function, using `it` for the inner item too would be confusing, so name the inner item's parameter according to the meaning of its value.
- The first-layer callback argument should basically use `(it, index, array) => ...`.
- For `reduce()` and `reduceRight()`, name the first argument (the accumulator) appropriately based on the meaning of the value being accumulated (e.g. `total` for a running sum).

```javascript
// OK: item parameter is it
const ids = samples
  .filter(it => it.enabled)
  .map(it => it.id)

// OK: name the inner nested item by meaning (outer it / inner user, etc.)
const names = teams
  .flatMap(it =>
    it.members.map(user => user.name)
  )

// OK: name the reduce accumulator by meaning (total for a running sum)
const total = prices
  .reduce((total, it) => total + it.amount, 0)
```
