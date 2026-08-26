# ESLint (jest plugin) mapping

This maps the jest rules **actually in effect** under this project's ESLint
configuration (`eslint-plugin-jest`'s `flat/recommended` plus overrides, bundled
by `@openreachtech/eslint-config`) to how this skill writes tests. **If you write
tests as this skill prescribes, `npm run lint` produces no jest-related errors.**
Items that overlap with conventions in other files reinforce one another.

> For the actual settings, see `eslint.config.js` (including the `tests/**/*.js`
> overrides) and `configurations/plugins/jest.js` in `@openreachtech/eslint-config`.

## Must follow (rules that error)

- **Use `test()` / `test.each()`, never `it()` / `it.each()`** (both at top level and inside `describe()`). For
  the reason, see the notation convention in [SKILL.md](../SKILL.md) (`it()` is
  easy to misread as `if()`). — `jest/consistent-test-it`
  (`fn: 'test'` / `withinDescribe: 'test'`)
- **Each `test()` has at least one `expect()` / `expect()` only inside `test()` /
  exactly one argument**. Wrapping assertions in a custom helper defeats detection
  and errors. — `jest/expect-expect` / `jest/no-standalone-expect` /
  `jest/valid-expect`
- **Do not write `test.skip` / `test.only` / `xtest` / `ftest` / `xdescribe` /
  `fdescribe`.** — `jest/no-disabled-tests` / `jest/no-focused-tests` /
  `jest/no-test-prefixes`
- **Do not leave commented-out tests.** — `jest/no-commented-out-tests`
- **Do not write conditionals (`if` / `switch` / ternary / `&&` / `??`) inside a
  `test()`, and do not place `expect()` under a conditional.** —
  `jest/no-conditional-in-test` / `jest/no-conditional-expect` (same as
  [anti-pattern.md](./anti-pattern.md))
- **Iterate with `.each`, not loops** (`test.each()` / `describe.each()` /
  `expect.each()`). — `jest/prefer-each`
- **Do not end a `describe()` / `test()` title with `.`.** — `jest/valid-title`
  (`mustNotMatch: '\\.$'`; consistent with shared/statements' no-trailing-period rule)
- **Use `async` / `await`, not a `done` callback.** — `jest/no-done-callback`
- **Do not `return` from a `test()` / do not `export` from a test file.** —
  `jest/no-test-return-statement` / `jest/no-export`
- **Hooks are only `beforeAll` / `beforeEach` / `afterAll` / `afterEach`, ordered
  before→after, no duplicates, placed at the top of `describe()`.** —
  `jest/prefer-hooks-in-order` / `jest/prefer-hooks-on-top` / `jest/no-duplicate-hooks`
- **Do not use jasmine globals (`spyOn` / `fail` / `jasmine.*`) or import
  `__mocks__`.** — `jest/no-jasmine-globals` / `jest/no-mocks-import`

## How to choose matchers (follow these; otherwise they error)

- **No alias matchers** (`toBeCalled` → `toHaveBeenCalled`, `toThrowError` →
  `toThrow`, etc.). — `jest/no-alias-methods`
- **Do not build comparisons/equality as expressions**: `expect(a === b).toBe(true)`
  → `expect(a).toBe(b)` (`jest/prefer-equality-matcher`); `expect(a > b).toBe(true)`
  → `expect(a).toBeGreaterThan(b)` (`jest/prefer-comparison-matcher`).
- **Use `toBe()` for primitive equality.** — `jest/prefer-to-be`
- **Use `toHaveLength()` for length** (`expect(arr.length).toBe(n)` is not allowed).
  — `jest/prefer-to-have-length` (consistent with shared/statements' "do not
  reference arrays individually via `[]` / `.length`")
- **Use `toContain()` for containment.** — `jest/prefer-to-contain`
- **Pass a message argument to `toThrow()`** (a bare `toThrow()` is not allowed). —
  `jest/require-to-throw-message`
- **Mock functions via `jest.spyOn()`, not a bare `jest.fn()`.** —
  `jest/prefer-spy-on` (details in [prohibit.md](./prohibit.md) / [mocks.md](./mocks.md))
- **Mock promises with `mockResolvedValue()` / `mockRejectedValue()`**
  (`mockImplementation(() => Promise.resolve())` is not allowed). —
  `jest/prefer-mock-promise-shorthand`
- **`jest.mock()` factories must be typed** (this skill prefers `jest.spyOn()`, so
  this rarely comes up). — `jest/no-untyped-mock-factory`

### Verifying calls (count and arguments)

- **Pin arguments whenever possible (`toHaveBeenCalledWith(...)`)**. Do not settle
  for just "was it called" (`toHaveBeenCalled()`). — `jest/prefer-called-with`
- **`toHaveBeenCalledTimes(n)` (count verification) may be used only when argument
  verification is present alongside it.** To verify the count, line up
  `toHaveBeenNthCalledWith(1, ...)` … `toHaveBeenNthCalledWith(n, ...)` n times,
  pinning the arguments of every call. Verifying only the count lets arguments pass
  through (QA stance; a convention of this skill, not lint-enforced).
- **Split count patterns into a `describe()` per count** (`when called once` /
  `when called twice` / `when called three times`, and if needed `when not called`
  (= 0 times)). Within each `describe()` you can write `expect()` out flat, without
  bringing in loops or conditionals (consistent with
  [anti-pattern.md](./anti-pattern.md) / [structure.md](./structure.md)).

```js
describe('#notify()', () => {
  describe('when called once', () => {
    const cases = [
      {
        input: { events: ['alpha'] },
        expected: { first: 'alpha' },
      },
    ]

    test.each(cases)('events: $input.events', ({ input, expected }) => {
      // arrange / act ...

      expect(handlerSpy)
        .toHaveBeenCalledTimes(1)
      expect(handlerSpy)
        .toHaveBeenNthCalledWith(1, expected.first)
    })
  })

  describe('when called twice', () => {
    const cases = [
      {
        input: { events: ['alpha', 'beta'] },
        expected: { first: 'alpha', second: 'beta' },
      },
    ]

    test.each(cases)('events: $input.events', ({ input, expected }) => {
      // arrange / act ...

      expect(handlerSpy)
        .toHaveBeenCalledTimes(2)
      expect(handlerSpy)
        .toHaveBeenNthCalledWith(1, expected.first)
      expect(handlerSpy)
        .toHaveBeenNthCalledWith(2, expected.second)
    })
  })

  describe('when not called', () => {
    const cases = [
      {
        input: { events: [] },
      },
    ]

    test.each(cases)('events: $input.events', ({ input }) => {
      // arrange / act ...

      expect(handlerSpy)
        .not
        .toHaveBeenCalled()
    })
  })
})
```

## Snapshots

- Inline snapshots up to 6 lines, external snapshots up to 12 lines. —
  `jest/no-large-snapshots`
- This skill pins concrete values with `toBe()` / `toEqual()`, so it does not use
  snapshots in principle (same spirit as "do not use loose matchers" in
  [prohibit.md](./prohibit.md)).

## Intentionally relaxed rules (this skill's patterns rely on these)

These are errors under `flat/recommended` but are turned **off** in this project.
This skill's patterns pass lint precisely because they are off.

- **`jest/no-identical-title` = off** → the **sticky-header approach of repeating
  the class-name `describe()` per member** ("index the 1st/2nd levels by definition
  name" in [SKILL.md](../SKILL.md)) passes lint. If it were on, identical titles
  would error.
- **`jest/prefer-lowercase-title` = off** (`ignoreTopLevelDescribe: true`) →
  capitalized titles such as `describe('ClassName')` are allowed.
- **`jest/prefer-strict-equal` = off** → `toEqual()` may be used (`toStrictEqual()`
  is not forced).
- **`require-top-level-describe` / `require-hook` / `no-hooks` (all hooks allowed) /
  `max-expects` / `prefer-expect-assertions` = off**.
- Under `tests/**/*.js`, `no-undefined` / `max-classes-per-file` / the no-static-class
  `no-restricted-syntax` selector are **off** (test code may write `undefined`
  literally, define multiple classes per file, and have classes without properties).
