---
name: hoc-accessors
description: "Conventions for class accessors, getters and setters alike. Use when adding or reviewing an accessor, a getter that holds a dependency included. Which getter a dependency takes, and the factory method beside it, belong to the dependency-wiring convention; where members sit in the class body, to the class notation convention."
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

## Getters that wire dependencies

- A class reaches what it depends on through static getters — `[TargetClassName]Ctor` for a class
  it instantiates, a getter returning the module as-is for one it uses directly. Which getter a
  dependency takes, and the factory method beside it, belong to `/hoc-wire-dependencies`.
- **They are getters, so everything below holds for them in full.** A `Ctor` getter returns the
  class and nothing else; the `.create(...)` call belongs in the factory method, never in the
  getter.

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
- Exception: an abstract getter that requires an override and declares itself unimplemented by
  throwing is out of scope, whichever of the two throws the error-handling convention gives it.
  **That covers the one that reaches the module's own error class through a factory method** —
  a call inside a getter, and the exception would be worth nothing without it, since the getter
  never returns. What the throw looks like follows the error-handling convention.
