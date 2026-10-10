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
2. **Delegate-style functional classes** (classes held in a property and used by delegating their functionality) are not an exception: they are dependencies, and how they are built — through a dedicated factory method, wherever they are created, and through a factory class where one class is built across many — belongs to `/hoc-wire-dependencies`.

```javascript
// OK (1): DTO-like modules may be instantiated directly with new
const amount = new BigNumber(value)
```

## DTO whitelist

- The following third-party modules are considered DTOs and may be freely instantiated directly with `new` (treatment (1)). Add to the whitelist as needed.

| module | class |
| :-- | :-- |
| bignumber.js | `BigNumber` |
