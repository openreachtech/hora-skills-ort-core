---
name: hoc-wire-dependencies
description: "How a class reaches the modules and classes it depends on, so that a subclass can patch one and a test can swap it. Use when a class imports something it uses, builds a dependency in `static create()` or anywhere else, or a dependency has to be patched or mocked. How a getter's body is written belongs to the accessors convention; `static create()` in general, to the methods convention."
---

# Wiring Dependencies

A class reaches what it depends on through **seams it declares itself**: a static getter that
names the dependency, and — where the dependency is a class to instantiate — a static factory
method that builds it. Both are resolved through `this`, so a subclass overrides either one and
every call site follows without being touched.

## Core principle: a dependency is reached through a seam, never through its imported name

- When a module (the class itself) depends on another module — whether a native module, a
  third-party module, or an in-house module — do not use the reference to that dependency
  directly as the imported identifier. **Extract it into a getter.**
- Reason:
  - **Easy patching**: if the dependency has a bug and needs an emergency patch, it suffices to
    override the seam in a subclass; call sites do not need to change. If the dependency were
    used directly, every call site would need to explicitly pass the patched one.
  - **Easy testing**: in tests, swapping the getter replaces the dependency entirely, and
    `jest.spyOn(ThisClass, 'createExternalApiClient')` swaps a factory's return value — with the
    same call form as production.
  - **Encapsulation of the dependency**: the caller's concern is the class in question, not its
    internal dependencies. The seam hides the dependency relationship inside the class, lowering
    coupling.
- The seams are lightweight DI (Dependency Injection), achieving, without a DI container:
  production = creation of the real dependency / testing = swap / hotfix = swap to a patched
  dependency.

## Which seams a dependency takes

| The dependency | Seams |
| :-- | :-- |
| A class to instantiate via `new` / `.create(...)` | A static getter named `[TargetClassName]Ctor`, and a dedicated factory method that instantiates through it |
| A module used as it is, such as `fs` | A single static getter that returns the module as-is |

## A class to instantiate: the `Ctor` getter and the dedicated factory method

When the dependency is a **class to be instantiated** via `new` / `.create(...)`, extract it into
a static getter named `[TargetClassName]Ctor`, and instantiate it via a dedicated factory method.

```javascript
// NG: using a dependency class directly
import Aggregator from './Aggregator.js'

export default class BaseRewardCalculator {
  constructor ({
    config,
    aggregator,
  }) {
    this.config = config
    this.aggregator = aggregator
  }

  static create ({
    config,
  } = {}) {
    const aggregator = Aggregator.create({ config }) // NG: instantiated directly

    return new this({
      config,
      aggregator,
    })
  }
}
```

```javascript
// OK: extract into a static getter named [TargetClassName]Ctor, and instantiate via a dedicated factory method
import Aggregator from './Aggregator.js'

export default class BaseRewardCalculator {
  constructor ({
    config,
    aggregator,
  }) {
    this.config = config
    this.aggregator = aggregator
  }

  static create ({
    config,
  } = {}) {
    const aggregator = this.createAggregator({ config }) // OK: via a dedicated factory method

    return new this({
      config,
      aggregator,
    })
  }

  static get AggregatorCtor () {
    return Aggregator
  }

  static createAggregator ({
    config,
  }) {
    return this.AggregatorCtor.create({ config })
  }
}
```

- When applying an emergency patch in a subclass, overriding `AggregatorCtor` alone suffices.

```javascript
// OK: swapping the corrected class in only requires overriding AggregatorCtor
import RoundHalfUpAggregator from './RoundHalfUpAggregator.js'

export default class UserRewardCalculator extends BaseRewardCalculator {
  /** @override */
  static get AggregatorCtor () {
    return RoundHalfUpAggregator
  }
}
```

### Name the getter `[TargetClassName]Ctor`

- When holding a target class's constructor in a static getter for delegation purposes, the getter
  name should be "**target class name + `Ctor`**".
- Example: for the `Constraint` class's constructor → `.get:ConstraintCtor`; for `Client` →
  `.get:ClientCtor`.
- When the target class is undetermined, such as in an abstract base class, a generic name
  expressing the role (e.g. `TargetCtor`) is fine.

```javascript
// OK: static getter holding the constructor of the delegate target class ([TargetClassName]Ctor)
static get ConstraintCtor () {
  return SampleConstraint
}
```

### Build the dependency in the dedicated factory method, never in a default argument

- Do not directly create a dependency class in a default argument of `.create(...)`, etc. That is,
  do not directly write either `new Dependency(...)` (a `new` expression) or
  `Dependency.create(...)` (calling the dependency class's factory method). Extract the creation of
  the dependency class into a dedicated factory method (e.g. `this.createExternalApiClient()`) and
  go through it.
- Unless there is a specific reason otherwise, the name of the dedicated factory method should
  basically be "`create` + class name" (e.g. `ExternalApiClient` → `createExternalApiClient`).

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

### A dependency built on the fly goes through the factory method too

- **A class held and used by delegation takes the factory method wherever it is built**, including
  when an instance method creates one temporarily and discards it. Do not `new` it there either.
- This holds for a class defined within the application as much as for a third-party one: what
  decides it is that the instance is used by delegating to it, not where the class comes from.

```javascript
// NG: a delegate built on the fly with a direct new
sendRequest () {
  const client = new ExternalApiClient({ env })

  return client.send()
}

// OK: built through the dedicated factory method
sendRequest () {
  const client = this.createExternalApiClient()

  return client.send()
}
```

## A module used as it is: a static getter that returns it

- When the dependency is a native module such as `fs` — where the module itself is the value and
  there is no instantiation via `new` / `.create(...)` — neither the `Ctor` suffix nor a dedicated
  factory method is needed. Define a single **static getter** that returns the dependency module
  as-is.
- **It is static because the body never touches `this`.** `return fs` is the whole of it, so
  nothing in it belongs to an instance. What decides the kind of a getter is its body, not the
  kind of value it holds — a getter that reaches for no instance state is a static getter,
  whatever it returns.
- **An instance reaches it through `#get:Ctor`**, as `this.Ctor.fs` — see `/hoc-classes-ctor`.
- Calling a function of the module from within the getter is prohibited, per the accessors
  convention's rule against calling a method from a getter. The getter must return nothing but the
  module reference itself.

```javascript
// OK: a static getter that returns a native module as-is, reached through #get:Ctor
import fs from 'node:fs'

export default class DeepLoader {
  static get fs () {
    return fs
  }

  get Ctor () {
    return /** @type {typeof DeepLoader} */ (this.constructor)
  }

  collectFileNames ({
    poolPath = this.poolPath,
  } = {}) {
    return this.Ctor.fs.readdirSync(poolPath)
      .filter(it => !it.startsWith('.'))
  }
}
```

## Consolidating the creation of a frequently used class

- The `XxxxFactory` pattern for consolidating creation of a frequently used dependency class (with
  the `BaseFactory` example), and the intent behind the selection (`.get:TargetCtor`) /
  instantiation (`.createTarget()`) two-stage separation, are collected in
  [references/factory-class.md](./references/factory-class.md).

## Where this stops

- **How a getter's body is written** — no branching, no method call — belongs to the accessors
  convention. **`#get:Ctor`, and the reservation of its name**, belong to `/hoc-classes-ctor`.
- **`static create (...)` in general** — that every class defines one, `new this(...)`, which
  defaults it applies, and when a direct `new` is allowed — belongs to the methods convention.
- **What an object hands its members** — one shared manifest rather than each member's own values
  — belongs to the manifest pattern convention.
