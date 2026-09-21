---
name: hoc-scope
description: "Conventions for scope references among class members. Covers using `this` for references between static members, going through `#get:Ctor` when referring from an instance to static members, and the prohibition on a bare `this` on the right-hand side — neither bound to a local nor destructured for its properties. What a class may hold belongs to the class design principles convention."
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

## Do not write a bare `this` on the right-hand side

**`this` never stands alone on the right of an assignment.** Destructuring a property out of
it (`const { alpha } = this`), or binding the object itself (`const self = this`), is
prohibited. A property is read where it is used, as `this.alpha`.

```javascript
// NG: the property taken into a local first
const {
  maxDocumentDepth,
} = this

return depths.filter(it =>
  it > maxDocumentDepth
)

// OK: read where it is used
return depths.filter(it =>
  it > this.maxDocumentDepth
)
```

- **A local copy is a second name for one value.** From that line on, two things refer to
  the property and nothing keeps them together — and the reader has to carry the alias to
  understand the lines below it.
- **Where the local looks unavoidable, what is missing is a member.** The value is being
  held because some logic needs it in hand; give the class the member that performs that
  logic, and the property is read inside it as `this.alpha` again.

### Reading the property is also the faster one

Measured on Node v22.23.2, medians over 11–12 rounds:

| Shape | `this.value` | `const { value } = this` | Difference |
| :-- | --: | --: | --: |
| Three reads inside a hot loop | 2.78 ms | 2.87 ms | +3% |
| Two reads per call, 3,000,000 calls | 66.17 ms | 68.28 ms | +3% |
| `filter()` over 8 elements, 300,000 calls | 21.74 ms | 25.87 ms | **+19%** |

- The gap opens widest in the shape that tempts the local in the first place: **a
  callback**. A monomorphic property load folds into an inline cache, while the
  destructured binding is created on every call and, once a closure captures it, lives in
  the context object.

### The type checker is what usually induces it

A property typed `number | null` is narrowed by a guard, **and the narrowing does not reach
inside a callback** — the comparison there is reported as possibly null (`TS18047`). Taking
the property into a local is what makes that message go away, which is why the local gets
written.

```javascript
// The narrowing does not cross into the callback
someMethod ({ depths }) {
  if (!this.maxDepth) {
    return []
  }

  return depths.filter(it =>
    it > this.maxDepth // TS18047: 'maxDepth' is possibly 'null'
  )
}

// The member carries the guard and the comparison in one body
exceedsMaxDepth ({ depth }) {
  if (!this.maxDepth) {
    return false
  }

  return depth > this.maxDepth
}
```

- The answer is the member, not the local: inside it the guard and the comparison sit in
  the same function body, so the narrowing holds and nothing is copied.
