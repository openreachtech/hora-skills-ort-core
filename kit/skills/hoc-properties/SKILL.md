---
name: hoc-properties
description: "Conventions for class properties — where they are set, whether they may change, and what may hold them. Use when declaring, assigning or reviewing a property of a class, including when a note writes a member as `#alpha`. `Map` and native private belong to the native-feature prohibition convention; accessors, to the accessors convention; whether a class needs properties at all, to the class design principles convention."
---

# Classes: Members / Properties

This summarizes conventions related to class property definitions.

## Set properties on `this` within the constructor

- That a property is set only as `this.xxx = xxx` inside the constructor, and why class fields
  are not used, belong to `/hoc-classes-principles`.
- It is read here for what it means to tests: a property set that way stays accessible from
  outside, so a test can read it directly.

## Classes are immutable / property reassignment is prohibited

- Basically, all classes are implemented as immutable. Once a property is set in the constructor, it must **not be reassigned** thereafter.
- `this.xxx = ...` within the constructor (the initial set) is permitted. ESLint also does not prohibit this.
- What is prohibited is **property reassignment outside the constructor, including anything under
  the property's object path** — `this.state.count += 1` as much as `this.state = ...`. ESLint
  enforces the direct form; what lies under the path is held by this rule alone.
- Being immutable means that even a property with public access scope is "protected by coding rules." Hence there is
  no need to make it native private for encapsulation purposes (for why, see `/hoc-prohibit-native-features`).

```javascript
// NG: reassigning a property after creation
scalar.normalizedValue = anotherValue

// NG: reassigning under a property's object path
this.state.count += 1

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
