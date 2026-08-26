---
name: hoc-methods
description: "Conventions for class method definitions. Covers named arguments, passing properties into private methods, factory methods, and related policies."
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

- Binding (inflator) methods do not use a named-argument object; they receive **the arguments to pass, flat**.
- An inflator method is a static method that binds the class passed as an argument and returns a memoized derived subclass (following the naming convention described later, such as `.use()` / `.of()` / `.to()`).
- Basically it receives **a single argument**. Only use multiple/array arguments when dealing with variadic or array input.
  - Single argument: `use(ConstraintCtor)` / `as(Schema)` / `toKey(MessageKeyCtor)` / `toValue(MessageValueCtor)`
  - Variadic: `of(...Ctors)`
  - Single array: `from(Ctors)` (it is standard practice for `from` to delegate to `of`)

```javascript
// NG: named-argument object
static use ({
  ConstraintCtor,
}) {
  // ...
}

// OK: flat single argument
static use (ConstraintCtor) {
  // ...
}

// OK: variadic / array arguments
static of (...Ctors) {
  // ...
}

static from (Ctors) {
  return this.of(...Ctors)
}
```

- Reason: this is so that calls read **declaratively**, as in `Document.as(bindingSchema)` or `UnionScalar.of(A, B)`. Wrapping them in a named-argument object would undermine this declarative feel.

#### Naming convention (short, preposition-like names)

- Inflator methods should be given short, preposition-like names (so that `Receiver.word(binding)` reads declaratively). Examples: `.as()` / `.use()` / `.of()` / `.to()`, etc.
- These are only representative examples, not the full vocabulary. The full vocabulary — the semantics of each word, its arguments, the forward vs. graft distinction, and how `.toKey()` / `.toValue()` are used — is owned by the inflator-methods convention as its single source of truth; it is not duplicated here.

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

## Factory methods must be defined without exception

- Class definitions must define a factory method without exception.
- Unless a class is inheriting, every class definition must define `static create (...)`.
  (When inheriting, the parent class's `create` is inherited, so it doesn't need to be redefined.)
- The parameters of `static create (...)` should define what is needed to construct the arguments passed to the constructor.
- Factory methods should, unless there is a specific reason not to, be implemented as **static methods**
  (`static create (...)`).

### Division of responsibility between the constructor and `static create (...)`

- The constructor should receive required arguments (e.g. `{ characters }`). It should not have default values.
- Applying default values for arguments is the responsibility of `static create (...)`.

### Variations should be distinguished by suffix

- When variations of `.create(...)` or `.createAsync(...)` are needed, distinguish them with a suffix.
  (e.g. `.createAsAlpha()` / `.createWithBeta()`)

### Placement order

- `static create (...)` should be placed immediately after the constructor.
- If `static createAsync (...)` is defined, it should in principle be placed immediately after `.create(...)`.

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

#### `XxxxFactory` class and the two-stage separation

- The `XxxxFactory` pattern for consolidating creation of a frequently used class (with the `BaseFactory` example), and the intent behind the selection (`.get:TargetCtor`) / instantiation (`.createTarget()`) two-stage separation, are collected in [references/factory-class.md](./references/factory-class.md).

### Instantiation of a dependency class should go through a factory method

- Do not directly create a dependency class in a default argument of `.create(...)`, etc. That is, do not directly write either `new Dependency(...)` (a `new` expression) or `Dependency.create(...)` (calling the dependency class's factory method). Extract the creation of the dependency class into a dedicated factory method (e.g. `this.createExternalApiClient()`) and go through it.
- Unless there is a specific reason otherwise, the name of the dedicated factory method should basically be "`create` + class name" (e.g. `ExternalApiClient` → `createExternalApiClient`).

```javascript
// NG: directly instantiating a dependency class
static create ({
  externalApiClient = ExternalApiClient.create({ env }),
} = {}) {
  // ...
}

// OK: go through a factory method
static create ({
  externalApiClient = this.createExternalApiClient(),
} = {}) {
  // ...
}
```

Reason:

- **Patching/overriding is easy**: if a dependency has a bug and you need to apply an emergency patch in a subclass, you only need to override the factory method in the subclass, without changing the call sites. If the dependency were instantiated directly, every call site would need to explicitly pass the patched instance.
- **Encapsulation of the dependency**: the caller's concern is the class in question, not its internal dependencies. The factory method hides the dependency relationship inside the class, lowering coupling.
- **Testing/DI is easy**: you can inject a mock via an argument, or swap the factory's return value with `jest.spyOn(ThisClass, 'createExternalApiClient')`, and test with the same call form as production.
- The factory method is lightweight DI (Dependency Injection), achieving, without a DI container: production = creation of the real dependency / testing = swap / hotfix = swap to a patched dependency.

### When asynchronous creation is needed, define `.createAsync(...)`

- The convention for `.createAsync(...)` when the constructor arguments need to be generated asynchronously (delegation to `.create(...)`, JSDoc, and a complete example) is collected in [references/create-async.md](./references/create-async.md).
