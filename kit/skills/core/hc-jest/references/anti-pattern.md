# Anti-pattern (Do Not Write Logic in Test Files)

Conventions for "syntax that may be written in test code" vs. "syntax that must
not be written" in jest. Referenced from `SKILL.md`.

## Core Principle: Do Not Write Logic in Test Files

A test should consist **solely of assertions** (`expect()` in Jest) that check
simple input and output values. Writing logic to prepare a test inside the test
itself is a well-known testing anti-pattern. Since the test code itself is not
verified, if raw logic written inside the test is wrong, nobody will notice, and
the reliability of the test collapses.

- What a test should verify is "the contract of the member under test," not the
  correctness of preparation logic assembled within the test (this shares the
  same root as the QA stance in [SKILL.md](../SKILL.md)).
- If preparation is needed (e.g. DB seeding), don't write logic inside the test
  — delegate it to an **external mechanism** (a seeder / an already-tested
  tool).

### Example: Preparation Logic Without a Seeder Produces False Positives

If you write a "document deletion" test without using a seeder, you tend to end
up with a procedure like this:

1. Insert the target within the test.
2. Delete the target within the test.
3. Confirm the target does not exist in the DB table (test passes).

This is clearly bad. **Even if the insert did nothing** (the table never
changed), and **even if the delete did nothing**, step 3's "does not exist"
check will still pass. To work around this you'd have to add an extra
`expect()` between steps 1 and 2 to confirm the DB actually changed — but
**deleting the document itself is not the purpose of this test**. The correct
approach is to delegate preparation to a seeder and not bring preparation logic
into the test.

## Only Logic That Is "Verified by a Test" May Be Used

The criterion for whether logic may be used inside a test is: **is that logic
itself tested elsewhere, with its behavior guaranteed?**

```js
// ❌️ Writing untested transformation logic inside the test
array.map(it => it.id)

// ✅️ Using an already-tested class (behavior is guaranteed)
SampleClass.create(...)
```

- ❌️ The transformation `array.map(it => it.id)` is not tested anywhere. If it
  is wrong, nobody will notice.
- ✅️ `SampleClass` has its own separate tests written, so it may be used freely
  inside a test.

### What May Be Used Freely

- **Data expressible as literals** (object/array literals, etc.).
- **Tested factory methods** (e.g. a `create()` that instantiates another
  class).
- **Tested helper functions** (ones extracted into `tests/tools/`, described
  below).

## Do Not Define Helper Functions Inside a Test File (Violation of File Responsibility)

Do not define helper functions inside a test file. Function definitions like
the following also count as "logic."

```js
// ❌️ Defining a helper function inside the test file
function createSamples ({ options }) {
  return new Sample(options)
}
```

- If **no test exists** to confirm the correctness of that function, this
  amounts to using untested logic in a test — not allowed.
- Even **if a test is written, but it lives in the same file** — that is **a
  violation of file responsibility**. A test file's **class name is its file
  name** (`Sample.js` tests the `Sample` class), and its responsibility is
  limited to "testing that class." Writing "a test for a test tool (helper
  function)" inside it clearly mixes responsibilities.

### If a Helper Function Is Truly Necessary

1. **Define** the helper function under `tests/tools/`.
2. **Write tests** for the helper function under `tests/__tests__/test-tools/`,
   **mirroring the directory structure** of `tests/tools/` (the same mirroring
   concept as [directory.md](./directory.md)).
3. Once a helper function has tests this way, it becomes a **tested helper
   function** that may be used freely inside test files.

## Prohibited Syntax

Since a test consists solely of input/output assertions, do not write control
flow, conditional branching, or unverified transformations inside `describe()`.
Specifically, the following are **prohibited**:

- `if` statements
- The ternary operator (`? :`)
- The nullish coalescing operator (`??`)
- Short-circuit evaluation (`||` / `&&`)
- **Higher-order functions** (`Array#map()` / `filter()` / `reduce()`, etc.). The
  moment the argument passed is a **function**, unverified logic can occur
  inside it.
- `Array#forEach()`. ESLint conditionally allows this ("permitted as long as no
  assignment statement is written inside"), but it is **prohibited within test
  files** (iteration should instead be expressed with `expect.each()`, described
  below).

### What Does Not Count as Logic

Code that **references a variable already defined within `describe()` to
construct a new object** is not judged to be logic. Assembling an `args` or a
`cases` element object by referencing a fixture variable is permitted to that
extent (what is prohibited is unverified transformation, branching, and helper
definitions).

```js
// OK: merely constructs an object by referencing a variable defined within describe()
const args = {
  scheme: input.scheme,
  credential,
}
```

## Do Not Apply DRY to Test Code (Define Things Inside Each `describe()`)

**The DRY principle generally does not apply to test code.** Even if the same
value or definition is needed in multiple places, do not consolidate it and
hoist it to file scope (outside all `describe()` blocks). This applies **not
only to assignment statements (variable definitions) but also to class
definitions, function definitions, and so on** — define each of these
**separately inside each `describe()`** that uses it. The **only** thing that
may live at file scope is **imports**.

- **Why DRY does not apply**: When you need to change the value of a shared
  definition, you must **consider the assumptions of every `describe()` that
  depends on it**. For example, if you want to change the value for just one
  method, you can't touch the shared definition, so you end up
  **copy-pasting and modifying it** inside that specific `describe()` anyway. If
  that's how it ends up, **it's correct to define it individually from the
  start**.
- Shared state at file scope is also hard to trace back to which
  `describe()` / `test()` depends on it, creating implicit coupling and
  contamination between tests. Confining it inside each `describe()` lets you
  read that block alone and have the assumptions be self-contained (this also
  aligns with the sticky-header policy of repeating the class-name describe per
  member;
  [structure.md](./structure.md#do-not-define-shared-variables-outside-describe-at-file-scope)).
- Even if generation cost is a concern (e.g. building a derived class via a
  factory), it's fine to define it each time (tests prioritize correctness and
  independence).

```js
// ❌️ Defined at file scope and reused across multiple describe() blocks (applying DRY)
const minValue = 10

describe('SampleClass', () => {
  describe('#calculateAverage()', () => {
    // use minValue here
  })
})

describe('SampleClass', () => {
  describe('#calculateTotal()', () => {
    // use minValue here too
  })
})
```

```js
// ✅️ Define it separately inside each describe() that uses it
describe('SampleClass', () => {
  describe('#calculateAverage()', () => {
    const minValue = 10
    // use minValue here
  })
})

describe('SampleClass', () => {
  describe('#calculateTotal()', () => {
    const minValue = 10
    // use minValue here
  })
})
```

## Use `expect.each()` for Iteration (Replacement for `forEach`)

Instead of `Array#forEach()`, use the Jest extension defined by
`@openreachtech/jest-expect-each` for iteration.

- `expect.each().toXxxx()`
- `expect.each().toXxxx.each()`

`toXxxx` refers to a **standard Jest matcher** (`toBe()` / `toEqual()` /
`toThrow()`, etc.). `expect.each()` is an extension, provided by
`@openreachtech/jest-expect-each`, that lets you apply that standard matcher
repeatedly across multiple values.
