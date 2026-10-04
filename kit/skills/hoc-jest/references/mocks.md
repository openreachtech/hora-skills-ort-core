# Mocks (How to Supply Mocks and Stubs)

Conventions for mocks and stubs in jest. Referenced from `SKILL.md`.
See [naming.md](./naming.md) for the meaning of the `override` case property.

**A test is driven with the real thing.** The collaborator a subject reaches is built from the
class that defines it, and what a case needs from that collaborator — a return value watched, a
return value swapped — is taken with `jest.spyOn()` on the instance itself. **Spying is part of
driving the real thing, not a departure from it**: the object is real, its code runs, and the spy
observes one member of it. What this rules out is the fabricated stand-in — an object literal cast
into the collaborator's type, or a `jest.fn()` standing where a member should be. The one
exception is a module the project does not own, and it is taken outside the test entirely (see
below).

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

## Never Read `mock.calls` in Assert; State It in the Matcher

**A spy is asserted through the matchers Jest gives it, never by reading the
call log it keeps.** `mock.calls`, `mock.lastCall` and `mock.results` do not
appear in an Assert phase.

| What is being checked | The matcher |
| :-- | :-- |
| Whether it was called | `toHaveBeenCalled()` / `.not.toHaveBeenCalled()` |
| How many times | `toHaveBeenCalledTimes(n)` |
| The arguments | `toHaveBeenCalledWith(expected)` |
| The arguments of the nth call | `toHaveBeenNthCalledWith(n, ...expected)` |
| The arguments of the last call | `toHaveBeenLastCalledWith(...expected)` |

Reading the log instead costs three things, every time.

- **It needs a subscript.** `spy.mock.calls[0][0]` is the subscript access the
  statement convention turns away, and it carries two meanings —
  `[which call][which argument]` — in position alone.
- **The expected value swells into the whole call log.** `[[expect.objectContaining({
  message })]]` expresses "called once, and its first argument was" through
  nothing but the depth of the brackets. Written as the matcher takes it, the
  same expectation is `expect.objectContaining({ message })` and two levels of
  brackets disappear.
- **The failure output degrades.** `toHaveBeenCalledWith` reports a diff against
  the arguments the spy received; `expect(spy.mock.calls).toStrictEqual(...)`
  reports a diff between nested arrays, and the reader has to work out which
  bracket was the call and which was the argument.

```js
// NG: the call log read in Assert
expect(reportErrorSpy.mock.calls[0][0])
  .toHaveProperty('message', expected)

// NG: the whole log compared
expect(reportErrorSpy.mock.calls)
  .toStrictEqual(expected) // expected: [[expect.objectContaining({ message })]]

// OK: stated in the matcher
expect(reportErrorSpy)
  .toHaveBeenCalledWith(expected) // expected: expect.objectContaining({ message })
expect(reportErrorSpy)
  .toHaveBeenCalledTimes(1)
```

- **Retrieving a value the spy was handed is not this.** Where a test has to
  take a callback that was passed to a dependency and call it, the log is the
  only handle Jest offers, and that retrieval belongs to Arrange or Act. A
  `mock.calls` appearing in Assert means a matcher was not known.

## Spy the instance the subject will use, never a class prototype

**A spy goes on the object the subject under test will actually reach** — the instance it
is handed, or the args object it is passed. Planting one on a class's `prototype` so that
every instance picks it up is not the same verification.

```js
// Avoid: every instance in the process now answers the spy
jest.spyOn(EnvironmentFacade.prototype, 'isProduction')
  .mockReturnValue(true)

const engine = new SomeServerEngine({ config, share, errorHash })
```

- **A prototype spy passes even when the subject reads something else.** It answers for
  every instance there is, so a member holding the wrong collaborator — one built
  elsewhere, one left over from a previous step — still returns the value the test planted,
  and the test reports success for a path it never exercised. Spying the instance the
  subject was given makes that the thing under observation.
- It also outlives the case in a way the instance does not: until `afterEach` restores it,
  the substitution is in force for anything else the test touches.
- The same reasoning turns away spying a **collaborator's** class where the subject's own
  class would do: the seam to take is the nearest one the subject reads.

### Where the collaborator cannot be spied, build it per case and pass it in

**Some collaborators refuse the spy.** An object behind a `Proxy` whose `set` trap throws,
a frozen object, a value produced fresh on every read — `jest.spyOn()` defines a property
on the target, so on any of these it fails at the moment it is called.

```
can not modify environment variable [isProduction]
```

That error is the proxy's, not Jest's: the spy was being planted on an environment object
whose trap rejects every assignment.

- **Build the collaborator the case needs and hand it to the subject.** Where the class
  that produces it can be constructed directly, construct it with the state the case wants
  (`new EnvironmentFacade({ environmentHash: { NODE_ENV: 'production' } })`) and pass it
  through whatever the subject takes it by. The case then carries the collaborator
  ([test-cases.md](./test-cases.md#a-case-carries-the-collaborator-already-built)).
- **This runs the collaborator's real code**, which the prototype spy replaced. A predicate
  such as `isProduction()` is exercised rather than stubbed, so the test covers the
  predicate as well as the branch that reads it.
- Reach for this only where the seam genuinely refuses a spy. Where a spy can be planted on
  the instance, plant it — the substitution stays smaller.

### A shared fixture may hold the member that is spied

A fixture defined once under the member describe
([structure.md](./structure.md#define-shared-fixtures-directly-under-the-member-describe))
**may carry the function each test spies on**, declared as a real function. Each test
plants its spy on that same object, and `afterEach(() => jest.restoreAllMocks())` puts the
function back before the next one runs.

```js
/** @type {SomeType.ValidationContext} */
const mockContext = /** @type {*} */ ({
  getFragment: () => null,
  reportError: () => {},
})

describe('should report each selection over the cap', () => {
  test.each(cases)('...', ({ input, expected }) => {
    const reportErrorSpy = jest.spyOn(mockContext, 'reportError')
    // ...
  })
})
```

- **Cloning the fixture per test to keep the spy out of it buys nothing**, and the copy —
  `{ ...mockContext, reportError: () => {} }` — reads as a second fixture whose difference
  from the first is the thing the reader has to work out.
- What makes this safe is the restore hook, not the shape of the fixture. Where a project
  has no such hook, the substitution does outlive the test, and that is a defect in the
  setup rather than a reason to clone.

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

## A third-party module is stood in for by a `MockXxxx` class

**A test never casts an object literal into a third-party type.** The stand-in is
a class of its own — `MockXxxx`, for the type `Xxxx` it stands in for — living
under `tests/mocks/`
([directory.md](./directory.md#a-stand-in-for-a-third-party-module-lives-under-testsmocks)).
Its `.create()` declares the real type as its return, so every test takes `Xxxx`
from `MockXxxx.create()` and writes no cast at all.

```js
// tests/mocks/MockValidationContext.js
export default class MockValidationContext {
  /**
   * Factory method.
   *
   * @returns {GraphqlType.ValidationContext} - Validation context.
   */
  static create () {
    return /** @type {*} */ ({
      getType: () => null,
      reportError: () => {},
    })
  }
}
```

```js
// the test holds the real type, and casts nothing
const mockContext = MockValidationContext.create()
const reportErrorSpy = jest.spyOn(mockContext, 'reportError')
```

- **The cast is spent once.** `/** @type {*} */` sits inside `.create()` and
  nowhere else, so that is the only place the checker is turned off. The JSDoc
  convention allows it there and refuses it at a test site.
- **A change to the third-party type has one place to land.** A literal cast
  written at each test site would have to be found and corrected everywhere, and
  nothing says where those places are.
- **The stand-in is tested like anything else the project writes**, at
  `tests/__tests__/tests/mocks/MockXxxx.js`: that `.create()` returns what it
  declares, and that every member a subject reaches answers as the stand-in
  promises.
- **An in-house collaborator never takes this shape.** Build the real class and
  hand the instance over, then take what the case needs from it with
  `jest.spyOn()`. The stand-in exists because a third-party type cannot be built
  here, which is a reason a class of ours never has.
