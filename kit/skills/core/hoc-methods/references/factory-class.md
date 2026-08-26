# `XxxxFactory` class and the two-stage separation

This covers the `XxxxFactory` pattern, which consolidates creation of a frequently used dependency class into one place, and the intent behind separating its creation into "selection (`.get:TargetCtor`)" and "instantiation (`.createTarget()`)." Referenced from the method-definition convention itself (`SKILL.md`).

## Example of an `XxxxFactory` class

- Consolidate creation of a frequently used class into one place, confining the `new` expression to a factory class. Inherit from the common base `BaseFactory`, and have the concrete class fix the target class solely by overriding `.get:TargetCtor`.

```javascript
/**
 * Base class of factories.
 *
 * @abstract
 */
class BaseFactory {
  constructor ({
    TargetCtor,
  }) {
    this.TargetCtor = TargetCtor
  }

  static create ({
    TargetCtor = this.TargetCtor,
  } = {}) {
    return new this({
      TargetCtor,
    })
  }

  static get TargetCtor () {
    throw new Error(`${this.name}.get:TargetCtor must be inherited`)
  }

  createInstance (...args) {
    return new this.TargetCtor(...args)
  }
}
```

```javascript
import SampleClient from './SampleClient.js'

class SampleFactory extends BaseFactory {
  /** @override */
  static get TargetCtor () {
    return SampleClient
  }

  createSampleClient ({
    options,
  }) {
    return this.createInstance(options)
  }
}
```

- `#TargetCtor` holds the constructor of the target class, and the `new` expression is consolidated into a single point, `BaseFactory#createInstance()`.
- `SampleFactory` does not redefine `create`; it inherits it from the parent, and fixes the target class solely by overriding `.get:TargetCtor`. This means swapping implementations (test doubles, patches) requires only overriding the getter.

## Intent behind the two-stage separation (`.get:TargetCtor` and `.createTarget()`)

- The creation of a delegate class is separated into two stages: **`.get:TargetCtor` (which class = selection)** and **`.createTarget()` (how to create it = instantiation)**. These are orthogonal, independent concerns, each serving as its own override / test seam.
- **`.get:TargetCtor` (the selection seam)**: consolidates "which class" the delegate target is into a single point. Since it is resolved via `this`, a subclass overriding it swaps the target class (polymorphism). In tests, `jest.spyOn(Factory, 'TargetCtor', 'get').mockReturnValue(StubClient)` swaps out **only the class**.
- **`.createTarget()` (the instantiation seam)**: the **single place** where `new this.TargetCtor(...)` is executed. Building and formatting arguments is also confined here. When you want to change "how it's created," you only need to override this method.
- **Why not merge them**: merging them into a single method would mean that even a "I just want to swap the class" change forces you to rewrite the creation logic as well, and vice versa. Keeping them separate makes the override points independent, each with a single responsibility. The `new` expression is confined to the single point of `.createTarget()`, and the class reference to the single point of `.get:TargetCtor`, isolating the volatile "which class" decision into the getter.
- This achieves, without a DI container (lightweight DI): production = real class creation / testing = class swap (getter) or creation swap (`.createTarget()`) / hotfix = overriding either one.
