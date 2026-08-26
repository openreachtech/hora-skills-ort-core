# Mocks (How to Supply Mocks and Stubs)

Conventions for mocks and stubs in jest. Referenced from `SKILL.md`.
See [naming.md](./naming.md) for the meaning of the `override` case property.

## Override with jest.spyOn() Instead of Defining a Derived Class

When you need a stub implementation of an abstract method, or need to swap out
a method's return value, **avoid defining a new derived class whenever
possible** and instead override (swap the return value of) a member on the
subject class itself using `jest.spyOn()`.

- For a getter: `jest.spyOn(TargetClass, '<name>', 'get').mockReturnValue(...)`.
- For a method: `jest.spyOn(TargetClass, '<method>').mockReturnValue(...)`
  (use `.mockImplementation(...)` if the return value needs to vary based on the
  call arguments).
- Keep the value being substituted in the `cases`' `override` property (per the
  purpose of `override` described in [naming.md](./naming.md): supplementing an
  abstract method with a stub implementation).
- Tests should run and verify **the subject class itself**, not a derived class
  (e.g. `toBeInstanceOf(TargetClass)`).
- Mocks are restored automatically after every test via
  `afterEach(() => jest.restoreAllMocks())` in `setup-after-env.js`, so manual
  restoration is unnecessary.

**Why**: Creating a test-only subclass introduces problems: (1) more boilerplate
(e.g. redefining members with JSDoc), (2) you end up verifying the subclass
rather than the base class you actually want to test, and (3) if the stub
implementation and the real implementation drift apart, the test cannot notice.
With `jest.spyOn()`, you can make the smallest possible substitution against the
base class itself, keeping the intent clear.

Correct example (overriding an abstract static getter / static method with
spyOn):

```js
describe('BaseAuthorizationBuilder', () => {
  describe('.create()', () => {
    describe('should be an instance of own class', () => {
      const cases = [
        {
          override: {
            schema: 'Bearer',
            credential: 'credential-0001',
          },
          input: {
            source: 'source-0001',
          },
        },
        // ...
      ]

      test.each(cases)('source: $input.source', ({ override, input }) => {
        jest.spyOn(BaseAuthorizationBuilder, 'schema', 'get')
          .mockReturnValue(override.schema)
        jest.spyOn(BaseAuthorizationBuilder, 'generateCredential')
          .mockReturnValue(override.credential)

        const builder = BaseAuthorizationBuilder.create(input)

        expect(builder)
          .toBeInstanceOf(BaseAuthorizationBuilder)
      })
    })
  })
})
```

Incorrect example (defining a test-only derived class to implement the abstract
method):

```js
test.each(cases)('source: $input.source', ({ override, input }) => {
  class DerivedAuthorizationBuilder extends BaseAuthorizationBuilder {
    static get schema () {
      return override.schema
    }

    static generateCredential ({ source }) {
      return override.generateCredential({ source })
    }
  }

  const builder = DerivedAuthorizationBuilder.create(input)

  expect(builder)
    .toBeInstanceOf(DerivedAuthorizationBuilder)
})
```

## Combining with constructorSpy

When verifying delegation to a constructor, combine an `jest.spyOn()` override
with `constructorSpy.spyOn()`. Because a member replaced with spyOn is
referenced through the prototype chain, the class returned by
`constructorSpy.spyOn()` also picks up the same return value.

```js
test.each(cases)('source: $input.source', ({ override, input, expected }) => {
  jest.spyOn(BaseAuthorizationBuilder, 'schema', 'get')
    .mockReturnValue(override.schema)
  jest.spyOn(BaseAuthorizationBuilder, 'generateCredential')
    .mockReturnValue(override.credential)

  const SpyClass = constructorSpy.spyOn(BaseAuthorizationBuilder)

  SpyClass.create(input)

  expect(SpyClass.__spy__)
    .toHaveBeenCalledWith(expected)
})
```

## Verifying Calls to a Function Passed as an Argument (callback / handler / deriver) with `jest.spyOn(args, key)`

When verifying that a **function passed as an argument** to the subject under
test (a `deriver` / callback / handler, etc.) was called correctly, do **not**
create a `jest.fn()` and inject it into args. Instead, **define args normally
with a real function, then spy on that function property with
`jest.spyOn(args, '<key>')`**.

- By default, `jest.spyOn()` **calls through to the real implementation**, so the
  real function (the actual transformation) runs as-is. There is no need to
  write out a stub implementation like `jest.fn(() => ...)`.
- This makes the relationship clear: args holds "the real value," and the spy
  merely observes the call.
- This is consistent with this file's policy of preferring `jest.spyOn()`.
- Name the variable holding the spy with a `~Spy` suffix
  ([naming.md](./naming.md#variables-that-hold-a-spy)).
- Placement: create the subject (the instance under test) first, then, after a
  blank line, group "the args definition + its `spyOn`" together
  ([aaa-pattern.md](./aaa-pattern.md#however-groups-of-statements-with-different-meanings-should-be-separated-by-blank-lines)).

```js
test.each(cases)('BaseCtor: $input.BaseCtor.name', ({ input, expected }) => {
  const registry = new BoundCtorRegistry(input)

  const args = {
    /**
     * @param {{ Ctor: new () => * }} params
     * @returns {new () => *}
     */
    deriver: ({ Ctor }) => class extends Ctor {},
  }
  const deriverSpy = jest.spyOn(args, 'deriver')

  registry.declareBoundCtor(args)

  expect(deriverSpy)
    .toHaveBeenCalledWith(expected)
})
```

Incorrect example (creating a `jest.fn()` and injecting it into args):

```js
const deriverSpy = jest.fn(({ Ctor }) => class extends Ctor {}) // writing out a stub implementation
const args = {
  deriver: deriverSpy,
}
```

## Exception (When It Is Acceptable to Define a Derived Class)

A derived class may be defined only in cases that cannot be expressed with
`jest.spyOn()` (e.g. when the thing being substituted extends beyond a member to
the structure of the `class` itself). Even then, first consider whether it can
be written with spyOn before resorting to this.

A derived class may also be defined when **a single `describe()`'s `cases`
requires multiple derived classes at the same time**. Since `jest.spyOn()` only
swaps out members of a single class, situations that require several distinct
concrete classes to exist **simultaneously** on a per-case basis (e.g. verifying
the type differences among multiple subclasses themselves, or distinguishing
by registering subclasses in a registry) cannot be expressed with spyOn. In
this case, hold the derived classes in the elements of `cases` and iterate over
them.

### For a Getter That Returns a Function, Spy on "the Real Function It Returns"

If a getter simply returns an existing function (e.g. a global function or a
function from another module), trying to spy on the getter **itself** will fail
to type-check. Jest's type definitions classify "a getter that returns a
function" as a method-like key, excluding it from the `'get'` accessor overload
(which only allows property-typed keys). As a result, the third argument
`'get'` collapses to `never`, producing `TS2345` / `TS2339`
(`mockReturnValue does not exist on never`) — it works at runtime but fails to
type-check. (An `/** @type {any} */` cast may not disappear depending on the
environment's TS settings, and even if it does, it stops catching property-name
typos.)

**Leave the getter alone, and instead call `jest.spyOn()` on the real function
that the getter returns.** Since the real function is an ordinary data property
(a function value), it type-checks fine as a method spy — no cast or derived
class is needed. Because the getter returns the real function whenever it is
called, the spied function is used automatically (since `jest.spyOn()` calls
through to the real implementation by default, if you don't need side effects
you can just verify the call with `toHaveBeenCalledWith`).

```js
// static get btoa () { return btoa } returns globalThis.btoa, so
// spy on the real function globalThis.btoa rather than the getter.
test.each(cases)('source: $input.source', ({ input, expected }) => {
  const btoaSpy = jest.spyOn(globalThis, 'btoa')

  const args = {
    source: input.source,
  }

  BasicAuthorizationBuilder.generateCredential(args)

  expect(btoaSpy)
    .toHaveBeenCalledWith(expected.source)
})
```

**Only** when the real function has no independently spyable location (e.g. the
getter generates a new function every time, or the return value is a closure
that cannot be referenced from outside), define a derived class inside
`test.each()`, override the getter, and plant a spy (`jest.fn()`). Because
spyOn is not used, the `never` problem never arises structurally, and type
safety is preserved. Perform the call on the derived class side
(`OverriddenClass.method(...)`), taking advantage of the fact that the
overridden getter is referenced via `this`.

```js
// Fallback: only when the real function cannot be spied on independently
test.each(cases)('source: $input.source', ({ input, expected }) => {
  const someSpy = jest.fn()

  class OverriddenClass extends SomeClass {
    /** @override */
    static get factory () {
      return someSpy
    }
  }

  OverriddenClass.run(input)

  expect(someSpy)
    .toHaveBeenCalledWith(expected)
})
```
