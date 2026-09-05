# AAA Pattern

Convention for Arrange / Act / Assert in Jest. Referenced from `SKILL.md`.
See [naming.md](./naming.md) for the naming of the variable that receives
Act's return value (`received`). Insert a **blank line** between Arrange /
Act / Assert to make the separation explicit.

```js
test.each(cases)('...', ({ input, expected }) => {
  const builder = SomeClass.create(input) // Arrange

  const received = builder.someMethod(input) // Act

  expect(received) // Assert
    .toBe(expected)
})
```

## Pass a variable to `expect()` (not an expression)

The **subject** passed to `expect()` should be **a variable**. Don't write a
property access or expression directly, as in `expect(received.name)`.

- If you're asserting on Act's return value itself, pass `received` as-is.
- If you're asserting on **a property** of Act's return value, first receive
  the return value with **a descriptively named variable** (for a class/
  constructor, PascalCase like `DeclaredCtor`), then create `received` as
  `const received = <var>.<prop>` and pass `received` to `expect(received)`.
  This separates "the execution result" and "the value being asserted" as
  variables.
- Since the return value and the extraction are tightly coupled, don't put a
  blank line between `const <Var> = ...` and
  `const received = <Var>.<prop>`; put the blank line after them to separate
  from Assert.

```js
// Good: receive Act's return value with a named variable, then create received before passing to expect
const DeclaredCtor = registry.declareBoundCtor(args) // Act
const received = DeclaredCtor.name

expect(received) // Assert
  .toBe(expected)
```

```js
// Avoid: passing an expression (property access) directly to expect()
const received = registry.declareBoundCtor(args)

expect(received.name)
  .toBe(expected)
```

## Assemble method arguments in Arrange, and pass a variable in Act

Do not **assemble the argument object of the method under test (Act) inline
in the call**. Building the argument is noise relative to the call logic, so
define it in Arrange as `const args = { ... }`, and pass that variable in
Act. Only when building args happens **multiple times** within a single test
(e.g. one for the constructor and one for a method) do you split it by the
target member name + `Args` instead of `args`
([naming.md](./naming.md#naming-the-args-object)).

- This applies when you're "assembling an object on the spot and passing it"
  (`method({ value: {} })`, `method({ value: input.value })`, shorthand
  `method({ valueHash })`, etc.).
- If you're just passing a value that's already a variable, as-is
  (`method(input)`), there's no assembly, so it's fine to inline it.
- Both building `args` and creating the test-target instance are Arrange, so
  do not separate them with a blank line ([Use blank lines only to separate
  the three phases](#use-blank-lines-only-to-separate-the-three-phases)).
- If **`args` is case-invariant** (it doesn't depend on `input` / `expected`
  received by the `test.each()` callback, and can be built from fixed
  fixtures alone), don't rebuild it every time inside the callback — define
  it **once, outside `test.each()`**. Only build arguments that depend on
  the case's value inside the callback (the same principle is detailed in
  [structure.md's placement rules](./structure.md#placement-within-a-describeeach-callback-what-may-go-before-testeach)).

```js
// Good: args is case-invariant (deriver is fixed) → defined once outside test.each()
describe('should be named after base constructor', () => {
  /**
   * @param {{ Ctor: new () => * }} params
   * @returns {new () => *}
   */
  const deriver = ({ Ctor }) => class extends Ctor {}

  const args = {
    deriver,
  }

  const cases = [ /* ... */ ]

  test.each(cases)('BaseCtor: $input.BaseCtor.name', ({ input, expected }) => {
    const registry = new BoundCtorRegistry(input) // depends on input, so it's inside the callback

    const DeclaredCtor = registry.declareBoundCtor(args)
    const received = DeclaredCtor.name

    expect(received)
      .toBe(expected)
  })
})
```

```js
// Good: assemble the argument in Arrange, and pass args in Act
test.each(cases)('value: $input.value', ({ input, expected }) => {
  const stringifier = new JsonRequestBodyStringifier({
    replacer: input.replacer,
  })
  const args = {
    value: input.value,
  }

  const received = stringifier.stringifyBody(args)

  expect(received)
    .toBe(expected)
})
```

```js
// Avoid: writing the argument object inline into the call (assembly logic leaks into Act)
test.each(cases)('value: $input.value', ({ input, expected }) => {
  const stringifier = new JsonRequestBodyStringifier({
    replacer: input.replacer,
  })

  const received = stringifier.stringifyBody({
    value: input.value,
  })

  expect(received)
    .toBe(expected)
})
```

## Only `expected` and `tally` may be passed to the matcher

Only **`expected`** and **`tally`** may be passed to Assert's matcher
(`toBe()` / `toHaveBeenCalledWith()`, etc.). **`input`** (or `override`),
which was used in Arrange to drive the call, **must not be passed to the
matcher**. This is to keep "the value passed in" and "the value expected"
separate even in the code.

- `expected`: the expected value. Use it when you want it kept separate from
  the value passed in (post-transformation, a different shape, a different
  thing).
- `tally`: a value used for a full comparison when the value passed in and
  the expected value are **identical**. Pass the **same** `tally` to
  **both** the call and the matcher ([naming.md](./naming.md)'s `tally`
  entry). `tally` may have multiple named members, and if each member is
  passed and asserted **as-is**, use the whole reference `tally.alpha` too
  (not partial use). If it's wrapped or only part of it is extracted before
  passing, use `input` instead.

- When the matcher takes **multiple arguments** (like
  `toHaveBeenCalledWith(a, b)`), make `expected` **an array of the
  arguments** and spread it with `...expected`. Spreading an array that
  represents the whole argument list corresponds 1:1 with the matcher's
  argument structure and reads better than destructuring into properties
  like `expected.a` / `expected.b`.
- When verifying multiple calls, make `expected` **an array of per-call
  argument arrays**, and spread each call as `...expected[0]` /
  `...expected[1]`, etc.

```js
const cases = [
  {
    input: {
      value: { id: 100001 },
      replacer: alphaReplacer,
    },
    expected: [
      { id: 100001 },
      alphaReplacer,
    ],
  },
]

test.each(cases)('value: $input.value', ({ input, expected }) => {
  const stringifySpy = jest.spyOn(JSON, 'stringify')

  const stringifier = new JsonRequestBodyStringifier({
    replacer: input.replacer,
  })
  const args = {
    value: input.value,
  }

  stringifier.stringifyBody(args)

  expect(stringifySpy)
    .toHaveBeenCalledWith(...expected) // spread expected, not input
})
```

## Write expected-value literals inline if single-line, bind to `const expected` if multi-line

When writing an expected value **as a literal in the test body** (a single
`test()` that doesn't use `cases` — a fixed value for `when called as is`,
inheritance, a fixed message for `when not inherited`, etc.), decide whether
to write it directly in the matcher or bind it to `const expected` based on
**whether the value is single-line or multi-line**. In `test.each()`,
`expected` is always a variable destructured from `cases`, so this
distinction only matters for a single `test()` that writes a literal in the body.

| Expected-value literal | How to write it |
| :-- | :-- |
| **Single-line** (`'...'` / number / boolean / `{ id: 100001 }` / `[Alpha, Beta]`) | Write **directly** in the matcher (`toBe(...)` / `toEqual(...)`) |
| **Multi-line** (a multi-key object, nested, an array of objects) | **Bind** to `const expected`, keep the matcher line as `.toEqual(expected)` |

- The deciding axis is not "is it a primitive or not" but "**can the matcher
  line fit in a short single token, or does it become a multi-line block**."
  Primitives are always single-line, so they fall into the direct-write case
  (as one instance of this).
- "Single-line vs. multi-line" is not an arbitrary threshold — it is
  **already determined** by the formatting convention in
  [test-cases.md](./test-cases.md#at-most-one-property-key-per-line) (single-key
  leaves may be inline, multi-key must be expanded, structural containers
  must be broken onto new lines). An object with multiple keys or more is
  always multi-line, so it falls into the bound-variable case.
- **Why multi-line is bound**: writing a multi-line literal directly into
  `.toEqual({ ... })` mixes the matcher's `)` with the object's `}`, erasing the
  outline of the Assert phase ([Use blank lines only to separate the three
  phases](#use-blank-lines-only-to-separate-the-three-phases)). Escaping it into
  `const expected` keeps the matcher line contained as `.toEqual(expected)`, which also
  aligns with the spirit of [Pass a variable to
  `expect()`](#pass-a-variable-to-expect-not-an-expression). The bound `const expected`
  belongs to the Arrange phase, so separate it from Act (`received`) with a blank line.
- **A literal `const expected` binding is different from binding a runtime-generated value**. A
  multi-line object literal (`const expected = { ... }`) is instantly recognizable as static, and is
  distinguishable at the binding point from a memoized `const expected = someCall()`
  ([structure.md](./structure.md#methods-that-memoize-and-return-the-same-reference-should-be-memoized)),
  so the meaning of `const expected` doesn't get confused.
- **When the expected value can't be compared as a literal** (`WeakMap` /
  `Set` / a memoized instance, etc. — something whose reference is unique and
  opaque), this falls back, before this branch, to `toBeInstanceOf(<type>)` or
  same-reference `toBe(expected)`
  ([structure.md](./structure.md#if-the-fixed-value-is-an-object-pin-it-by-type-tobeinstanceof)).

```js
// Single-line → write directly in the matcher
test('should be fixed value', () => {
  const received = CreateBenchmarkProjectMutationResolver.schema

  expect(received)
    .toBe('createBenchmarkProject')
})
```

```js
// Multi-line → bind to const expected, keep the matcher line as .toEqual(expected)
test('should be fixed value', () => {
  const expected = {
    type: 'object',
    required: [
      'id',
    ],
  }

  const received = SomeResolver.schema

  expect(received)
    .toEqual(expected)
})
```

## Add `// same reference` to a `toBe()` that checks the same reference

When passing an object (a function, instance, array, etc. — a non-primitive
value) to `toBe()` to verify **the same reference**, add a
`// same reference` comment at the end of the assertion line. Since `toBe()`
uses `Object.is` comparison, this makes explicit that the intent is checking
"is it the same reference," not primitive value equality.

- This applies only when passing an object reference. Don't add the comment
  for primitive value equality (strings, numbers, etc.), such as
  `toBe('Basic')`.

```js
test('should be fixed value', () => {
  const received = BasicAuthorizationBuilder.btoa

  expect(received)
    .toBe(btoa) // same reference
})
```

## Use blank lines only to separate the three phases

**Blank lines should only be used to separate the three phases** of
Arrange / Act / Assert. Even if a single phase has multiple steps, **do not
separate its interior** with blank lines. This makes it possible to tell
unambiguously "where the phase changes" from the presence of a blank line.

- Keep multiple lines within one phase close together, and leave a blank
  line only at the phase boundary.
- For example, if you split Arrange into two steps — "build the argument
  object" and "create the instance" — **don't put a blank line between
  them**, since both belong to the same Arrange.

```js
test.each(cases)('...', ({ input, expected }) => {
  const args = { // Arrange (logic (1): building the argument object)
    scheme: input.scheme,
    credential,
  }
  const builder = new BaseAuthorizationBuilder(args) // Arrange (logic (2): creating the instance)

  const received = builder.generateHeaderValue() // Act

  expect(received) // Assert
    .toBe(expected)
})
```

A wrong example (splitting one tightly-coupled piece of logic with a blank
line within the same Arrange):

```js
test.each(cases)('...', ({ input, expected }) => {
  const args = {
    scheme: input.scheme,
    credential,
  }

  const builder = new BaseAuthorizationBuilder(args) // ← don't add a blank line here, since this preparation is tightly coupled to creating the instance

  const received = builder.generateHeaderValue()

  expect(received)
    .toBe(expected)
})
```

### However, groups of statements with different meanings should be separated by blank lines

Even within the same Arrange, if **groups of statements with different
purposes (meanings)** appear side by side, it is fine to separate the groups
with a blank line. This is not as strong as the phase boundary (A/A/A), but
visually separating "distinct concerns within the preparation" improves
readability.

- Steps that are **tightly coupled toward a single result**, like "build the
  argument → create with that argument," should not be separated (as above).
- Groups with **separate concerns**, like "setting up mocks" and "creating a
  spy," should be separated by a blank line.
- **Creating the subject** (the test-target instance) and **the group of
  argument preparation** (defining `args` + its `jest.spyOn(args, key)`,
  etc.) are also separate concerns. **Create the subject first**, then place
  the argument-preparation group after a blank line. Since `args` and the
  `spyOn` applied to it are tightly coupled, don't separate the interior of
  that group ([mocks.md](./mocks.md#verifying-calls-to-a-function-passed-as-an-argument-callback--handler--deriver-with-jestspyonargs-key)).

```js
test.each(cases)('source: $input.source', ({ override, input, expected }) => {
  jest.spyOn(BaseAuthorizationBuilder, 'schema', 'get') // Arrange (concern 1: stub the return value)
    .mockReturnValue(override.schema)
  jest.spyOn(BaseAuthorizationBuilder, 'generateCredential')
    .mockReturnValue(override.credential)

  const SpyClass = constructorSpy.spyOn(BaseAuthorizationBuilder) // Arrange (concern 2: create the spy)

  SpyClass.create(input) // Act

  expect(SpyClass.__spy__) // Assert
    .toHaveBeenCalledWith(expected)
})
```

```js
test.each(cases)('BaseCtor: $input.BaseCtor.name', ({ input, expected }) => {
  const registry = new BoundCtorRegistry(input) // Arrange (concern 1: subject)

  const args = { // Arrange (concern 2: argument preparation; the args definition and its spyOn are tightly coupled, so don't separate the interior)
    /**
     * @param {{ Ctor: new () => * }} params
     * @returns {new () => *}
     */
    deriver: ({ Ctor }) => class extends Ctor {},
  }
  const deriverSpy = jest.spyOn(args, 'deriver')

  registry.declareBoundCtor(args) // Act

  expect(deriverSpy) // Assert
    .toHaveBeenCalledWith(expected)
})
```
