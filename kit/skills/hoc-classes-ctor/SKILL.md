---
name: hoc-classes-ctor
description: "The `Ctor` getter an instance reaches its own class through: how `#get:Ctor` is defined, typed and overridden, why the name is reserved, and why `this.constructor` is never written outside it. Use when an instance member refers to a static member or to its own class, or when about to write `this.constructor`. Getters that hold a dependency's class belong to the dependency-wiring convention."
---

# Classes: Ctor

How an instance reaches its own class. `Ctor` is the library's word for a class held as a value,
and `#get:Ctor` is the one getter that hands an instance its own.

## An instance reaches its class through `#get:Ctor`

- **When an instance method or getter refers to a static member, it goes through `this.Ctor`.**
  Neither the class name nor `this.constructor` is written there: for a static member,
  `this.constructor` appears in one place only, the body of `#get:Ctor`.
- Hardcoding the class name fixes the reference to the base class's definition. Going through
  the instance's own constructor is what lets a call from an inheriting subclass reach the
  subclass's own static definition.
- `this.constructor` written directly reaches the same class, but its type is too broad for the
  checker to know which static members exist. `#get:Ctor` is where that type is settled, once.
- In the body of `#get:Ctor`, **type resolution (a type cast) is needed** between `return` and
  `this.constructor`: cast it to the type of the class in question.
- **`#get:Ctor` is an adapter that exists to resolve the type.** A native property of the
  constructor that needs no type resolution — `this.constructor.name` above all — is read
  directly, as the class name in an error message is:
  `` throw new Error(`${this.constructor.name}#normalizeValue() must be inherited`) ``.

```javascript
/**
 * get: Constructor of this class.
 *
 * @returns {typeof SampleClass} Constructor of this class.
 */
get Ctor () {
  return /** @type {typeof SampleClass} */ (this.constructor)
}

someInstanceMethod () {
  return this.Ctor.someStaticMethod()
}

// NG: this.constructor written outside #get:Ctor
someInstanceMethod () {
  return this.constructor.someStaticMethod()
}
```

### When a derived class has more static members to reference, override `#get:Ctor`

- When a derived class gains additional "static members referenced from instance members," override and redefine `#get:Ctor` in the derived class, resolving the type to the derived class's type.

```javascript
class DerivedSample extends BaseSample {
  /** @override */
  get Ctor () {
    return /** @type {typeof DerivedSample} */ (this.constructor)
  }

  someDerivedMethod () {
    return this.Ctor.derivedStaticMethod()
  }
}
```

### Do not define a `#get:xxxx` shortcut for `this.Ctor.xxxx`

- Do not define a getter (`#get:xxxx`) solely to access `this.Ctor.xxxx` (a static member). Write static member access explicitly as `this.Ctor.xxxx`.
- Reason: inserting such a getter makes it **implicit, from the caller's `this.xxxx`, that this is a static-member call**.

```javascript
// NG: a shortcut getter for this.Ctor.DEFAULT_VALUE (makes the static reference implicit)
get defaultValue () {
  return this.Ctor.DEFAULT_VALUE
}

// OK: reference static members explicitly via this.Ctor
someInstanceMethod () {
  return this.Ctor.DEFAULT_VALUE
}
```

## The name `Ctor` is reserved

- `#get:Ctor` is reserved as the conventional getter that returns `this.constructor`.
- Therefore, do not use the name `Ctor` as a member name for any other purpose.
- `Ctor` is a getter, so the accessors convention holds for it in full: its body is the cast and
  nothing else.
