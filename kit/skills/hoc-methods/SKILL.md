---
name: hoc-methods
description: "Conventions for class method definitions and their signatures. Use when defining, calling or reviewing a method. Method names belong to the naming convention; what the body is written with, to the statements convention; how a factory method builds a dependency, to the dependency-wiring convention; how a recursion is laid out, to the method recursion convention."
---

# Classes: Members / Methods

This summarizes conventions related to class method definitions.

## Arguments should be a single named-argument object

- With some exceptions, the arguments of a defined method should in principle be received as a single object, using
  named arguments (destructuring assignment).
- Break each argument onto its own line and chop it down (one property per line).

```javascript
// NG: positional arguments
generate (length) {
  // ...
}

// OK: single named-argument object, chopped down
generate ({
  length,
}) {
  // ...
}
```

### Exception: binding (inflator) methods pass arguments flat

- An inflator method — a static method that binds a value to the class and returns a derived
  subclass, such as `.use()` / `.of()` / `.to()` — is the exception to the rule above. What it
  receives instead, the exceptions to that, and its naming belong to `/hoc-classes-inflators`.

## Do not pass properties directly to private methods

- Unless there is a specific reason, do not pass an (instance) property directly as an argument to a private /
  internal method.
- Internal methods should reference the properties they need directly from `this`. This avoids passing arguments around and preserves encapsulation.
- If a piece of logic needs a property passed as an argument, that is a sign that "it should be a static method rather than an instance method."
- However, making it static would remove the point of instantiation, so the correct approach is for instance methods to reference `this` with no arguments.

```javascript
// NG: passing your own property into an internal method
buildKey () {
  return this.formatName({
    name: this.name,
  })
}

// OK: the internal method references `this` directly
buildKey () {
  return this.formatName()
}

formatName () {
  return this.name.toUpperCase()
}
```

## A method that extracts resolves the value; it does not interpret it

**Where a method's job is to take a value out of something — a config, a payload, a
response — it returns what is there, and `null` where nothing is.** What absence means is
the caller's to decide, and deciding it inside the extractor hides the decision in the one
member whose name promises not to make any.

```javascript
// NG: the extractor decides that no cap means an unreachable one
static extractMaxDocumentDepth ({
  config,
}) {
  return config.maxDocumentDepth
    ?? Infinity
}

// OK: it resolves, and leaves the meaning to whoever asked
static extractMaxDocumentDepth ({
  config: {
    maxDocumentDepth = null,
  },
}) {
  return maxDocumentDepth
}
```

- **A default at the destructuring point is not interpretation.** `= null` states that an
  absent key and an explicit `null` arrive the same way; `?? Infinity` states what absence
  is worth, which is a different claim.
- **The caller skips the work rather than being handed a value that stands in for nothing.**
  Where the value is absent, the member that would have used it returns early, and the
  behaviour that depends on it does not run.

```javascript
// The consumer decides what an absent cap means
exceedsMaxDocumentDepth ({
  depth,
}) {
  if (!this.maxDocumentDepth) {
    return false
  }

  return depth > this.maxDocumentDepth
}
```

- Two members then each hold one thing: the extractor holds where the value comes from, and
  the consumer holds what its absence does. A substitute value inside the extractor merges
  the two, and the merged version reads as though no decision had been made.

## A recursion sits behind an entry point of its own

- How a recursion is laid out — the plain-named entry point, the `deep~` member behind it, and
  where the accumulator's initial value is written — belongs to `/hoc-methods-recursion`.
- **Both members are methods, so everything on this page holds for them in full.** The entry
  point and the `deep~` member each take a single named-argument object, the accumulator
  included.

## Factory methods must be defined without exception

- Class definitions must define a factory method without exception.
- Unless a class is inheriting, every class definition must define `static create (...)`.
  (When inheriting, the parent class's `create` is inherited, so it doesn't need to be redefined.)
- The parameters of `static create (...)` should define what is needed to construct the arguments passed to the constructor.
- Factory methods should, unless there is a specific reason not to, be implemented as **static methods**
  (`static create (...)`).

### Division of responsibility between the constructor and `static create (...)`

- What the constructor decides, and what it leaves to the factory methods, belong to
  `/hoc-classes-constructor`. It is read here for the two things that land in
  `static create (...)`: a value the caller did not supply gets its default here, and where the
  constructor's parameter list divides what goes up to the base from what the class keeps, the
  factory method's parameter list repeats that division.

### Variations should be distinguished by suffix

- When variations of `.create(...)` or `.createAsync(...)` are needed, distinguish them with a suffix.
  (e.g. `.createAsAlpha()` / `.createWithBeta()`)

### Placement order

- Where the factory methods sit in a class body belongs to `/hoc-classes-notations`.

### JSDoc format

- Unless there is a specific reason otherwise, the JSDoc of a factory method must follow the format below without exception.
  (Replace `ThisClass` with the name of the class being defined.)

```javascript
/**
 * Factory method.
 *
 * @template {X extends typeof ThisClass ? X : never} T, X
 * @param {...} [params] - Parameters for the factory method.
 * @returns {InstanceType<T>} Instance of this class.
 * @this {T}
 * @public
 */
```

- Since the factory method is an entry point called from outside, it is annotated with `@public`.

### Instantiation

- Within `.create(...)`, do not call `new` using the class name. Instantiate with `new this(...)`.
  (Using `this` ensures that even when called from an inheriting subclass, an instance of that subclass is created.)
- Since every defined class always has a factory method defined, whenever depending on another class, always instantiate it via its factory method (`.create(...)`).
- Consequently, the form `new Sample(...)` against a defined class never appears outside test files.
  (The exception is `new this(...)` inside `.create(...)`. This creates an instance of the class itself using `this`, not the class name.)

```javascript
// NG
static create (...) {
  return new RandomTextGenerator({ characters })
}

// OK
static create (...) {
  return new this({ characters })
}
```

#### Exception: direct `new` for built-in classes and third-party modules

- The exceptions where a direct `new` expression is allowed without going through a factory method (JavaScript built-in classes, DTO-like third-party modules, and the DTO whitelist) are collected in [references/instantiation.md](./references/instantiation.md).

### A dependency is built in a factory method of its own

- **A dependency `static create (...)` needs is never built in its default argument.** It is built
  by a dedicated factory method — `this.createExternalApiClient()` — instantiating through a
  `[TargetClassName]Ctor` getter. That structure, and the seams it leaves for patching and
  testing, belong to `/hoc-wire-dependencies`.

### When asynchronous creation is needed, define `.createAsync(...)`

- The convention for `.createAsync(...)` when the constructor arguments need to be generated asynchronously (delegation to `.create(...)`, JSDoc, and a complete example) is collected in [references/create-async.md](./references/create-async.md).
