# Naming (Variable and Naming Conventions)

Naming conventions for variables, identifiers, and case properties in jest.
Referenced from `SKILL.md`.

## Notation of Class Members

The `describe()` name of a member follows the notation in the table below. Prefix
instance members with `#` and static members with `.`. Getters / setters use a
`get:` / `set:` prefix (e.g. `.get:schema` instead of `.schema`).

This notation is **governed by "Notation of Class Members" in the documentation
convention**. The table below restates it in the context of jest; when the
governing table changes, align this one with it.

| notation | members |
| :-- | :-- |
| `#instanceProperty` | instance property |
| `#instanceMethod()` | instance method |
| `#get:instanceGetter` | instance getter |
| `#set:instanceSetter` | instance setter |
| `.staticProperty` | static property |
| `.staticMethod()` | static method |
| `.get:staticGetter` | static getter |
| `.set:staticSetter` | static setter |

When referring to a member **within the prose** of `describe()` / `test()` at the
behavior layer, **follow the 2nd-level `describe()` format** as well (append `()`
for methods).

```js
// Good example: append () for methods
describe('should call JSON.stringify() with value and replacer', () => {})

// Bad example: no ()
describe('should call JSON.stringify with value and replacer', () => {})
```

## Notation When the Subject Is Not a Class

The table above covers class members. Two other subjects appear, and each has its
own level-2 notation
([structure.md](./structure.md#when-the-subject-is-not-a-class)).

| subject | level 1 | level 2 | notation |
| :-- | :-- | :-- | :-- |
| module of exported constants | module name | exported name | as the source writes it (`ERROR_CODE_HASH`) |
| data file, a part asserted | the file's path | the part's key path | as the file writes it (`message.form`) |
| data file, the whole asserted | the file's path | the reading | a bare lower-case noun phrase (`key set`) |

The last row carries no `#` or `.` by design: the absence is what tells a reading
of a file from a definition on a class. This notation governs the `describe()`
name only — the governing "Notation of Class Members" table covers class members
alone and is unchanged by it.

## Test Case Variables

- Name the array of test cases `cases`.
- For the inner cases of a double loop, use a prefixed `~Cases` name (e.g.
  `valueHashCases`, `betaCases`). The prefix represents the target being iterated
  over on the inner side (the method argument).

## Property Names of Case Objects

Use **only** the following four top-level properties on the element objects of
`cases`. Do not invent names like `params` / `args`. However, for the outer
`cases` of a double loop only, it is also acceptable to have a prefixed inner
`cases` property (`~Cases`) in addition to the ones below.

| property | purpose |
| --- | --- |
| `override` | Used to supplement an abstract method with a stub implementation when testing an abstract class. Not used for concrete classes. |
| `input` | The argument(s) passed to the subject under test (constructor / method). |
| `tally` | A value that **serves as both** `input` and `expected`. When the same value is both passed in and used as the expected value, combine it into a single `tally` instead of splitting it into `input` and `expected` (e.g. pass it to `create(tally)` and also verify it with `toHaveBeenCalledWith(tally)`). The name means "tally" in the sense of a matching token, expressing the intent that the passed value and the expected value are **checked to be identical**. This name was chosen as a short word that does not collide with common domain vocabulary. Use it **only when the passed value and the expected value are the same**; if the implementation transforms the argument such that the two differ, do not use `tally` — split it into `input` / `expected`. |
| `expected` | The expected return value / property value. |

### `tally` is only for when the "value passed to the subject" and "expected value" are the **same object reference / same value**

**Mechanical check (highest priority; if this is not satisfied, it is not `tally`)**:
`tally` must **appear in both** Arrange/Act (where the value is passed to the
subject) and Assert (where it is passed to a matcher)
(`member(tally)` … `expect(...).toXxx(tally)`).
If `tally` appears **in only one of the two**, that is a misuse.

- If `tally` appears only in Assert (the matcher) — i.e. Act passes a different
  value such as `input` — then it is **expected-value only**, so it should be
  `expected`. Example: calling `create(input)` and then asserting
  `expect(spy).toHaveBeenCalledWith(tally)` is wrong → use `expected`.
- If `tally` appears only in Act (Assert checks a different or derived value),
  then the value being passed should be `input`.
- What matters is **whether `tally` appears in both Act and Assert**, not the
  shape of the destructuring in `cases`. It is fine for `tally` to coexist with
  other properties, as in `({ input, tally })` or `({ tally, expected })` (passing
  a different argument via `input`, or checking a different value via
  `expected`, while `tally` still appears on both sides).
- If the member **transforms, infers, or renames the input to produce the
  expected value** (`rawSchema` → an inferred `BootScalar`, `null` →
  `SentinelScalar`, receiving a value under a different property name, etc.),
  do not use `tally` even if **the value happens to match in some cases**
  (`IntegerScalar` → `IntegerScalar`). Once a transformation is involved, the
  "value passed" and the "expected value" are different things. If even one
  case in the same `cases` array undergoes an effective transformation (e.g.
  `null` → `SentinelScalar`), treat the entire `cases` array as `input` /
  `expected`.

`tally` may only be used when the thing **passed to the subject under test** and
the thing **expected in the assertion** are **exactly the same value** (the same
scalar, or the same object reference). The criterion is not "a string that looks
the same" but "the very same value."

In particular, when the **subject under test receives an object and the expected
value is a scalar nested inside it** (or vice versa), the passed value (the
object) and the expected value (the scalar) are **different things**, so `tally`
cannot be used. Split into `input` / `expected`.

- Decision procedure: line up "the `X` passed to `subjectUnderTest(X)`" and "the
  `Y` in `expect(...).toXxx(Y)`". If `X` and `Y` are the same value, use `tally`;
  if they differ in any way (object vs. its inner scalar, only some properties,
  after transformation, etc.), use `input` / `expected`.
- The value of `tally` should be **what is passed as-is to the subject under
  test** (i.e. also expected as-is). If the structure wraps a single value when
  passing it, e.g. `{ prop: tally }`, then the passed side is an object while the
  expected side is a scalar, making them non-identical — do not use `tally`;
  split into `input` / `expected`. If both the passed and expected sides are
  objects, put **the whole object** into `tally` (do not put only the inner
  contents of a wrapper like `{ prop: value }` into `tally`).
- `tally` may have **multiple named members**
  (`tally: { alpha: { id: ... }, beta: { value: '...' } }`). If each member is
  used **as-is** (passed without modification, and asserted as-is), then
  referencing `tally.alpha` is "used wholesale," not "partial use."
- "Partial use" means passing a member after **wrapping it in a different
  structure / extracting only part of it** — i.e. **after processing** it. For
  example, wrapping it as in `new SomeClass({ replacer: tally.replacer })` or
  `method({ value: tally.value })` is not "as-is," so use `input` instead of
  `tally` in that case.

```js
// OK: each member is passed as-is and asserted as-is (used wholesale)
test.each(cases)('id: $tally.alpha.id', ({ tally }) => {
  const spy = jest.spyOn(target, 'method')

  target.run(tally.alpha, tally.beta) // pass as-is

  expect(spy)
    .toHaveBeenCalledWith(tally.alpha, tally.beta) // assert as-is
})

// NG: a member is wrapped again before being passed (not "as-is") → use input
test.each(cases)('value: $input.value', ({ input }) => {
  const stringifier = new SomeClass({ replacer: input.replacer })
  stringifier.method({ value: input.value })
  // ...
})
```

```js
// NG: tally cannot be used here. What is passed is { source } (an object),
//     and what is expected is btoa's argument = source (a scalar) — they differ.
const cases = [
  { tally: 'source-0001' },
]
test.each(cases)('...', ({ tally }) => {
  Builder.generateCredential({ source: tally }) // pass an object
  expect(btoaSpy).toHaveBeenCalledWith(tally)    // expect a scalar → not identical
})

// OK: split into input (the object passed) and expected (the scalar arg to btoa)
const cases = [
  {
    input: { source: 'source-0001' },
    expected: 'source-0001',
  },
]
test.each(cases)('source: $input.source', ({ input, expected }) => {
  Builder.generateCredential(input)
  expect(btoaSpy).toHaveBeenCalledWith(expected)
})

// Example of tally used correctly: the passed value and expected value are the same scalar
const cases = [
  { tally: 100001 },
]
test.each(cases)('id: $tally', ({ tally }) => {
  SomeClass.create(tally)                  // value passed
  expect(SpyClass.__spy__).toHaveBeenCalledWith(tally) // expect the same value
})

// NG: a single value is put in tally and then wrapped as { replacer: tally } when passed.
//     What is passed is { replacer } (an object), and what is expected is replacer (a scalar) — they differ.
const cases = [
  { tally: alphaReplacer },
]
test.each(cases)('...', ({ tally }) => {
  const instance = new SomeClass({ replacer: tally }) // pass an object
  expect(instance).toHaveProperty('replacer', tally)  // expect a scalar → not identical
})

// OK: split into input (the value passed) and expected (the property value)
const cases = [
  {
    input: { replacer: alphaReplacer },
    expected: alphaReplacer,
  },
]
test.each(cases)('replacer: $input.replacer', ({ input, expected }) => {
  const instance = new SomeClass({ replacer: input.replacer })
  expect(instance).toHaveProperty('replacer', expected)
})

// OK: if the passed object and the expected object are the same, put that "object" into tally
const cases = [
  { tally: { replacer: alphaReplacer } },
]
test.each(cases)('replacer: $tally.replacer', ({ tally }) => {
  SomeClass.create(tally)                              // pass the object
  expect(SpyClass.__spy__).toHaveBeenCalledWith(tally) // expect the same object
})
```

## Act's Return Value

- The variable that receives the return value in the Act step of the AAA pattern
  should be named `received`, not `actual`.
- `received` is **the value passed to `expect()`** (the value being asserted). If
  you assert the Act return value itself, that is what should be `received`. If
  you assert a **property** of the return value, receive the return value in a
  variable with a descriptive name (PascalCase for a class) first, then set
  `const received = <var>.<prop>`
  ([aaa-pattern.md](./aaa-pattern.md#pass-a-variable-to-expect-not-an-expression)).

## Naming the args Object

The variable in Arrange that assembles the argument object to pass to the
subject under test should, **in principle**, be named `args`.

**However, if a single `test` builds more than one args object** (e.g. one for
the constructor and one for a method — i.e. the destination splits across two or
more targets), `args` alone cannot distinguish them, so name each build as **the
destination member's name + `Args`**. Do not use role-based generic names like
`propertyArgs` / `methodArgs`.

- **Only one build** → `args`.
- **Multiple builds** → name each as the destination member's name + `Args`. The
  constructor uses `constructorArgs`; the method `foo()` uses `fooArgs` (e.g.
  `buildPathname()` → `buildPathnameArgs`).
- Splitting is needed when a single `input` is passed to multiple destinations —
  see
  [structure.md](./structure.md#if-input-mixes-properties-and-arguments-split-into-args-constructorargs--methodnameargs)
  for details.

```js
// Only one build → args
const args = {
  value: input.value,
}

const received = stringifier.stringifyBody(args)
```

```js
// Multiple builds (splitting one input into constructor and method parts)
// → destination member's name + Args
const constructorArgs = {
  templatePathname: input.templatePathname,
}
const buildPathnameArgs = {
  valueHash: input.valueHash,
}

const builder = new PathnameBuilder(constructorArgs)

const received = builder.buildPathname(buildPathnameArgs)
```

> Note: Multiple builds occur when a single `input` **mixes constructor
> properties and method arguments** and both need to be built. If `input` maps
> 1:1 to the constructor **and can be passed directly**, there is only one build
> (for the method), and `args` is used (the same applies to a double loop: if the
> outer input matches the constructor, e.g. `{ BaseCtor }`, pass it directly with
> `new SomeClass(input)`, and only build `args` for the inner argument).

## Variables That Hold a Spy

This convention applies only to variables whose value is a `jest.fn()` (i.e. a
spy/mock **function** itself). **The criterion is "does the held value equal
`jest.fn()`"**, and any variable meeting that criterion must **without
exception** get a `~Spy` suffix.

- Name variables holding a `jest.fn()` with a `~Spy` suffix (e.g. `btoaSpy`,
  `fetchSpy`). Do not use other terms like `tally`. This makes it explicit by
  name that the subject of call verification "is a function spy."
- If held in a constant (upper snake case), also uppercase the suffix, giving
  `XXXX_XXX_SPY` (e.g. `BTOA_SPY`, `FETCH_HANDLER_SPY`). This matches the naming
  convention's casing while still conveying the `~Spy` meaning via `_SPY`.
- `constructorSpy.spyOn()` returns a **class** (not a `jest.fn()`), so it falls
  **outside** this convention — it remains `SpyClass` (not because it is
  excluded/exempted, but because it never meets the criterion in the first
  place). The actual function spy can be retrieved as `SpyClass.__spy__`. If you
  bind this `__spy__` to a variable, it is equivalent to a `jest.fn()`, so give
  it a `~Spy` suffix. See the constructorSpy section of [mocks.md](./mocks.md)
  for details.

```js
const btoaSpy = jest.fn()

// ...

expect(btoaSpy)
  .toHaveBeenCalledWith(expected.source)
```
