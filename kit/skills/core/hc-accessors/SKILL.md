---
name: hc-accessors
description: "Conventions for class accessor (getter/setter) definitions: setters are prohibited for immutability; `#get:Ctor` is reserved for `this.constructor`; dependency references are extracted into getters — a static `[TargetClassName]Ctor` for classes to instantiate, or a plain instance getter for non-instantiated dependencies like native modules; getter bodies forbid branching and method calls, staying pure property references."
---

# Classes: Members / Accessors

This summarizes conventions related to class accessor (getter / setter) definitions.

## Setters are prohibited

- Do not define setters (`set xxx () {}`).
- Reason: this codebase implements all classes as immutable (properties are set once in the constructor and never
  reassigned). A setter would allow reassignment after creation, breaking the premise of immutability.
- When you want to change state, instead of rewriting via a setter, generate a new instance via a factory method (see "Classes are immutable / property reassignment is prohibited" in the property-definition convention).
- Exception: for arguments passed to `Proxy` (such as the handler in `new Proxy(target, handler)`), a `set` trap may
  be defined as needed. This is not a reassignment of a class property but the definition of `Proxy` behavior, and
  does not conflict with the intent of immutability.

## Reserve `#get:Ctor` as a conventional getter

- `#get:Ctor` is reserved as the conventional getter that "returns `this.constructor`."
  (For usage, type resolution, and override details, see the scope-reference convention.)
- Therefore, do not use the name `Ctor` as a member name for any other purpose.

## Extract references to dependency modules into a getter

- When a module (the class itself) depends on another module — whether a native module, a third-party module, or an in-house module — do not use the reference to that dependency directly as the imported identifier. **Extract it into a getter.**
- Reason:
  - **Easy patching**: if the dependency module has a bug and needs an emergency patch, it suffices to override the getter in a subclass; call sites do not need to change.
  - **Easy mocking**: in tests, swapping the getter is enough to replace the dependency module entirely.
- When the dependency is a **class to be instantiated** via `new` / `.create(...)`, extract it into a static getter named `[TargetClassName]Ctor` per the next section, and instantiate via a dedicated factory method (see "Instantiate dependency classes via a factory method" in the method-definition convention).

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
// OK: swapping in the patched class only requires overriding AggregatorCtor
import PatchedAggregator from './PatchedAggregator.js'

export default class UserRewardCalculator extends BaseRewardCalculator {
  /** @override */
  static get AggregatorCtor () {
    return PatchedAggregator
  }
}
```

## Static getters holding a constructor used for delegation should be named `[TargetClassName]Ctor`

- When holding a target class's constructor in a static getter for delegation purposes, the getter name should be "**target class name + `Ctor`**".
- Example: for the `Constraint` class's constructor → `.get:ConstraintCtor`; for `Client` → `.get:ClientCtor`.
- When the target class is undetermined, such as in an abstract base class, a generic name expressing the role (e.g. `TargetCtor`) is fine.

```javascript
// OK: static getter holding the constructor of the delegate target class ([TargetClassName]Ctor)
static get ConstraintCtor () {
  return SampleConstraint
}
```

## Native modules and other dependencies that involve no instantiation are returned as-is via an instance getter

- When the dependency is a native module such as `fs` — where the module itself is the value and there is no instantiation via `new` / `.create(...)` — neither the `Ctor` suffix nor a dedicated factory method is needed. Define a single **instance getter** that returns the dependency module as-is.
- Calling a function of the module from within the getter is prohibited, per the "do not call a method from a getter" convention. The getter must return nothing but the module reference itself.

```javascript
// OK: an instance getter that returns a native module as-is
import fs from 'fs'

export default class DeepLoader {
  get fs () {
    return fs
  }

  collectFileNames ({
    poolPath = this.poolPath,
  } = {}) {
    return this.fs.readdirSync(poolPath)
      .filter(it => !it.startsWith('.'))
  }
}
```

## Do not write branching inside a getter

- Do not write **branching** (`if` statements, ternary operators, etc.) in the body of a getter.
- One of a getter's responsibilities is **drilling down into properties** to resolve the Law of Demeter. Confine deep property chains within the getter, keeping the caller shallow.
- A getter must not return `undefined`. When a value cannot be obtained, resolve it to `null` with `?? null`.
  - This `??` is itself a kind of branching, but since the condition is limited solely to "identifying `undefined`," it is permitted as an exception.
- When drilling down into properties, treat `this.xxxx` as the receiver, and follow "one property chain per line" from the coding-styles convention. That is, chop it down so that there is **at most one receiver per line, and at most one property call per line**.

```javascript
// OK: this.entity is the receiver, one property per line. undefined is resolved to null with ?? null
get authorId () {
  return this.entity.comment
    ?.author
    ?.id
    ?? null
}
```

- When drilling down into properties, if **the leading lines of the chain recur across multiple getters**, extract that leading part into a separate getter. The subsequent getters then continue using the extracted getter as their receiver.

```javascript
// NG: the leading `this.entity.comment ?.author` is duplicated across multiple getters
get authorId () {
  return this.entity.comment
    ?.author
    ?.id
    ?? null
}

get authorName () {
  return this.entity.comment
    ?.author
    ?.name
    ?? null
}

// OK: extract the common leading part into a getter, and use it as the receiver from then on
get author () {
  return this.entity.comment
    ?.author
    ?? null
}

get authorId () {
  return this.author?.id
    ?? null
}

get authorName () {
  return this.author?.name
    ?? null
}
```

## Do not call a method from a getter

- Do not **call a method** from the body of a getter. The receiver does not matter: `this.xxxx()`, `this.Ctor.xxxx()`, a method on an object reached by drilling down, and a global function are all prohibited.
- Reason: from the caller's side, a getter looks like a **property access** (`this.authorLabel`). Calling a method behind it makes it **implicit, from the caller's side, that processing is running**. What looks like a property must be complete as a property reference alone.
- A getter's responsibility is limited to the property drilling of the previous section. When processing is needed, define it as a method and let the caller call it as a method.

```javascript
// NG: calling a method from a getter. The caller's this.authorLabel looks like a property, yet formatting runs behind it
get authorLabel () {
  return this.buildLabel(this.author)
}

// OK: keep the getter to a property reference, and let the caller call the formatting as a method
get author () {
  return this.entity.comment
    ?.author
    ?? null
}

buildAuthorLabel () {
  return this.buildLabel(this.author)
}
```

- When the value reached by drilling down is a function, **returning it without calling it** is out of scope. What is prohibited is calling within the getter; returning a function as a value is not restricted.
- Exception: an abstract getter that requires an override and declares itself unimplemented via `throw new Error()` is out of scope. Its format follows the error-handling convention.
