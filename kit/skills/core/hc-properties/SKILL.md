---
name: hc-properties
description: "Conventions for class property definitions. Covers setting properties on `this` within the constructor, immutability (no reassignment, prohibiting Map), and the policy of not using JavaScript native private."
---

# Classes: Members / Properties

This summarizes conventions related to class property definitions.

## Set properties on `this` within the constructor

- Properties should be set on `this` within the constructor.
- To write tests against properties, properties should be kept accessible from outside.

```javascript
// OK: set on this within the constructor
constructor ({
  delimiter,
}) {
  this.delimiter = delimiter
}
```

## Classes are immutable / property reassignment is prohibited

- Basically, all classes are implemented as immutable. Once a property is set in the constructor, it must **not be reassigned** thereafter.
- `this.xxx = ...` within the constructor (the initial set) is permitted. ESLint also does not prohibit this.
- What is prohibited is **property reassignment outside the constructor**. This is enforced by ESLint.
- Being immutable means that even a property with public access scope is "protected by coding rules." Hence there is
  no need to make it native private for encapsulation purposes (for details, see "The meaning of `#alpha` notation and
  the treatment of native private" below).
- **Updating a collection (Array/Set) itself is permitted. What is prohibited is a structure that references individual elements** — pulling out a single element via `array[i]` and treating it as mutable state. This subverts the prohibition on mutable objects and is not permitted. A collection's value must always be "used all at once" (scanned/transformed/aggregated over every element as a whole). When you want to change scalar state, generate a new instance via a factory method (for the policy of not deep-freezing collections, see the class design principles convention).

```javascript
// NG: reassigning a property after creation
scalar.normalizedValue = anotherValue

// OK: if a different state is needed, generate a new instance via a factory method
const next = Scalar.create({
  normalizedValue: anotherValue,
})
```

### `Map` is prohibited; `WeakMap` is free to use

- **`Map` is prohibited (enforced by ESLint).** Do not use it unless there is a special reason in module development. In application code, there has been no case where `Map` was used other than as an evasion.
- `WeakMap`, on the other hand, may be used freely. Associating objects by identity is `WeakMap`'s responsibility, and enumeration is unnecessary (when you want to enumerate, hold the keys in an Array and traverse through them).
- Making a `Map`'s key a primitive value (`number` / `string`, etc.) is a circumvention of the reassignment prohibition and the prohibition on mutable objects. An object-keyed `Map` is also unnecessary: if enumeration is not needed, `WeakMap` suffices, and even if it is, an Array of keys + `WeakMap` covers it — so there is no reason to choose `Map`.
- If a mutable aggregate seems necessary, reconsider the design (assemble it via a higher-order function and return it, or generate a new instance via a factory method).

## The meaning of `#alpha` notation and the treatment of native private

- `#` notation such as `#alpha` is not a JavaScript native private designation; it means "instance-private" in member notation.
  (Member notation follows "Notation of Class Members" in the documentation convention.)
- **JavaScript native private fields (`#` fields / `#` methods) must never be used unless a human specifically instructs it.**
- Therefore, even if an instruction says `#alpha`, that alone is not a reason to implement it as a `#` field. It is normally defined as `this.alpha`.

### Reason

- Native private cannot be read even from an inheriting subclass, which blocks extension/substitution via inheritance.
- This codebase basically implements all classes as immutable and does not permit reassignment of properties at all (this is also prohibited by ESLint). Therefore, even with a public access scope, it is protected by coding rules, and there is no need to make it native private.
- Rather, it is more beneficial to avoid the disadvantage where, when you want to apply a patch (such as a hotfix) that temporarily changes behavior via inheritance, a parent class's private property would block that.
