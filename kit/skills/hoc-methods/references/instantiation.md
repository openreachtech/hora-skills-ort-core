# Exceptions to direct `new` without a factory method

This covers the exceptions where a direct `new` expression is allowed without going through a factory method (`.create(...)`) — JavaScript built-in classes and DTO-like third-party modules — along with the DTO whitelist. Referenced from the method-definition convention itself (`SKILL.md`).

## Exception: JavaScript built-in classes

- JavaScript's built-in classes (`Date` / `WeakMap` / `Set` / `RegExp` / `Error`, etc.) are not subject to the `new`-expression restriction. They may be freely instantiated directly with `new`, without going through a dedicated factory method.
  - `Map` is the exception. Even though it is a built-in class, `Map` is entirely prohibited and must not be instantiated directly with `new` either (see "`Map` is prohibited; `WeakMap` is free" in the property-definition convention). Use `WeakMap` when association is needed.
- These are neither classes defined by this codebase nor its dependency classes, and they have no `.create(...)` factory method, so they fall outside this convention.

```javascript
// OK: built-in classes may be instantiated directly with new
const date = new Date(value)
const pattern = new RegExp(source, 'ug')

throw new Error(`${this.constructor.name}#normalizeValue() must be inherited`)
```

## Third-party modules

- Third-party modules are subject to the factory-method `new`-expression restriction **depending on their intended purpose**.

1. Modules that behave as **DTOs** (e.g. `BigNumber`) are treated the same as built-in classes. They may be freely instantiated directly with `new`, without going through a dedicated factory method.
   - However, writing a direct `new` expression is allowed **only if the module does not provide a factory method** such as `Model.build()` / `Sample.create()`. If a factory method is provided, use that factory method instead of a `new` expression.
   - When it is hard to determine whether something is DTO-like, **define a factory method instead of asking a
     human**.
   - If the same class is instantiated **frequently across multiple classes**, introduce a new `XxxxFactory` class. It holds the target class's constructor as a property and **performs the `new` expression through that factory class**. If a common base `BaseFactory` class exists, use it (via inheritance); if not, provide a **generic implementation** that doesn't depend on a specific class.
2. For **delegate-style functional classes** (classes held in a property and used by delegating their functionality), basically implement a factory method to go through (following "instantiation of a dependency class should go through a factory method").
3. Delegate-style functional classes require a factory method **regardless of whether they are created on the fly within an instance method**. Even when created temporarily within a method, do not `new` them directly; go through a dedicated factory method (e.g. `this.createExternalApiClient()`).

- Note that this principle (2)(3) is **not limited to third-party modules**. Even for classes defined within the application, if the structure delegates use of an instance, instantiation via a factory method is equally required.

```javascript
// OK (1): DTO-like modules may be instantiated directly with new
const amount = new BigNumber(value)

// NG (2)(3): directly `new`-ing a delegate-style functional class (not allowed even when created on the fly)
sendRequest () {
  const client = new ExternalApiClient({ env })

  return client.send()
}

// OK (2)(3): go through a factory method
sendRequest () {
  const client = this.createExternalApiClient()

  return client.send()
}
```

## DTO whitelist

- The following third-party modules are considered DTOs and may be freely instantiated directly with `new` (treatment (1)). Add to the whitelist as needed.

| module | class |
| :-- | :-- |
| bignumber.js | `BigNumber` |
