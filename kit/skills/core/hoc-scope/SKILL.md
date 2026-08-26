---
name: hoc-scope
description: "Conventions for scope references among class members. Covers using `this` for references between static members, and going through `#get:Ctor` when referring from an instance to static members."
---

# Classes: Shared / Scope

This summarizes conventions related to scope references among class members.

## References between static members should use `this`

- References between static members should use `this` rather than the class name.
  (Using `this` ensures that even when called from an inheriting subclass, the subclass's own definition is referenced.)

```javascript
// NG
static create ({
  characters = RandomTextGenerator.DEFAULT_CHARACTERS,
} = {}) {
  return new this({ characters })
}

// OK
static create ({
  characters = this.DEFAULT_CHARACTERS,
} = {}) {
  return new this({ characters })
}
```

## When referencing a static member from an instance member, go through `#get:Ctor`

- When referencing a static member from an instance method or getter, go through `this.constructor` rather than hardcoding the class name.
- Define the conventional getter `#get:Ctor`, which returns `this.constructor`, and reference the member in the form `this.Ctor.staticMember`.
- Using `this.constructor` ensures that even when called from an inheriting subclass, the subclass's own static definition is referenced. Hardcoding the class name would fix it to the base class's definition.
- In the body of `#get:Ctor`, **type resolution (a type cast) is needed** between `return` and `this.constructor`. Since the type of `this.constructor` is broad, cast it to the type of the class in question.

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
