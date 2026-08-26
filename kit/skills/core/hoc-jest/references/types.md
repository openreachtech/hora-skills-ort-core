# Types (type resolution)

Convention for type annotations and type resolution in Jest. Referenced from `SKILL.md`.

## Type Resolution

Attach a type annotation to `cases` **only when needed to resolve or suppress
a type error**. The purpose of `@type` is to make the type check pass; if the
correct type can be inferred from the literal and there is no type error
whether or not you attach it, it is **redundant, so it can be omitted**.

A cast via `@type {Array<*>}` should be limited to suppressing type errors
**to write irregular values** (`null` / `undefined` / missing keys, etc. that
violate the declared type). Irregular values are isolated into the
**abnormal-value-series `describe()`** by the valid/invalid separation
([Separate Valid / Invalid
Values](./structure.md#separate-valid--invalid-values)), so **only the
abnormal-value series needs the cast**.

- Normal-value series: if the value conforms to the type and the type can be
  correctly inferred from the literal, `@type` **may be omitted** (it's
  redundant since there's no type error). Attach `@type` only when inference
  alone wouldn't produce the intended type, or when you want to make it
  explicit for some other reason (and even then, do **not** add the
  `Array<*>` cast).
- Abnormal-value series: since you are deliberately passing type-violating
  values, cast the array literal with `@type {Array<*>}` in addition to
  `@type`, to suppress the intentional type error.

### Whether `@type` is needed should be confirmed with a type checker (where `tsc` is available)

To correctly judge "it can be omitted if there's no type error," you need to
**actually check whether a type error occurs, using a type checker**. **Where
`tsc` is available**, confirm with `npx tsc -p jsconfig.json --noEmit` (adjust
to the project's type-check configuration).

- **A clean `eslint` run is not proof of type resolution**. ESLint does not
  report `checkJs` type errors (`TS2769` overload mismatch / `TS2345` type
  incompatibility / `TS2349` not callable, etc.), so types may be broken even
  when `eslint` passes. Don't conclude "`@type` is unnecessary because eslint
  is clean."
- Real examples that are easy to miss: an overload mismatch in `test.each`
  lining up dynamic keys `Record<string, *>`, a not-callable error from
  calling a getter that returns a function as `predicate(value)`, a test
  stub that doesn't match a declared type (e.g. `ConstraintCtor`). eslint
  passes through all of these.
- In environments where `tsc` is unavailable, verify **by hand**, guided by
  the decision procedures in each section of this file (dynamic keys,
  generic base classes, function-returning getters, etc.).

```js
// Normal value (type can be inferred from the literal, so @type is omitted)
const cases = [
  // ...
]

// Abnormal value (writes irregular values, so a cast is used)
/**
 * @type {Array<{
 *   input: {
 *     templatePathname: string
 *   }
 *   valueHashCases: Array<{
 *     valueHash: Record<string, *>
 *     expected: string
 *   }>
 * }>}
 */
const cases = /** @type {Array<*>} */ ([
  // ... null / undefined / missing keys, etc.
])
```

## `TS2769` from passing a union type to an overloaded function is resolved with `Parameters<typeof Fn>[N]`

Passing a value of **a union type that spans** the parameter shapes of
multiple overloads to a function that has **multiple overloads**, like
`JSON.stringify`, matches none of the overloads and produces `TS2769` (No
overload matches this call). For example, if `replacer`'s type is
`((key, value) => *) | Array<string | number> | null`, it fits neither the
"function" overload nor the "array" overload.

The resolution is to **cast that argument to the type of the matching
overload's parameter**. Rather than spelling out a hand-written type,
referencing it via `Parameters<typeof Fn>[N]` (the type of `Fn`'s Nth
parameter) is accurate and keeps up with signature changes.

- The cast is placed **inline at the value position (right before the
  argument)**. Since this is not an assignment but an argument in an
  expression, it is exempt from [Write the type of an assigned value above
  the statement](#write-the-type-of-an-assigned-value-above-the-statement) — it should not become a
  statement-level declaration.
- This is **different** from "cast away a normal value" (forbidden in [For
  normal values, don't cast — prepare a real value matching the declared
  type](#for-normal-values-dont-cast--prepare-a-real-value-matching-the-declared-type)). The value stays a
  legitimately typed union value; the cast is to resolve overload-resolution
  constraints, which is a different purpose from an `Array<*>` value cast
  applied to an abnormal value that **violates** the declared type.
- The same thing shows up in test code (e.g. passing a union fixture for
  replacer/reviver to `JSON.stringify`). Align it with the same style used to
  resolve this on the source side.

```js
// TS2769: replacer's union type matches no overload
return JSON.stringify(value, this.replacer)

// Resolution: cast to the type of the matching argument (2nd argument = [1])
return JSON.stringify(
  value,
  /** @type {Parameters<typeof JSON.stringify>[1]} */ (this.replacer)
)
```

## Fix normal-value cases involving `Record<string, *>` (dynamic keys), etc., with a `@type` declaration

When the test target receives an argument with **a type that has no concrete
keys**, such as `Record<string, *>` (an index signature / dynamic keys), a
literal written into cases like `{ id: 100001 }` / `{ value: false }` gets
inferred as a **concrete key type** such as `{ id: number }` /
`{ value: boolean }` respectively. Because the shape of `valueHash` then
disagrees across cases, a **type error** occurs at the point where it's
passed to `test.each()` or to `method(args)`.

In this case, attach **only a declaration** of `@type {Array<{...}>}` to
`cases`, fixing the target field to the type on the declaration side (e.g.
`Record<string, *>`). Attaching the declaration aligns every case's value to
a single type, eliminating the inference drift and resolving the type error.

- Since this is a **normal value** (the value conforms to the declared type),
  do **not** add the `/** @type {Array<*>} */` **value cast**. The `Array<*>`
  value cast is the last resort for writing an irregular value that
  **violates** the declared type (abnormal-value series) ([Type
  Resolution](#type-resolution)). If you just want to fix the type for a
  normal-value series, the `@type` declaration alone is enough.
- This is an **exception** to "normal-value series can omit `@type` since it
  can be inferred from the literal" ([Type Resolution](#type-resolution)).
  When a dynamic-key type such as `Record<string, *>` is involved, plain
  inference does not produce a matching type, so in this case you attach the
  declaration rather than omitting it.
- Rule of thumb: if the source-side `@param` has **a non-fixed-key type**
  such as `Record<string, *>` / `{ [key: string]: * }` / `object`, and the
  cases line up **multiple literals with different keys**, attach a `@type`
  declaration.

```js
// Normal value, but with a Record<string, *> argument. Attach only a declaration (no value cast)
describe('with valid values', () => {
  /**
   * @type {Array<{
   *   input: {
   *     templatePathname: string
   *   }
   *   valueHashCases: Array<{
   *     valueHash: Record<string, *>
   *     expected: string
   *   }>
   * }>}
   */
  const cases = [
    {
      input: {
        templatePathname: '/users/[id]',
      },
      valueHashCases: [
        {
          valueHash: { id: 100001 }, // plain inference would make this { id: number }, disagreeing with Record
          expected: '/users/100001',
        },
        // ... postId/commentId, value: false, etc. — literals with different keys continue
      ],
    },
  ]
  // ...
})
```

```js
// Contrast: an abnormal value needs both the declaration and the Array<*> value cast (since it writes a type-violating value)
/**
 * @type {Array<{
 *   input: {
 *     templatePathname: string
 *   }
 *   valueHashCases: Array<{
 *     valueHash: Record<string, *>
 *     expected: string
 *   }>
 * }>}
 */
const cases = /** @type {Array<*>} */ ([
  // ... null / undefined / missing keys, etc.
])
```

## `@type` declarations on `cases` must type every field precisely (don't paper over with `*`)

Once you decide to attach a `@type` declaration to `cases`, **write out each
field inside it with its actual type**. Do not escape into `*` (any) for a
field just because typing it is bothersome or non-obvious, as in
`expected: *`. The purpose of the declaration is to **fix** the type and
eliminate inference drift/mismatch; a `*` field fixes nothing, and in fact
**buries a type mismatch that could otherwise be detected**. Once you commit
to attaching a declaration, mixing in `*` is self-contradictory.

- As noted in [Type Resolution](#type-resolution), `*` is the last resort
  used as the `Array<*>` value cast for the **abnormal-value series**
  (writing irregular values that violate the declared type). Don't habitually
  use `*` for fields of a **normal-value** `@type` declaration.
- Even if a field's type isn't obvious, write **a type derivable from an
  existing value**. Deriving it from the test target or its return value
  aligns the types of `received` and `expected`, backing the `.toBe()`
  comparison with types too. Example: when looking up a value in a
  namespace, use `(typeof ScalarHash)[keyof typeof ScalarHash]` (i.e. one of
  the exported values); for a class, `typeof SomeClass`; for an instance,
  `SomeClass`.
- "Write any as `*`" ([Write any as `*`](#write-any-as-)) is about places
  that are **genuinely fine as any** (implicit any arguments of a fixture,
  etc.). It is not an excuse to make a field that **has a concrete type**,
  like `expected`, into `*`.

```js
// Good: write expected out with its real type too (aligns the type with received)
/**
 * @type {Array<{
 *   input: {
 *     name: keyof typeof ScalarHash
 *   }
 *   expected: (typeof ScalarHash)[keyof typeof ScalarHash]
 * }>}
 */
const cases = [
  // ...
]
```

```js
// Avoid: attaching a declaration but then escaping expected into * (buries the type mismatch)
/**
 * @type {Array<{
 *   input: {
 *     name: keyof typeof ScalarHash
 *   }
 *   expected: *
 * }>}
 */
const cases = [
  // ...
]
```

## For normal values, don't cast — prepare a real value matching the declared type

When a type error occurs for a normal-value series, it is often a sign that
"the value doesn't match the declared type." **Before** papering it over with
`/** @type {*} */` or `Array<*>`, first consider **whether the value can be
swapped for a real value that matches the declared type**. A cast is a last
resort for writing an irregular value (abnormal-value series), not something
to use to hide a type mismatch in a normal value.

- If the argument's declared type is **a concrete object type** (`WeakMap` /
  `Map` / `Set` / a specific instance, etc.), pass **a real instance of that
  type** (`new WeakMap()`, etc.) rather than a substitute literal like
  `{ id: 100001 }`. This way, neither `@type` nor a cast is needed, and the
  type simply passes.
- Passing a substitute literal and suppressing it with a cast (1) breaks the
  meaning of the type, and (2) makes it impossible for the test to verify
  whether the implementation truly requires that type.
- Real instances are often opaque and can't carry a readable index. In that
  case, follow the `$#` numbering scheme in
  [test-cases.md](./test-cases.md#for-opaque-value-titles-too-first-use-a-readable-identifying-property--is-a-last-resort)
  for the case title.

```js
// Avoid: passing a plain object for a declared WeakMap type and suppressing it with a cast
const cases = [
  {
    input: {
      weakMapKey: /** @type {*} */ ({ id: 100001 }),
    },
  },
]

// Good: pass a real instance matching the declared type (no cast needed, type passes)
const cases = [
  {
    input: {
      weakMapKey: new WeakMap(),
    },
  },
]
```

## Write any as `*`

When representing "any" in a JSDoc type annotation, use `*` rather than the
literal `any` (e.g. `(key: string, value: *) => *`, `@param {{ value: * }}`).
The source-side JSDoc also uses `value: *`, so align with it.

## Resolve implicit-any fixtures with `@type`

When a fixture arrow function has an implicit-any argument and breaks the
type check (e.g. `key` / `value` in
`const alphaReplacer = (key, value) => value`), resolve it with a `@type`
annotation **rather than rewriting the value to avoid the type error**. Write
any as `*`.

```js
/** @type {(key: string, value: *) => *} */
const alphaReplacer = (key, value) => value
```

### For a function's implicit-any arguments, type with `@param` (`@returns` is required, multi-line block)

When the implicit any is **a function's argument**, it is more straightforward
to **type just the argument with `@param`** rather than writing the whole
function with `@type` (since `@param` points directly at where the implicit
any lives, i.e. the argument). If it's a property of an object literal, like
`const args = { deriver: ({ Ctor }) => ... }`, place it **directly above that
property**; for a standalone `const`, place it directly above that.

Note the ESLint (jsdoc plugin) constraints:
- Once you write `@param`, **`@returns` is also mandatory**
  (`jsdoc/require-returns`). It cannot be omitted if the function returns a value.
- A block containing `@param` **cannot be a single line**
  (`jsdoc/multiline-blocks`; single-line is only allowed for some tags such as
  `@type`). → It must always be written as a **multi-line block**.
- A constructor type only needs `new () => *` if it's used only as
  `extends` (no need to go as far as `new (...args: Array<*>) => *`).

```js
const args = {
  /**
   * @param {{ Ctor: new () => * }} params
   * @returns {new () => *}
   */
  deriver: ({ Ctor }) => class extends Ctor {},
}
const deriverSpy = jest.spyOn(args, 'deriver')
```

A single-line `@type` (`/** @type {(key, value) => *} */`) also passes the
type check, but that writes the type of the **whole** function. If all you
want is to resolve the implicit any on the argument, `@param` expresses the
intent more clearly.

## Write the type of an assigned value above the statement

When explicitly stating **the type after assignment (the declaration)** of a
value assigned to a variable, place `/** @type {...} */` **directly above the
statement**. Don't stack a fixed cast inline on the value side.

- Declare the type after assignment (i.e. the type that variable will hold
  from here on) with `@type` on **the line above**.
- When the right-hand-side value cannot be directly assigned to the declared
  type (e.g. putting `jest.fn()` into a function type), bridge the gap with
  **a single temporary `/** @type {*} */` cast on the value side**. Place
  the temporary cast right before the value, keeping the declared type
  separated onto the line above.
- This way, "what is this variable's type" can be read in one line directly
  above the declaration, and `*` is understood as playing the role of "a
  temporary escape hatch just to let the assignment through."

```js
// Good: the type after assignment goes on the line above; the * temporary cast goes on the value side
/** @type {typeof someFn} */
const someFnSpy = /** @type {*} */ (jest.fn())
```

```js
// Avoid: stacking a fixed cast on the value side (the declared type gets buried in the right-hand side)
const someFnSpy = /** @type {typeof someFn} */ (/** @type {*} */ (jest.fn()))
```

### Resolve dynamic-key types on the `cases` side, keeping the access site and Arrange clean

When looking up a namespace or object using a `cases` value (e.g.
`input.name`) as a **dynamic key** (`ScalarHash[input.name]`), a fixed-key
type cannot be indexed with `string` and produces a type error (`TS7053`). In
this case, the first move is to resolve it by **narrowing the type of the
`cases` that is the source of the key** — declare `input.name` as
`keyof typeof <lookup target>` in the `@type` of `cases` (an application of
[Fix normal-value cases involving dynamic keys with a `@type`
declaration](#fix-normal-value-cases-involving-recordstring--dynamic-keys-etc-with-a-type-declaration)).

- **Why on the `cases` side**: (1) The access site `ScalarHash[input.name]`
  can be read as a **plain expression with no cast**, keeping the test's main
  point (what is looked up and compared against what) clear. (2) The key
  string is **type-checked against the actual key set** of the lookup target,
  so a typo in an alias name or an export rename can be caught by the type
  system (a stronger contract).
- **Alternatives to avoid**:
  - A **type-resolution-only intermediate variable** (creating
    `const scalarHash = ScalarHash` just to do `scalarHash[key]`). This
    inserts an assignment into Arrange that's unrelated to the test, just to
    make one lookup work, diluting the AAA intent (noise for the reader).
  - A **value-position cast at the access site**
    (`/** @type {Record<string, *>} */ (ScalarHash)[key]`). The type passes,
    but it degrades to a `string` index, so **the key's type check no longer
    applies**, and noise remains in the access expression too.
- A value-position cast is appropriate only when **the key does not come
  from `cases`** (e.g. it's generated on the spot and cannot be constrained
  by the type of `cases`). If an element of `cases` is used as the key,
  always resolve it on the cases side.

```js
// Good: narrow input.name to keyof via the type of cases → the access site needs no cast
/**
 * @type {Array<{
 *   input: {
 *     name: keyof typeof ScalarHash
 *   }
 *   expected: (typeof ScalarHash)[keyof typeof ScalarHash]
 * }>}
 */
const cases = [
  {
    input: {
      name: 'Bool',
    },
    expected: BooleanScalar,
  },
  // ...
]

test.each(cases)('name: $input.name', ({ input, expected }) => {
  const received = ScalarHash[input.name] // no cast needed; the key is already type-checked

  expect(received)
    .toBe(expected) // same reference
})
```

```js
// Avoid: a value-position cast (the key's type check no longer applies, and noise remains in the expression)
const received = /** @type {Record<string, *>} */ (ScalarHash)[input.name]

// Avoid: a type-resolution-only intermediate variable (inserts an assignment unrelated to Arrange)
/** @type {Record<string, *>} */
const scalarHash = ScalarHash
const received = scalarHash[input.name]
```

## If a derived class defined for testing extends a generic class, resolve it with `@extends`

When a derived class defined inside a test (the kind used for a variable
element in `#get:Ctor`, or a class-name difference in `when not inherited`,
such as `class AlphaSub extends SomeBase {}`) `extends` **a generic class
that has `@template`**, attach a JSDoc `@extends {Base<...>}` directly above
that class declaration to **resolve the generics**. Without it, the type
arguments remain unspecified, and the editor/tsc warns that "type arguments
are missing."

- Ideally you'd write the concrete type if it's determined, but for a
  test-only vessel it's **fine to paper over it with `*`**
  (`@extends {SomeBase<*, *>}`). The **number** of type arguments should match
  the number of class-level `@template` parameters on the base class (if
  `SomeBase` has `@template A, B` — 2 parameters — use `<*, *>`).
- This may be written as a single-line block
  (`/** @extends {SomeBase<*, *>} */`).
- This applies only when **directly extending a generic base class**. It is
  unnecessary for a derived class that extends a class whose type arguments
  are already fixed (`class Foo extends SomeFixedBase {}`, where
  `SomeFixedBase` is already defined as `SomeBase<string, string>`).

```js
/** @extends {SomeBase<*, *>} */
class AlphaSub extends SomeBase {}

/** @extends {SomeBase<*, *>} */
class BetaSub extends SomeBase {}
```
