# Prohibit (Prohibited Practices)

Collects the notation and matchers that **must not** be used in jest tests.
Referenced from `SKILL.md`. Each individual prohibition is a consequence of the
QA-stance core principle in [SKILL.md](../SKILL.md): "write tests so that they
would catch it if the implementation were wrong."

> For prohibited syntax related to "don't write logic in tests" — control flow,
> unverified logic, helper definitions, etc. — see
> [anti-pattern.md](./anti-pattern.md). This file covers other prohibitions
> (such as overly loose assertions).

## Do Not Use Loose Matchers (`expect.anything()` / `expect.any(Object)`)

**`expect.anything()`** and **`expect.any(Object)`** are **prohibited**. Both are
matchers that **accept a wide range of values without verifying anything
specific**, letting a test pass even when the implementation is wrong (a
violation of the QA stance).

- **`expect.anything()`** matches **anything except** `null` / `undefined`. Even
  if the implementation returns the wrong object, string, or number, the test
  will **pass** as long as it's non-null. It cannot detect hardcoding or
  regressions — the assertion effectively "does nothing."
- **`expect.any(Object)`** only checks "is it an instance of Object." In JS,
  almost everything is an Object (`{}` / arrays / functions / Date / RegExp,
  ...), so it **barely constrains anything**. Even if the shape of the return
  value regresses into something completely different, the test passes as long
  as the type matches.

Do not loosen the assertion because "writing the expected value as a literal is
tedious" or "the value is runtime-generated and can't be written out." Always
pin it to a **concrete value** or **concrete type**.

- **If there is a concrete value** → pass a literal `expected` to `toBe()` /
  `toEqual()` to pin the value (the same root as the uniqueness convention in
  [test-cases.md](./test-cases.md)).
- **If you want to pin the type** (e.g. the fixed value is an object with a
  unique reference) → pin it with a **concrete type**, e.g.
  `toBeInstanceOf(WeakMap)`, instead of `expect.any(Object)`
  ([structure.md](./structure.md#if-the-fixed-value-is-an-object-pin-it-by-type-tobeinstanceof)).
- **For a runtime-generated value** (e.g. the same reference returned by
  memoization) → bind the first return value to `expected` and verify identity
  with `toBe(expected)`
  ([structure.md](./structure.md#methods-that-memoize-and-return-the-same-reference-should-be-memoized)).

```js
// ❌️ Letting the value pass through unchecked (passes even if the implementation is wrong)
expect(received)
  .toEqual(expect.anything())

expect(received)
  .toEqual(expect.any(Object))

// ✅️ Pin a concrete value
expect(received)
  .toBe('Basic')

// ✅️ To pin the type, use a concrete type
expect(received)
  .toBeInstanceOf(WeakMap)
```

## Do Not Write `jest.fn()` Directly Inline Inside `test()`

Creating and using a `jest.fn()` inside `test()` (injecting it into args,
plugging it into a member) is **prohibited in principle**. Mocks and stubs
should spy on **a real seam** (a member of the real class / a function property
of args / a global function, etc.) using `jest.spyOn()`. There are three
reasons for this.

- **It amounts to writing out a stub implementation**: `jest.fn()` is an empty
  fake function; to give it meaning you end up having to
  **hand-write a stub implementation** like `jest.fn(() => ...)`. This is a copy
  of the real thing, and if it drifts, nobody notices — it introduces
  **unverified logic** into the test ([anti-pattern.md](./anti-pattern.md) /
  a violation of the QA stance). `jest.spyOn()` **calls through to the real
  implementation** by default, so no stub implementation is needed.
- **It is not restored automatically**: `jest.spyOn()` is restored after every
  test via `afterEach(() => jest.restoreAllMocks())`, but a member into which a
  `jest.fn()` was plugged inline is not restored, becoming a source of
  **cross-test contamination**.
- **It isn't tied to a real seam**: `jest.fn()` is a fake not tied to
  anything real, so it hides whether the actual collaborator works.
  `jest.spyOn()` targets a real, existing seam, observing calls while letting
  the real thing run.

See [mocks.md](./mocks.md) for the concrete techniques
(`jest.spyOn(args, key)` / `constructorSpy` / spying on global functions) and
for the **one exception** where `jest.fn()` is allowed (only when there is no
seam through which the real function can be spied on independently, override a
getter in a derived class and plant a `jest.fn()` there).

```js
// ❌️ Creating a jest.fn() and injecting it into args (writing out a stub implementation)
const deriverSpy = jest.fn(({ Ctor }) => class extends Ctor {})
const args = {
  deriver: deriverSpy,
}

// ✅️ Define args with a real function, and spyOn its function property
const args = {
  deriver: ({ Ctor }) => class extends Ctor {},
}
const deriverSpy = jest.spyOn(args, 'deriver')
```
