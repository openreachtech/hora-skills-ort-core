# Structure (describe / test structure)

Conventions for assembling and nesting describe / test blocks in Jest.
Referenced from `SKILL.md`. See [naming.md](./naming.md) for variables and naming.

## describe() Structure

Put the class-name describe() on the **outside** and the member describe() on the
**inside**. However, the class-name describe() and the member describe() must
correspond **one-to-one**. In other words, **repeat** the class-name describe() for
each member, and put only **one** member into each class-name describe().

- Do not bundle multiple members into a single class-name describe().
- The nesting order is `describe(class) > describe(member) > describe(behavior) > test()`.
- Name the behavior-layer describe() after the expected behavior using
  **`should [verb]`** (e.g. `describe('should keep property')` /
  `describe('should call constructor')`). Do not use `to [verb]`. However, when the
  behavior **depends on a precondition**, insert `describe('when [precondition]')` and
  express it via `test('should [verb]')` inside it (e.g. `when not inherited` /
  `when called as is`).

### For classes that inherit, place `describe('inheritance')` first

If the class under test `extends` another class, place `describe('inheritance')` as
the **first** class-name describe() to verify that it inherits from the base class.
Place it **before** the other member describe()s.

- Since inheritance does not depend on input, do not use `cases` / `test.each()` —
  place a single `test()`.
- Verify inheritance via the prototype chain (that `SubClass.prototype` is an instance
  of the base class). Use the `toBeInstanceOf()` assertion.
- The nesting order is `describe(class) > describe('inheritance') > test()`.
- Import the base class as well.

```js
import BasicAuthorizationBuilder from '.../concretes/BasicAuthorizationBuilder.js'

import BaseAuthorizationBuilder from '.../BaseAuthorizationBuilder.js'

describe('BasicAuthorizationBuilder', () => {
  describe('inheritance', () => {
    test('should be correct class', () => {
      const received = BasicAuthorizationBuilder.prototype

      expect(received)
        .toBeInstanceOf(BaseAuthorizationBuilder)
    })
  })
})
```

### For throws on abstract members, insert `when not inherited`

When verifying that an abstract member (a static getter / method that throws when
called on the base class as-is, without being overridden by a subclass) throws, insert
`describe('when not inherited')` as the behavior layer to make explicit the
precondition that it was "called **without being inherited (overridden)**".

- Express the calling condition (precondition) with `describe('when not inherited')`,
  and express the behavior with `test('should throw error')` inside it.
- The nesting order is
  `describe(class) > describe(member) > describe('when not inherited') > test()`.
- If you are simply verifying that it throws, `cases` / `test.each()` are unnecessary
  since the throw does not depend on input. Just place a single `test()` inside the
  describe() that expresses the precondition.

```js
describe('BaseAuthorizationBuilder', () => {
  describe('.get:schema', () => {
    describe('when not inherited', () => {
      test('should throw error', () => {
        expect(() => BaseAuthorizationBuilder.schema)
          .toThrow('BaseAuthorizationBuilder.get:schema must be inherited')
      })
    })
  })
})
```

- However, if the thrown error message **varies by derived class** (e.g. it includes
  `this.name` as in `` `${this.name}...` ``), turn that variation into `cases`.
  Prepare the base class and several derived classes with different names, and verify
  that the expected message changes per class. Whether it is acceptable to define a
  derived class follows "When it is acceptable to define a derived class" in
  [mocks.md](./mocks.md) (since a single `describe()`'s `cases` needs several derived
  classes simultaneously, it is acceptable to define them).

```js
describe('BaseAuthorizationBuilder', () => {
  describe('.get:schema', () => {
    describe('when not inherited', () => {
      class AlphaAuthorizationBuilder extends BaseAuthorizationBuilder {}

      class BetaAuthorizationBuilder extends BaseAuthorizationBuilder {}

      const cases = [
        {
          input: { Builder: BaseAuthorizationBuilder },
          expected: 'BaseAuthorizationBuilder.get:schema must be inherited',
        },
        {
          input: { Builder: AlphaAuthorizationBuilder },
          expected: 'AlphaAuthorizationBuilder.get:schema must be inherited',
        },
        {
          input: { Builder: BetaAuthorizationBuilder },
          expected: 'BetaAuthorizationBuilder.get:schema must be inherited',
        },
      ]

      test.each(cases)('Builder: $input.Builder.name', ({ input, expected }) => {
        expect(() => input.Builder.schema)
          .toThrow(expected)
      })
    })
  })
})
```

- If the abstract member is an **instance method** (`#instanceMethod()`, not static),
  **instantiate** it before calling. Since the constructor arguments are irrelevant to
  the verification, fill them with neutral values (the same idea as
  [Isolate the property under test](#isolate-the-property-under-test)). If the message
  varies with `this.constructor.name`, turn the derived classes into `cases` and
  create an instance with `new input.Stringifier(...)`.

```js
describe('BaseRequestBodyStringifier', () => {
  describe('#stringifyBody()', () => {
    describe('when not inherited', () => {
      class AlphaRequestBodyStringifier extends BaseRequestBodyStringifier {}

      class BetaRequestBodyStringifier extends BaseRequestBodyStringifier {}

      const cases = [
        {
          input: { Stringifier: BaseRequestBodyStringifier },
          expected: 'BaseRequestBodyStringifier#stringifyBody() must be inherited',
        },
        {
          input: { Stringifier: AlphaRequestBodyStringifier },
          expected: 'AlphaRequestBodyStringifier#stringifyBody() must be inherited',
        },
        {
          input: { Stringifier: BetaRequestBodyStringifier },
          expected: 'BetaRequestBodyStringifier#stringifyBody() must be inherited',
        },
      ]

      test.each(cases)('Stringifier: $input.Stringifier.name', ({ input, expected }) => {
        const stringifier = new input.Stringifier({
          replacer: null, // Neutral value. Since throwing is what's under test, the value can be anything
        })
        const args = {
          value: {},
        }

        expect(() => stringifier.stringifyBody(args))
          .toThrow(expected)
      })
    })
  })
})
```

### For fixed-value static getters / static properties, insert `when called as is`

When verifying a static getter that returns a fixed value (constant), or a static
property initialized with a fixed value (e.g. `static schema = 'Basic'` /
`static pool = new WeakMap()`), insert `describe('when called as is')` as the behavior
layer to make explicit the precondition that it was "**referenced as-is**". Place a
single `test('should ...')` inside it and assert the value (whether you write the
value directly in the matcher or bind it to `const expected` is decided by the rule in
[aaa-pattern.md](./aaa-pattern.md#write-expected-value-literals-inline-if-single-line-bind-to-const-expected-if-multi-line)).

- Since fixed values do not depend on input, do not use `cases` / `test.each()`. Do not
  create a `cases` array with `input` / `expected`. Whether the expected value is
  written directly in the matcher or bound to `const expected` is decided by whether
  the value is single-line or multi-line
  ([aaa-pattern.md](./aaa-pattern.md#write-expected-value-literals-inline-if-single-line-bind-to-const-expected-if-multi-line)).
- Use the same form for a static property (not a getter) too. The member describe
  notation follows [naming.md](./naming.md): `.staticProperty` (getters are
  `.get:staticGetter`). `when called as is` may be used as-is for property references
  too.
- The nesting order is
  `describe(class) > describe(member) > describe('when called as is') > test()`.
- Only when there is an exceptional reason (the getter's value actually varies with
  state, etc.) may you drop this form for `cases` / `test.each()`.

```js
describe('BasicAuthorizationBuilder', () => {
  describe('.get:schema', () => {
    describe('when called as is', () => {
      test('should be fixed value', () => {
        const received = BasicAuthorizationBuilder.schema

        expect(received)
          .toBe('Basic')
      })
    })
  })
})
```

#### If the fixed value is an "object", pin it by type (`toBeInstanceOf`)

If the fixed value is a **primitive** (string, number, etc.), pin the exact value
itself with `test('should be fixed value')` + `toBe(<fixed value>)`. On the other
hand, if the fixed value is an **object such as `WeakMap` / `Map` / `Set`**, the
reference is unique per instance and you cannot write `toBe(<literal>)`, so **pin it
by type** instead.

- The behavior (`test`) name expresses the **type**, not the value
  (`should be a WeakMap`).
- Use `toBeInstanceOf(<type>)` as the assertion, not `toBe()`.
- This applies to cases where a static property initializes a memoization pool, etc.
  Memoization behavior such as "the same reference is returned on every call" is
  verified on the **method side** that uses it, not on the property alone
  (see [Methods that memoize and return the same reference](#methods-that-memoize-and-return-the-same-reference-should-be-memoized)).

```js
describe('BoundCtorRegistry', () => {
  describe('.BoundCtorPool', () => {
    describe('when called as is', () => {
      test('should be a WeakMap', () => {
        const received = BoundCtorRegistry.BoundCtorPool

        expect(received)
          .toBeInstanceOf(WeakMap)
      })
    })
  })
})
```

### Instance getters require `test.each()` (the variable element is the instance)

An instance getter (e.g. `get Ctor () { return this.constructor }`), or a getter whose
value **varies depending on the instance's state or type**, is not a "fixed value". Do
not reduce it to a single `test()` under `when called as is` — instead, put the
**variable element (i.e. which instance it is) into `cases` and drive it with
`test.each()`**.

- Do not shrink it to a single `test()` just because "the base class currently being
  verified always yields the same value". That is a **shortcut justified by the
  implementation** — it would even pass with a hardcoded implementation like
  `return BoundCtorRegistry`, and would fail to verify that the getter returns
  `this.constructor` (the contract) (see the QA principle in
  [SKILL.md](../SKILL.md)).
- For an instance getter, the variable element is "which class the instance is an
  entity of." Turn instances of the **base class and several derived classes** into
  cases, and confirm the getter returns the correct value for each. Since several
  derived classes are needed simultaneously, it is acceptable to define them following
  "When it is acceptable to define a derived class" in [mocks.md](./mocks.md).
- A single `test()` (`when called as is`) is permitted only for the case above —
  [fixed-value static getters / static properties](#for-fixed-value-static-getters--static-properties-insert-when-called-as-is)
  — which return a constant **regardless of input or state**.

```js
describe('BoundCtorRegistry', () => {
  describe('#get:Ctor', () => {
    class SampleBaseCtor {}

    class AlphaBoundCtorRegistry extends BoundCtorRegistry {}
    class BetaBoundCtorRegistry extends BoundCtorRegistry {}

    describe('should be constructor of instance', () => {
      const cases = [
        {
          input: {
            Registry: BoundCtorRegistry,
          },
          expected: BoundCtorRegistry,
        },
        {
          input: {
            Registry: AlphaBoundCtorRegistry,
          },
          expected: AlphaBoundCtorRegistry,
        },
        {
          input: {
            Registry: BetaBoundCtorRegistry,
          },
          expected: BetaBoundCtorRegistry,
        },
      ]

      test.each(cases)('Registry: $input.Registry.name', ({ input, expected }) => {
        const registry = new input.Registry({
          BaseCtor: SampleBaseCtor,
        })

        const received = registry.Ctor

        expect(received)
          .toBe(expected) // Same reference
      })
    })
  })
})
```

**Why repeat it (important)**: This is an optimization for the human reader. Even as
the test file grows long, a class-name describe() always sits directly above every
member describe(), so no matter where you stop while fast-scrolling, "which class and
what am I looking at" is always visible on screen. If a single class describe() wraps
all members, the class context scrolls off screen the moment you pass the top, and you
need to scroll back up to tell which class you're in. Think of it as achieving a
"sticky header" through syntax. We accept that the class name is duplicated in the
Jest output, in favor of readability.

Correct example:

```js
describe('PathnameBuilder', () => {
  describe('constructor', () => {
    describe('should keep property', () => {
      test.each(cases)('...', () => { /* ... */ })
    })
  })
})

describe('PathnameBuilder', () => {
  describe('.create()', () => {
    describe('should be an instance of own class', () => {
      test.each(cases)('...', () => { /* ... */ })
    })
  })
})

describe('PathnameBuilder', () => {
  describe('#buildPathname()', () => {
    describe('should interpolate placeholder', () => {
      test.each(cases)('...', () => { /* ... */ })
    })
  })
})
```

Incorrect example (a single class-name describe() bundling multiple members):

```js
describe('PathnameBuilder', () => {
  describe('constructor', () => { /* ... */ })
  describe('.create()', () => { /* ... */ })
  describe('#buildPathname()', () => { /* ... */ })
})
```

### Methods that take arguments require `test.each()` (the variable element is the argument)

For a method whose output **depends on its arguments** (static or instance, whether
`create(args)` / `buildSchema(args)` — anything whose result is determined by the
value passed in), you must **not** fix a single input in a single `test()`. Turn the
argument into the variable element as `cases`, and drive **two or more inputs** with
`test.each()`. With only one input, an implementation that ignores the argument and
hardcodes the value would still pass (the QA stance in [SKILL.md](../SKILL.md)).
Whereas [Instance getters require `test.each()`](#instance-getters-require-testeach-the-variable-element-is-the-instance)
has "the variable element is the instance," here "the variable element is the
argument."

- Even when you insert `describe('when ...')` per branch (dispatch), the inside must
  still use **`test.each()` with ≥2 inputs**. For `when schema is array`, pass two or
  more different arrays; for `when value is scalar constructor`, pass two or more
  different constructors — showing that the argument actually has an effect.
- A single `test()` is permitted only for behavior that **does not depend on input**:
  [when not inherited](#for-throws-on-abstract-members-insert-when-not-inherited) (throwing
  does not depend on arguments) /
  [when called as is](#for-fixed-value-static-getters--static-properties-insert-when-called-as-is)
  (fixed value) / inheritance / default-value filling when called with no arguments,
  etc.
- If an argument is passed but **that axis is a neutral collaborator uninvolved in the
  behavior**, drive the axis that *is* involved (a different argument/property) with
  `test.each()` and fix only the neutral argument.

```js
// Bad: fixing a method that takes arguments to a single input
describe('when schema is array', () => {
  test('should build array schema', () => {
    const received = SomeClass.buildSchema({ schema: [Alpha, Beta] })
    // Would still pass even with a hardcoded `return [Alpha, Beta]`
  })
})

// Good: drive the argument with test.each (2+ inputs)
describe('when schema is array', () => {
  const cases = [
    {
      input: {
        schema: [Alpha, Beta],
      },
      expected: [Alpha, Beta],
    },
    {
      input: {
        schema: [Gamma, Delta],
      },
      expected: [Gamma, Delta],
    },
  ]

  test.each(cases)('schema: $input.schema', ({ input, expected }) => {
    const received = SomeClass.buildSchema(input)

    expect(received)
      .toEqual(expected)
  })
})
```

## When the subject is not a class

The rule above fixes level 1 to a class name and level 2 to a member name. Some
tests have neither: a module that exports constants, or a data file such as a
message catalogue. The index rule still holds — only what is indexed changes.

### Level 1 names what the module default-exports

The class rule reads level 1 off the class because **the class is what the file is
about**. A module is the same question with a different answer, and the answer comes
from what it default-exports:

| `export default` is | Level 1 |
| :-- | :-- |
| A class | The class's name |
| A function | The function's name |
| A `const` this file declares | That binding's name |
| An import binding whose name matches the file it came from | That binding's name |
| A value with no name of its own | **A name chosen for what the file is about** |

A module with no default export at all falls in the last row too.

- **The first four rows are not the file's basename.** They coincide in practice,
  because a file is named after what it holds, but the anchor is the definition
  rather than the filename. Rename the file and the describe does not move.
- **Borrow the identifier wherever there is one to borrow.** A class, a function
  and a `const` are all names the source already chose, and a name the source chose
  is one a reader can search for. The rows differ in what is exported, not in how
  the level is decided.
- **An import binding is borrowed only while it still names its source.**
  `import core from './rules/core.js'` passes something along under the name it
  already had — the binding and the file agree, so the name was not invented here.
  `import config from './rules/core.js'` renames on the way in, and that name is
  this file's private label for somebody else's export: it names nothing a reader
  can trace, so level 1 falls to the last row.
- **The last row is a judgement, and that is deliberate.** `export default { ... }`
  written inline has no identifier at all — `index` or `constants-error` names the
  file, not the subject. Choose the name a reader would use for what the file is
  about.

**The judgement is only affordable because the test file mirrors the source path**
([directory.md](./directory.md#directory-structure)). Somebody who edits the source
reaches its test through the path, so level 1 is what they read once they arrive,
not what they search for. A test file named by convention rather than by mirroring
takes that away, and then neither end can be derived.

```js
describe('constants-error', () => {
  describe('ERROR_CODE_HASH', () => {
    describe('should map every code to a message', () => {
      test.each(cases)('code: $input.code', ({ input, expected }) => {
        // ...
      })
    })
  })
})
```

### A data file is indexed by its path

When the subject is a data file — a message catalogue, a fixture set, a generated
manifest — level 1 is **the data file's path**: written from the mirroring base,
carrying the extension as the source spells it, with a varying segment written as
`*`.

The varying segment is whatever a family of files differs by, and nothing about
the rule is tied to what that is: `i18n/locales/*/message.json`,
`fixtures/*/user.json`, `manifests/*.json`.

Level 2 is one of two things.

**The key path of the part under assertion**, when the assertion is about one part
of the file:

```js
describe('i18n/locales/*/message.json', () => {
  describe('message.form', () => {
    describe('should hold the same keys in every file', () => {
      test.each(cases)('locale: $input.locale', ({ input, expected }) => {
        // ...
      })
    })
  })
})
```

**A bare noun phrase naming the reading**, when the assertion is about the file as
a whole and no part can be named. The precondition layer takes `when …` and the
assertion moves into `test()`, as it does for a fixed value:

```js
describe('i18n/locales/*/message.json', () => {
  describe('key set', () => {
    describe('when every file is read', () => {
      test('should be identical across them', () => {
        // ...
      })
    })
  })
})
```

- **The key path comes first.** Reach for a noun phrase only when the assertion
  covers the whole file.
- **Why the path sits at level 1**: for a class the target of a fix is a member,
  but for a data file it is the file, and what a person greps is its path. Level 1
  therefore carries the index, and level 2 says which reading of the file is
  asserted.
- **Why a bare noun phrase**: every class and module member carries a sigil —
  `#`, `.`, or an upper-case export name — so a lower-case noun phrase at level 2
  marks a reading of a data file rather than a definition, without a new symbol.
- **Do not borrow `.` for a reading.** `.keys` claims a static property that does
  not exist, and whoever greps for it finds nothing. `.` is for definitions only.
- One reading per path describe(), repeated per reading — the same consequence
  the class rule has.

### A reconciliation is indexed by the relation

Some tests have no single subject at all. What is under test is **whether two collections
still agree** — the identifiers a dependency publishes against the configuration that
declares each one, a schema against the fixtures built for it, an enum against the table it
is stored in. Neither side is the subject; the agreement between them is.

Level 1 names the relation. Level 2 names each side of it, one describe per collection,
plus one for the catalogue they are reconciled against.

```js
describe('rule coverage', () => {
  describe('core rules', () => {
    // the declarations carrying the error severity
  })

  describe('deprecated rules', () => {
    // the declarations carrying the off severity
  })

  describe('plugin rules', () => {
    // the catalogue both are reconciled against
  })
})
```

- **Level 1 is a noun phrase naming the reconciliation**, and carries no `#` or `.` — the
  same reason a data file's reading does ([naming.md](./naming.md#notation-when-the-subject-is-not-a-class)).
- **Level 2 names a collection**, in the plural of what it holds. There is nothing to grep
  for at either level, and that is the point: no definition owns this test.
- This is the one shape where the index rule **cannot** be satisfied. Satisfying it
  would mean electing one of the two sides as the subject, and the election is wrong
  either way — the test fails when the two disagree, not when either one is wrong.

#### Close the chain end to end

A reconciliation is worth writing only if **removing one entry anywhere makes it fail.**
Lay out the seams and put a check on each:

| Seam | What it catches |
| :-- | :-- |
| A case's own fields against each other | A typo in the data every other test trusts |
| Each case against the declaration | A declaration missing, or carrying the wrong severity |
| The **unique** case count against the declaration count | A case dropped from the array, or written twice |
| The declaration total against the catalogue | An entry the catalogue holds and nothing declares |

Miss one seam and a gap opens that every remaining test passes over. Without the third,
deleting a case shrinks the coverage silently — the cases that remain all still pass, and
the total still reconciles.

**Verify both directions before believing the chain is closed.** Delete an entry on the
catalogue side and confirm the failure; then delete one on the declaration side and confirm
it again. A chain that only catches one direction is half a chain, and which half is
missing is not visible from reading it.

**Count the entries left after duplicates are dropped, never `cases.length`.** Write the
same identifier into two rows and the length still matches, while the entry it displaced
goes unverified — and a duplicate is not something a reader finds. In a table of seventy
rows, whether two of them are the same cannot be seen by looking.

Dropping duplicates is a transformation, so it does not belong inside the test
([anti-pattern.md](./anti-pattern.md)). Put it in a helper under `tests/tools/`, give the
helper its own test
([directory.md](./directory.md#test-tools-where-to-place-helper-functions)), and call it
from the assertion.

```js
const received = extractUniqueIds({ values: cases })

expect(received)
  .toHaveLength(Object.keys(core.rules).length)
```

**The data's own consistency is a behavior of its own**, and goes in its own behavior
describe rather than folded into the one that checks the declaration.

```js
describe('should prefix each id with the plugin namespace', () => {
  test.each(cases)('input: $input', ({ input, expected }) => {
    const received = `plugin/${input}`

    expect(received)
      .toBe(expected)
  })
})

describe('should declare each rule of the plugin as an error', () => {
  // the same cases, checked against the declaration
})
```

- **What is under test here is the case data itself**, which is why it reads oddly at
  first: nothing of the subject is called, and the assertion compares two literals through
  one interpolation. That is the point. Every other seam trusts these literals, and whether
  a literal is right cannot be seen by reading it — a namespace misspelt in one row of
  seventy looks exactly like the sixty-nine that are correct.
- **Folding it into the declaration check hides which of the two failed.** Split, a failure
  names its own cause: either the data is wrong, or the declaration is. Together, every
  failure looks like a missing declaration, and the first move is to go looking in the
  wrong file.

#### Reconcile totals by summing the parts, never by merging them

Where the declarations are split across several collections, compare the catalogue's count
against **the sum of the parts**. Do not spread them into one object and count its keys.

```js
// Good: summed, so an entry declared in both collections is counted twice and fails
const expected = Object.keys(core.rules).length
  + Object.keys(deprecated.rules).length

const received = Object.keys(plugin.rules)

expect(received)
  .toHaveLength(expected)
```

```js
// Avoid: a merge collapses a key present in both, and the total matches
//        even though that entry is declared twice
const merged = Object.keys({
  ...core.rules,
  ...deprecated.rules,
})
```

A merge is the natural way to write "everything declared," which is exactly why it is the
trap. It answers *how many distinct things are declared*, and the question being asked is
*how many declarations there are*. The two differ by precisely the bug worth catching.

#### For an empty collection, leave the describe out

When a collection is **empty** — a deprecated set with nothing in it yet — do not
write its describe around an empty `cases`. **`test.each([])` throws**, so the file
fails to run at all rather than reporting a passing empty group. Leave the describe
out, and let the total reconciliation cover that side: zero sums correctly.

- The describe comes back the moment the collection has an entry.
- This is not licence to drop a describe whose cases are merely few. A single element keeps
  it — the "two or more" guidance in [test-cases.md](./test-cases.md) yields here, because
  what fixes the count is the collection, not the author.

## Define shared fixtures directly under the member describe

If the same fixture variable (e.g. `alphaReplacer`) is used by **multiple behavior
describes** under the same member, define it **once, directly under the member
(second-level) describe**, and share it. Do not repeat the same declaration for each
behavior.

- A variable used by only one behavior belongs inside that behavior's describe (do not
  force-hoist it). Only lift it up to the member level when sharing actually occurs.
- **When one behavior audits another's data, sharing is not a convenience — it is what
  makes the audit true.** A `cases` array that a sibling describe counts, or checks for
  completeness, has to be **the same array**, declared once where both can reach it. Give
  each describe its own copy and the audit checks a copy of itself, while the array
  actually driving the tests goes unchecked; the two then drift with nothing to reveal it.
  A `const` declared inside one describe is not visible from a sibling, so the choice here
  is never "share or repeat" — it is **share or verify nothing**.

```js
describe('BaseRequestBodyStringifier', () => {
  describe('.create()', () => {
    /** @type {(key: string, value: *) => *} */
    const alphaReplacer = (key, value) => value

    const betaReplacer = ['id', 'name']

    describe('should be an instance of own class', () => {
      const cases = [
        // Shares alphaReplacer / betaReplacer / null
      ]
      // ...
    })

    describe('should call constructor', () => {
      const cases = [
        // Shares the same alphaReplacer / betaReplacer
      ]
      // ...
    })
  })
})
```

## Do not define shared variables outside `describe()` (at file scope)

A variable used across multiple `describe()`s must **not** be defined outside all `describe()`s (i.e. at file scope
/ module top level) — for example, placing a derived binding at the top of the file and reusing it across every
describe. Only imports are allowed at file scope. A variable you want to share must be **defined fresh inside each
`describe()` that uses it** (when sharing across behaviors within the same member, do so [once directly under the
member describe](#define-shared-fixtures-directly-under-the-member-describe); when sharing across members, redefine
it in each member describe).

- **Why**: File-scope shared state is hard to trace — it's unclear which `describe()`
  / `test()` depends on it — and it invites implicit coupling and contamination
  between tests. Confining it inside the describe block means reading just that block
  is enough to know the full precondition (this also aligns with the sticky-header
  policy of repeating the class-name describe per member).
- Even if you're worried about the cost of creation (e.g. building a derived class via
  a factory), it's fine to define it fresh each time (it is memoized / correctness is
  the priority in tests).
- Imports (the class under test, the base class, test declaration classes, etc.) are
  side-effect-free bindings, so they are fine at file scope.

```js
// Bad: a shared binding at file scope (outside all describes)
const BoundClass = SomeClass.of(SomeCollaborator)

describe('SomeClass', () => {
  describe('#alpha()', () => {
    // Uses BoundClass
  })
})

describe('SomeClass', () => {
  describe('#beta()', () => {
    // This one also uses BoundClass
  })
})
```

```js
// Good: define it in each describe that needs it (imports remain at file scope)
import SomeCollaborator from '.../SomeCollaborator.js'

describe('SomeClass', () => {
  describe('#alpha()', () => {
    const BoundClass = SomeClass.of(SomeCollaborator)
    // ...
  })
})

describe('SomeClass', () => {
  describe('#beta()', () => {
    const BoundClass = SomeClass.of(SomeCollaborator)
    // ...
  })
})
```

## Constructor Tests

In the constructor's `should keep property`, insert one `describe('#<propertyName>')`
**per** property retained, and write `cases` / `test.each()` inside it.

- If there are multiple properties, line them up as `#alpha`, `#beta`, and so on.
- The nesting order is
  `describe(class) > describe('constructor') > describe('should keep property') > describe('#<property>') > test()`.
- Use `toHaveProperty()` as the assertion, not `toBe()`.

```js
test.each(cases)('alpha: $input.alpha', ({ input, expected }) => {
  const instance = new SomeClass(input)

  expect(instance)
    .toHaveProperty('alpha', expected)
})
```

```js
describe('PathnameBuilder', () => {
  describe('constructor', () => {
    describe('should keep property', () => {
      describe('#alpha', () => {
        const cases = [
          // ...
        ]

        test.each(cases)('alpha: $input.alpha', ({ input, expected }) => {
          // ...
        })
      })

      describe('#beta', () => {
        // ...
      })
    })
  })
})
```

### Isolate the property under test

Even when a constructor takes multiple arguments, put **only the property under
test** into the `input` of each `#<property>`'s `cases`. Do not write other required
arguments into `cases` — fill them with a **neutral value** (such as an empty string)
when assembling `args` in the test body.

- Do not mix unrelated arguments into `input` (do not write `credential` into the
  `#scheme` cases).
- Assemble the argument object passed to the constructor as `args` inside the test,
  filling the target property from `input` and everything else with a default value.

**Why**: To keep clear, for each property's test, what is being varied and what is
being verified. It minimizes `cases`, and makes the variable under test obvious at a
glance.

#### If there is no addition/removal of properties, pass `input` as-is

Assembling an argument object (`args`; see [naming.md](./naming.md#naming-the-args-object))
is only necessary **when you need to fill in unrelated required arguments with neutral
values** (i.e. when the properties of the argument passed exceed those in `input`). If
the properties of the argument passed to the target under test (constructor / method)
**match `input` exactly, with nothing missing or extra**, pass `input` directly rather
than reassigning it into `args`.

- Do not create an intermediate variable like `const args = { source: input.source }`
  that just maps `input` 1:1 (it's redundant and duplicates management with `input`).
- Only assemble `args` when there is something to fill in, mixing `input`'s values with
  defaults.

```js
// Good: the argument matches input, so pass input directly
test.each(cases)('source: $input.source', ({ input, expected }) => {
  const received = BasicAuthorizationBuilder.generateCredential(input)

  expect(received)
    .toBe(expected)
})
```

```js
// Avoid: creating args that just copy input
test.each(cases)('source: $input.source', ({ input, expected }) => {
  const args = {
    source: input.source,
  }

  const received = BasicAuthorizationBuilder.generateCredential(args)

  expect(received)
    .toBe(expected)
})
```

```js
describe('#scheme', () => {
  const cases = [
    {
      input: { scheme: 'Bearer' },
      expected: 'Bearer',
    },
    {
      input: { scheme: 'Basic' },
      expected: 'Basic',
    },
  ]

  test.each(cases)('scheme: $input.scheme', ({ input, expected }) => {
    const args = {
      scheme: input.scheme,
      credential: '', // Fill the unrelated required argument with a neutral value
    }

    const builder = new BaseAuthorizationBuilder(args)

    expect(builder)
      .toHaveProperty('scheme', expected)
  })
})
```

#### If `input` mixes properties and arguments, split into `args` (`constructorArgs` / `<methodName>Args`)

If a single `input` contains **both a class's (constructor) property and an argument
passed to the method under test**, you must **not** pass `input` as-is — splitting
into an `args` per destination is **mandatory**. Name the variables per the convention
in [naming.md](./naming.md#naming-the-args-object):
**destination member name + `Args`** (the constructor's share is `constructorArgs`,
the method's share is `<methodName>Args`). Do not use generic, role-based names like
`propertyArgs` / `methodArgs`.

- Do not pass the entirety of `input` directly to the constructor or method (mixing
  properties and arguments makes it unreadable which one affects which).
- Do not lump them into a single ambiguous `args` — split and name by destination
  member (constructor → `constructorArgs`, `buildPathname()` → `buildPathnameArgs`; if
  there is a third recipient, add `<memberName>Args` in the same way).
- This is an exception to
  ["If there is no addition/removal of properties, pass `input` as-is"](#if-there-is-no-additionremoval-of-properties-pass-input-as-is).
  Since the destination splits into two or more, a direct pass is not possible.
- Splitting and naming by member is about the case where **values mixed into a single
  `input` need to be built separately for the constructor and for the method** (i.e.
  when there are multiple builds). If a single build suffices, keep it as `args`
  ([naming.md](./naming.md#naming-the-args-object)).
- In a double loop, the outer = constructor property and the inner = method argument
  usually have **separate sources**, so in most cases there is no need for multiple
  builds. If the outer `input` matches the constructor 1:1, pass it **directly** as
  `new SomeClass(input)`, and the only build needed is a single `args` for the inner
  argument (no member-name split needed in this case). Only split by member name when
  two or more builds are actually needed (e.g. the constructor side also needs
  neutral-value filling and thus a build).
- **If both the property and the argument are involved in the behavior, you must not
  use a single loop.** Follow
  [Constructor Property × Method Argument (double loop)](#constructor-property--method-argument-double-loop)
  and lay out a 2×2 or larger grid with `describe.each()` × `test.each()`. A single
  loop (pairing one property value with one argument value, one pair at a time) is
  permitted only when **one axis is a neutral collaborator uninvolved in the
  behavior** (e.g. only the target property is verified, and the method argument is
  fixed at a neutral value).

```js
test.each(cases)('...', ({ input, expected }) => {
  const constructorArgs = {
    templatePathname: input.templatePathname,
  }
  const buildPathnameArgs = {
    valueHash: input.valueHash,
  }

  const builder = new PathnameBuilder(constructorArgs)

  const received = builder.buildPathname(buildPathnameArgs)

  expect(received)
    .toBe(expected)
})
```

## Factory Method Tests

For a factory method like `.create()`, test only the following:

- That it returns an instance of its own class (`toBeInstanceOf`).
- That it delegates to the constructor (verify `toHaveBeenCalledWith` via
  `constructorSpy`).

Do **not** test property retention. It is the constructor's responsibility, covered
by the constructor's own tests — do not duplicate it here.

### Verify default values on omitted arguments by "the default value reaching the constructor"

When a factory fills in a default value for an omitted argument (e.g.
`create ({ replacer = null } = {})`), split out `describe('should fill default <name>')`
and verify — via `constructorSpy` — that the constructor is called with the default
value when the factory is **called with no arguments**. Here too, do not look at the
return value's properties (property retention is the constructor's responsibility).

- When there is **only one** omittable argument, since the default value does not
  depend on input, do not use `cases` / `test.each()` — place a single
  `test('with no arguments')` (see the example below).
- When there are **two or more** omittable arguments, a single `test()` is not enough.
  Follow
  [If there are multiple omittable arguments, run all combinations including omission (2ⁿ − 1)](#if-there-are-multiple-omittable-arguments-run-all-combinations-including-omission-2ⁿ--1).

```js
describe('should fill default replacer', () => {
  test('with no arguments', () => {
    const expected = {
      replacer: null,
    }

    const SpyClass = constructorSpy.spyOn(BaseRequestBodyStringifier)

    SpyClass.create()

    expect(SpyClass.__spy__)
      .toHaveBeenCalledWith(expected)
  })
})
```

### If there are multiple omittable arguments, run all combinations including omission (2ⁿ − 1)

When there are **multiple** omittable arguments, the "call with no arguments" single
`test()` above can verify only **one** case — omitting everything. That would miss
coupling bugs where the default value of one argument "only takes effect when another
argument is omitted/specified (or interferes with it)". To show that each argument's
default-value filling takes effect **independently** of the others, vary each argument
across the two states **specified (present) / omitted (absent)**, and turn **every
combination in which at least one is omitted** into `cases`. **The single case where
everything is specified fills in no default values at all**, so exclude it (that is
covered by the delegation test in the "should call constructor" section above). With
**n** arguments, there are **2ⁿ − 1** cases (with 3, as in
`create ({ a = null, b = null, c = false })`, that's **7**).

- A single `test('with no arguments')` suffices only when there is **one** omittable
  argument. With **two or more**, enumerate 2ⁿ − 1.
- Do not name the `describe()` after an individual argument — name it after
  default-value filling **as a whole**, as in
  `describe('should fill default value')`.
- Put **only the arguments being specified** into `input`; for omitted arguments,
  **comment them out** — the same technique as
  [making missing properties explicit](#separate-valid--invalid-values) — leaving
  behind the intent "this falls back to the default."
- Write `expected` as the **complete shape** that reaches the constructor (specified
  arguments keep their value, omitted arguments take the default), and verify it with
  `constructorSpy`'s `toHaveBeenCalledWith(expected)`.
- Give specified arguments values that are **unique and different from the default**
  (if the value matches the default, you would miss a bug where a specified value is
  ignored in favor of the default). Since each argument only has the two states
  present/absent, the values themselves will **inevitably repeat** across combinations
  (the uniqueness convention in [test-cases.md](./test-cases.md) does not apply here —
  what identifies a case is not the value but the **combination**).
- Since `input` is sparse, name the `test.each()` titles by **each value in
  `expected`**.
- Order per [list valid values first](./test-cases.md#list-normal-values-first): put cases with
  **more items specified (present) first**, and put the **all-omitted (`input: {}`)** case last.
  All-omitted is the most trivial edge case, so it does not go first (reducing from 2 specified →
  1 → 0 reads more naturally).

```js
describe('SomeClass', () => {
  describe('.create()', () => {
    describe('should fill default value', () => {
      const cases = [
        {
          input: {
            alpha: 100001,
            beta: 100002,
            // gamma: omitted → default null
          },
          expected: {
            alpha: 100001,
            beta: 100002,
            gamma: null,
          },
        },
        // ... the remaining 2 combinations with 2 specified, then 3 with 1 specified, continuing as the specified count decreases
        {
          input: {
            // alpha: omitted → default null
            // beta: omitted → default null
            gamma: 100003,
          },
          expected: {
            alpha: null,
            beta: null,
            gamma: 100003,
          },
        },
        {
          input: {
            // alpha: omitted → default null
            // beta: omitted → default null
            // gamma: omitted → default null
          },
          expected: { // all-omitted is an edge case, so it goes last
            alpha: null,
            beta: null,
            gamma: null,
          },
        },
        // 2³ − 1 = 7 combinations (all-specified is excluded since it fills no defaults; covered by should call constructor)
      ]

      test.each(cases)('alpha: $expected.alpha, beta: $expected.beta, gamma: $expected.gamma', ({ input, expected }) => {
        const SpyClass = constructorSpy.spyOn(SomeClass)

        SpyClass.create(input)

        expect(SpyClass.__spy__)
          .toHaveBeenCalledWith(expected)
      })
    })
  })
})
```

## Separate Valid / Invalid Values

When testing a method, if the argument values include a mix of **valid values (normal
values)** and **invalid values (abnormal values)**, separate them with two
`describe()`s to make the intent clear.

- Valid values: the value is valid and is processed as expected (e.g. a placeholder
  gets interpolated).
- Invalid values: the value is invalid and falls back (e.g. a missing key, `null`, or
  `undefined` becomes an empty string).
- Do not mix both into a single `describe()` (or `cases` / `~Cases`).
- For a case representing "a missing property", leave the missing property **commented
  out** to make explicit what is being omitted (do not simply leave it out silently).

```js
// Make the missing property explicit
{
  valueHash: {
    postId: 10,
    // commentId: 20,
  },
  expected: '/posts/10/comments/',
}
```

```js
describe('#buildPathname()', () => {
  describe('with valid values', () => {
    // Only valid values that get interpolated
  })

  describe('with invalid values', () => {
    // Only invalid values that fall back to an empty string
  })
})
```

## Split out recursive / nested structures into a dedicated `describe()`

If a member processes a **recursive (self-nesting) structure** (a definition
containing sub-definitions, a tree structure, an array/object nesting further
arrays/objects, etc.), **split the recursive (nested) case out into a dedicated
`describe()`** and verify it explicitly. Do not settle for flat (single-level) input
alone.

- **Why**: With only single-level input, it would still pass even if the
  implementation **does not actually recurse into the nesting and instead flattens it
  shallowly**. Verifying that recursion actually takes effect (each nested layer is
  processed correctly) requires input nested **at least two levels deep** (the QA
  stance in [SKILL.md](../SKILL.md)).
- Name the dedicated describe to indicate recursion (e.g. `describe('with nested ...')`
  / `describe('when nested')`). Since this is an axis **independent of** the
  valid/invalid split, do not mix it into the same describe as the flat cases — split
  it out for recursion to make the intent stand out.
- For recursive cases, pin the fact that the **output is also nested** via the nested
  structure of `expected` (use a value that would not match a shallow flattening). A
  depth of two levels is usually sufficient, but if there are variations of nesting
  (an array within an array vs. an array within an object, etc.), place several
  representative cases.

```js
describe('SomeClass', () => {
  describe('#normalize()', () => {
    describe('with valid values', () => {
      // Flat (single-level) input
    })

    describe('with nested values', () => { // Split out recursion
      const cases = [
        {
          input: {
            value: {
              child: {
                id: 100001,
              },
            },
          },
          expected: {
            child: {
              id: 100001,
            },
          },
        },
        // ... other nesting shapes (an array within an object, an array of arrays, ...)
      ]

      test.each(cases)('value: $input.value', ({ input, expected }) => {
        const received = SomeClass.create(input).normalize()

        expect(received)
          .toEqual(expected)
      })
    })
  })
})
```

## Boolean-like methods (separating truthy / falsy)

For a method whose return value is boolean (or can be judged as truthy / falsy), do
not lump it into a single `cases` with `expected: true / false`. Instead, separate it
with `describe('should be truthy')` / `describe('should be falsy')`, and within each
describe, **omit `expected`** and fix the assertion to `toBeTruthy()` / `toBeFalsy()`.

- Which result is being verified is made explicit by the describe name, making the
  intent easier to read.
- `expected` becomes unnecessary, so `cases` only needs `input` (or `~Cases`).

```js
describe('#isValid()', () => {
  describe('should be truthy', () => {
    const cases = [
      { input: { value: 100001 } },
      { input: { value: 100002 } },
    ]

    test.each(cases)('value: $input.value', ({ input }) => {
      const instance = SomeClass.create(input)

      const received = instance.isValid()

      expect(received)
        .toBeTruthy()
    })
  })

  describe('should be falsy', () => {
    const cases = [
      { input: { value: null } },
      { input: { value: undefined } },
    ]

    test.each(cases)('value: $input.value', ({ input }) => {
      const instance = SomeClass.create(input)

      const received = instance.isValid()

      expect(received)
        .toBeFalsy()
    })
  })
})
```

## Methods that memoize and return the same reference (`should be memoized`)

For a method that, when called multiple times with the same argument, returns the
**same instance (same reference)** (memoization, caching, pooling — e.g. storing in a
`WeakMap` and returning the stored value from the second call onward), verify that "the
same reference is returned every time." Express the behavior layer with
`describe('should be memoized')`.

- **Bind the first call's return value to `expected` in Arrange**, and treat the second
  call's return value as `received` (Act), verifying **the same reference** with
  `toBe(expected)`. Append `// same reference` at the end of the assertion line (per
  the same-reference comment convention in [aaa-pattern.md](./aaa-pattern.md)).
- Even when the expected value is generated at runtime (the result of the first call),
  since it is "the value we expect," **bind it to `expected`** (do not use an alias
  such as `first` / `initial`). This lets you express the comparison of two calls
  while still satisfying [aaa-pattern.md](./aaa-pattern.md)'s rule that "only
  `expected` and `tally` may be passed to the matcher."
- There is no static expected-value literal, so `cases` only needs **`input`**. Place
  several cases with varying arguments to show that memoization takes effect for any
  key.
- "What is returned" (a `WeakMap`, an instance, etc.) is pinned earlier, in a separate
  behavior from memoization (e.g. `describe('should be a WeakMap')`), using
  `toBeInstanceOf()`. Do not mix the type check and the memoization check into a single
  test.

```js
describe('BoundCtorRegistry', () => {
  describe('.ensureCtorPool()', () => {
    describe('should be a WeakMap', () => {
      const cases = [
        {
          input: {
            weakMapKey: { id: 100001 },
          },
        },
        {
          input: {
            weakMapKey: { id: 100002 },
          },
        },
      ]

      test.each(cases)('weakMapKey: $input.weakMapKey', ({ input }) => {
        const received = BoundCtorRegistry.ensureCtorPool(input)

        expect(received)
          .toBeInstanceOf(WeakMap)
      })
    })

    describe('should be memoized', () => {
      const cases = [
        {
          input: {
            weakMapKey: { id: 100003 },
          },
        },
        {
          input: {
            weakMapKey: { id: 100004 },
          },
        },
      ]

      test.each(cases)('weakMapKey: $input.weakMapKey', ({ input }) => {
        const expected = BoundCtorRegistry.ensureCtorPool(input) // 1st call = memoized reference

        const received = BoundCtorRegistry.ensureCtorPool(input) // 2nd call

        expect(received)
          .toBe(expected) // Same reference
      })
    })
  })
})
```

### For shared static state (a pool), use a fresh key per case

When the memoization container is a **static property** (e.g.
`static pool = new WeakMap()`), that state **persists across tests** (`afterEach`'s
`jest.restoreAllMocks()` only restores spies — it does not reset the contents of a
static `WeakMap`). To avoid cross-contamination between cases and between tests, use a
**fresh object reference per case** as the key (e.g. `{ id: 100001 }`). Since a
`WeakMap` distinguishes keys by object reference, a different reference means a
different entry, so there is no interference.

## Single loop

When there is **no combination** between constructor properties and method arguments,
use a single loop (a single tier of `test.each`). Gather the required arguments into
one `input`.

```js
describe('PathnameBuilder', () => {
  describe('#buildPathname()', () => {
    describe('should interpolate placeholder', () => {
      const cases = [
        {
          input: {
            valueHash: { id: 123 },
          },
          expected: '/users/123',
        },
      ]

      test.each(cases)('valueHash: $input.valueHash', ({ input, expected }) => {
        const builder = PathnameBuilder.create({ templatePathname: '/users/[id]' })

        const received = builder.buildPathname(input)

        expect(received)
          .toBe(expected)
      })
    })
  })
})
```

## Constructor Property × Method Argument (double loop)

When a member takes both a **constructor property** and a **method argument**, and
**both** are involved in the output (including behaviors like memoization), **default
to this double loop**. Combine `describe.each()` (outer = constructor property) with
`test.each()` (inner = method argument). A member such as
`#ensureBoundCtor({ bindings, deriver })` (property `BaseCtor` × argument `bindings`)
fits this.

- **Do not lay out a single loop pairing "one property value × one argument value" one
  pair at a time.** That fails to verify that each axis independently affects the
  behavior, and invites false positives (it would still pass with one side held
  fixed). For the same reason as
  [when multiple properties are involved in the output](#when-multiple-properties-are-involved-in-the-output-property--property),
  lay out **at least 2×2** (2 or more values per axis).
- The exception is when **one axis is "a neutral collaborator not under test."** If an
  argument must be passed but does not affect the behavior (i.e. only one side is
  involved in the output), it is fine to use a single loop and fix the other side at a
  neutral value. For the `constructorArgs` / `<methodName>Args` split in that case, see
  [If `input` mixes properties and arguments, split into `args`](#if-input-mixes-properties-and-arguments-split-into-args-constructorargs--methodnameargs).
  Fix a value as neutral only when you can say **the contract guarantees that axis
  does not affect the behavior** — not because "reading the implementation shows it
  has no effect, so I'll use a single loop" (the QA principle in
  [SKILL.md](../SKILL.md)).
- Each element of the outer `cases` **bundles together** `input` (the constructor
  property) with its own dedicated inner cases (`~Cases`).
- For the naming convention of the inner cases variable (the prefixed `~Cases`), see
  [naming.md](./naming.md).
- Use `override` / `input` / `tally` on the **outer** element side. `expected` may be
  placed on the **inner** side if its value **varies per inner argument**, or on the
  **outer** side if it is **constant per outer property** (the only reserved property
  allowed in the inner `~Cases` is `expected`).

```js
describe('SomeClass', () => {
  describe('#someMethod()', () => {
    const cases = [
      {
        input: {
          alpha: 1,
        },
        betaCases: [
          {
            beta: 10,
            expected: 9,
          },
          {
            beta: 20,
            expected: 19,
          },
        ],
      },
      {
        input: {
          alpha: 2,
        },
        betaCases: [
          {
            beta: 20,
            expected: 18,
          },
          {
            beta: 30,
            expected: 28,
          },
        ],
      },
    ]

    describe.each(cases)('alpha: $input.alpha', ({ input, betaCases }) => {
      test.each(betaCases)('beta: $beta', ({ beta, expected }) => {
        const instance = SomeClass.create(input)

        const received = instance.someMethod({ beta })

        expect(received)
          .toBe(expected)
      })
    })
  })
})
```

### Placement within a `describe.each()` callback (what may go before `test.each()`)

Inside a `describe.each()` callback, the following may be placed **before**
`test.each()`.

1. **Variables that can be built without using the inner `~Cases` values** (e.g. an
   instance built only from the outer `input`) go before `test.each()`. Only things
   that depend on the inner value (such as the `args` argument) go inside
   `test.each()`. This lets the same variable be shared across all inner tests,
   reducing duplication. This **applies equally to a single `test.each()`**: any
   variable that does not depend on the per-case value the callback receives (`input`
   / `expected`, etc.) should be defined once, outside `test.each()`. If `args` is
   **case-invariant** (buildable from fixed fixtures alone), pull it outside the
   callback rather than rebuilding it inside
   ([aaa-pattern.md](./aaa-pattern.md#assemble-method-arguments-in-arrange-and-pass-a-variable-in-act)).
2. When placing statements such as assignments before `test.each()`, insert **a blank
   line** between them and `test.each()` (to make the boundary between setup and
   iteration explicit).
3. If the inner `~Cases` passed to `test.each()` is **the same across every
   `describe.each()` case**, you can define it **once, right before `test.each()`**,
   rather than bundling it into each element of the outer `cases` (the outer can then
   focus purely on `input` / `expected`, eliminating duplication of the inner data).
   If the inner cases differ per case, bundle `~Cases` into the outer elements as
   before.

If `expected` is **constant per outer property** (unchanged even as the inner argument
varies), place `expected` on the outer element, and let the inner `~Cases` hold only
the arguments. Below is an example applying (1)(2)(3) above to the verification that
`#ensureBoundCtor()` "derives from `BaseCtor`". The derivation source is determined by
`BaseCtor` (outer), and does not change even as `bindings` (inner) varies. Hence
`expected` is placed on the outer; `bindingsCases`, being the same across every
`BaseCtor`, is defined right before `test.each()`; and `registry`, buildable from
`input` alone, is placed before `test.each()`.

```js
describe('BoundCtorRegistry', () => {
  describe('#ensureBoundCtor()', () => {
    /**
     * @param {{ Ctor: new () => * }} params
     * @returns {new () => *}
     */
    const deriver = ({ Ctor }) => class extends Ctor {}

    describe('should be derived from base constructor', () => {
      const cases = [
        {
          input: {
            BaseCtor: AlphaBaseCtor,
          },
          expected: AlphaBaseCtor, // Constant regardless of the inner bindings → placed on the outer
        },
        // ... an element for BetaBaseCtor follows
      ]

      describe.each(cases)('BaseCtor: $input.BaseCtor.name', ({ input, expected }) => {
        const registry = new BoundCtorRegistry(input) // (1) buildable from input alone → before test.each()

        const bindingsCases = [ // (3) the same across every BaseCtor → defined once, right before
          {
            bindings: [
              { id: 100001 },
            ],
          },
          {
            bindings: [
              { id: 100002 },
            ],
          },
        ] // (2) separated from test.each() by the blank line that follows

        test.each(bindingsCases)('bindings: $bindings', ({ bindings }) => {
          const args = {
            bindings, // Depends on the inner value, so assembled inside test.each()
            deriver,
          }

          const BoundCtor = registry.ensureBoundCtor(args)
          const received = BoundCtor.prototype

          expect(received)
            .toBeInstanceOf(expected)
        })
      })
    })
  })
})
```

## When multiple properties are involved in the output (property × property)

When a method's (or getter's) output involves **two or more properties**, test their
**combinations** with a double loop of `describe.each()` (outer = one property) ×
`test.each()` (inner = the other property). To confirm each property independently
affects the output, write **at least 2 × 2 = 4 cases** (2 or more values per
property).

- Do not use a single loop varying only one property. Fixing one side makes it
  impossible to confirm whether the other side contributes to the output (this
  invites false positives).
- The variable naming and inner cases (`~Cases`) conventions are the same as in
  [Constructor Property × Method Argument](#constructor-property--method-argument-double-loop).
- Keep primitive values **unique** across all 4 cases ([test-cases.md](./test-cases.md)).

```js
describe('BaseAuthorizationBuilder', () => {
  describe('#generateHeaderValue()', () => {
    describe('should join scheme and credential', () => {
      const cases = [
        {
          input: { scheme: 'Bearer' },
          credentialCases: [
            {
              credential: 'credential-0001',
              expected: 'Bearer credential-0001',
            },
            {
              credential: 'credential-0002',
              expected: 'Bearer credential-0002',
            },
          ],
        },
        {
          input: { scheme: 'Basic' },
          credentialCases: [
            {
              credential: 'credential-0003',
              expected: 'Basic credential-0003',
            },
            {
              credential: 'credential-0004',
              expected: 'Basic credential-0004',
            },
          ],
        },
      ]

      describe.each(cases)('scheme: $input.scheme', ({ input, credentialCases }) => {
        test.each(credentialCases)('credential: $credential', ({ credential, expected }) => {
          const builder = new BaseAuthorizationBuilder({
            scheme: input.scheme,
            credential,
          })

          const received = builder.generateHeaderValue()

          expect(received)
            .toBe(expected)
        })
      })
    })
  })
})
```

## Separate mutually-interacting arguments with `describe()`

When a method has two or more arguments and **one changes how the other is
interpreted or behaves** (a mode / flag / strategy — not independent, but
**interacting**), **split by the value of the mode-like argument, using
`describe()`**. The standard form is `describe('with <arg>: <value>')`. Fix the mode
argument within each describe and vary the remaining arguments.

- If the two axes affect the output independently, the
  [double loop](#constructor-property--method-argument-double-loop) grid suffices. But if
  **one conditions the other** (a flag changes the processing path itself), splitting
  by describe per path reads better.
- Why split: mixing modes into a single `cases` makes it hard to trace which mode each
  case verifies, and makes it hard to confirm coverage per mode (whether the same
  input range was tried under each mode). Splitting by describe makes the mode
  explicit by name, and coverage within each mode is visible at a glance.
- Inside the mode describe, the remaining axis may be driven using the existing
  conventions ([Constructor Property × Method Argument](#constructor-property--method-argument-double-loop)
  / [when multiple properties are involved in the output](#when-multiple-properties-are-involved-in-the-output-property--property)
  / an axis × count grid, etc.).
- Since the mode argument becomes a fixed value within each describe, **omit** it from
  the `cases` elements and write it directly into `args` in the `test` body (e.g.
  `mode: true`).

```js
describe('SomeClass', () => {
  describe('#run()', () => {
    describe('with mode: true', () => {
      const cases = [
        {
          input: {
            value: 'alpha',
          },
          expected: '...',
        },
        // ... the input range when mode is true
      ]

      test.each(cases)('value: $input.value', ({ input, expected }) => {
        const args = {
          value: input.value,
          mode: true, // mode is fixed by the describe → written directly into args
        }

        const received = SomeClass.create().run(args)

        expect(received)
          .toBe(expected)
      })
    })

    describe('with mode: false', () => {
      // Run the same input range with mode: false (the path changes, so expected changes too)
    })
  })
})
```

## Separate boolean arguments/properties with `describe()`

For a **boolean argument or property** that the target under test receives (a true/false
flag such as `isEnabled`), do not fold it into `cases`'s `input` — instead, **separate
by `describe()` per value** (`describe('when #isEnabled:false')` /
`describe('when #isEnabled:true')`). Since a boolean only takes two possible values,
splitting by describe is easy, and once split, each `cases`'s `input` / `expected` can
**focus purely on the other axis**, making it simpler.

- This is the classic case of [Separate mutually-interacting arguments with
  `describe()`](#separate-mutually-interacting-arguments-with-describe). Since the boolean value is fixed per
  describe, **omit** it from the `cases` elements and write it directly into `args` in the `test` body (e.g.
  `isEnabled: false`). Leave `input` holding **only the other axis's value**.
- When coexisting with another describe layer (such as the valid/invalid split), put
  the **boolean describe on the inside**
  (`describe('with valid values') > describe('when #isEnabled:false')`). Valid/invalid
  reflects the nature of the input value, while a boolean flag is a mode — so split by
  the nature of the value first, and vary the mode within it.
- The label is `when <flag>:<value>`. If the flag is an **instance property** (one
  that exists as `this.<flag>`, such as `isNullable`), prefix it with **`#`** per the
  member notation (`when #isNullable:true`). A pure method-argument flag that is not
  turned into a property (e.g. `mode`) uses the bare name (`when mode:true`). The
  `args` key and `this.<flag>` remain in their bare form — `#` appears only in the
  describe label's notation (the same as `#instanceMethod()`).
- Even when a boolean flag **changes the very determination of valid/invalid** (e.g.
  under `#isEnabled:true`, a value that was previously invalid becomes valid, or only
  the null guard remains), place inside each mode's `describe()` only the results that
  actually occur under that mode (do not include, say, borrowed values that throw,
  per the policy in [test-cases.md](./test-cases.md)).

```js
describe('SomeClass', () => {
  describe('#someMethod()', () => {
    describe('with valid values', () => {
      describe('when #isEnabled:false', () => {
        const cases = [
          {
            input: {
              value: 'alpha-0001',
            },
            expected: 'alpha-0001',
          },
          // ...
        ]

        test.each(cases)('value: $input.value', ({ input, expected }) => {
          const args = {
            value: input.value,
            isEnabled: false, // The boolean flag is fixed per describe → folded into args, not into cases
          }
          const instance = SomeClass.create(args)

          const received = instance.someMethod()

          expect(received)
            .toBe(expected)
        })
      })

      describe('when #isEnabled:true', () => {
        // Same shape with isEnabled: true. input holds only the value axis
      })
    })

    describe('with invalid values', () => {
      // describe('when #isEnabled:false') / describe('when #isEnabled:true') likewise
    })
  })
})
```

## For a member that may depend on a property, split `describe()` by that property axis to show (in)dependence too

When a member's output **may depend** on a class property such as `#isNullable`,
verify it by splitting on that property axis with `describe('when #isNullable:false')`
/ `describe('when #isNullable:true')`. Even when it **does not depend** on it (an
override ignores it and always returns the same result), **do not omit this** — only
by verifying that both modes produce the same result can you guarantee the contract
that this member is **independent** of the property (if you skip it based on
implementation knowledge — "it shouldn't have an effect" — you would fail to notice if
it accidentally regresses into depending on the property; the QA stance in
[SKILL.md](../SKILL.md)).

- This is an application of
  [Separate boolean arguments/properties with `describe()`](#separate-boolean-argumentsproperties-with-describe).
  The only difference is that a **dependent** member (e.g. `#normalizeValue()`
  changing its result based on `#isNullable`) produces **different** results per mode,
  while an **independent** member (e.g. `#hasAvailableNormalizedValue()` below)
  produces the **same** result per mode — but both split by the property axis with
  `describe()`.
- When coexisting with another describe layer (`should be truthy` / valid-invalid,
  etc.), the property describe goes **on the inside**
  (`describe('should be truthy') > describe('when #isNullable:false')`).
- Since the label refers to an instance property, prefix it with `#`
  (`when #isNullable:true`). Fix the property value per describe and write it into the
  `test` body's `args` (e.g. `isNullable: false` — no `#` prefix there).
- Even for an independent member, including a value the base class treats specially
  (e.g. `null`) on the value side of the cases lets you demonstrate independence from
  **both** the property and the value at once.

```js
// SomeClass#isAvailable() ignores #isNullable and always returns available.
// It is still split by #isNullable, and both modes are verified to return the same truthy, showing independence.
describe('SomeClass', () => {
  describe('#isAvailable()', () => {
    describe('should be truthy', () => {
      describe('when #isNullable:false', () => {
        const cases = [
          {
            input: {
              value: 100001,
            },
          },
          {
            input: {
              value: null, // The base class treats null as unavailable, but this override returns available
            },
          },
        ]

        test.each(cases)('value: $input.value', ({ input }) => {
          const args = {
            value: input.value,
            isNullable: false,
          }
          const received = SomeClass.create(args).isAvailable()

          expect(received)
            .toBeTruthy()
        })
      })

      describe('when #isNullable:true', () => {
        // Same value cases with isNullable: true. Still all truthy
      })
    })
  })
})
```
