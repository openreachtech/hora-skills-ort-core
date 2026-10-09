---
name: hoc-properties
description: "Conventions for class properties — where they are set, whether they may change, and what may hold them. Use when declaring, assigning or reviewing a property of a class, including when a note writes a member as `#alpha` or a value is about to be kept in a `Map`. Accessors belong to the accessors convention; whether a class needs properties at all, to the class design principles convention."
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
  no need to make it native private for encapsulation purposes (for why, see `/hoc-classes-principles`).

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

### A collection is built, then used all at once

**Code that makes an instance mutable is prohibited, uniformly — the reassignment prohibition is
one means to that, and a collection is no exception.** The one thing an Array or a Set allows is
being built: elements are added while the collection is assembled, and the finished collection is
then used all at once, every element together. A collection added to or taken from between reads,
while the instance is in use, is state that changes; the change goes to a new instance through
the factory method instead.

This holds wherever the collection sits — a property, or anything under its object path — and a
Set is held to it exactly as an Array is.

- **Its elements are used through a higher-order function** — `map()` / `filter()` /
  `reduce()` and the like. Taking an individual element out — `[n]` on an Array,
  `values().next()` or `[...set][0]` on a Set — is not using the collection all at once.
- **An element is never replaced in place.** Taking a position — `indexOf()`, `findIndex()` —
  and replacing what sits there with `splice(index, 1, newEntity)` is `array[index] = newEntity`
  under another name. The changed collection is another array, built with `map()`.
- **Its count is asked of the class that holds it.** `.length` / `.size` is read only inside that
  class, in a member that answers the count — `get entityCount ()` — and whether it is empty is
  answered the same way, by `isEmpty ()`. A caller handed the collection asks the holder rather
  than reading its length itself.
- **A collection is held for its elements.** An array whose only use is its `length` is not
  being used as an array; what it holds is a number.
- For the policy of not deep-freezing collections, see the class design principles convention.

```javascript
// NG: the collection grows while the instance is in use
addEntity ({
  title,
}) {
  this.entities.push(
    TodoEntity.create({
      title,
    })
  )
}

// OK: the changed collection goes to a new instance
addEntity ({
  title,
}) {
  return this.Ctor.create({
    entities: [
      ...this.entities,
      TodoEntity.create({
        title,
      }),
    ],
  })
}

// NG: the element at a position is replaced in place
markEntityDone ({
  title,
}) {
  const index = this.entities.findIndex(it => it.title === title)

  this.entities.splice(index, 1, this.entities.find(it => it.title === title).generateDone())
}

// OK: another array, held by a new instance
markEntityDone ({
  title,
}) {
  return this.Ctor.create({
    entities: this.entities
      .map(it => (
        it.title === title
          ? it.generateDone()
          : it
      )),
  })
}

// NG: the caller reads the length of a collection it was handed
const count = todoList.entities.length

// OK: the holder answers its count and its emptiness
get entityCount () {
  return this.entities.length
}

isEmpty () {
  return this.entities.length === 0
}
```

### `Map` is prohibited; `WeakMap` is free to use

- **`Map` is prohibited (enforced by ESLint).** Do not use it unless there is a special reason in module development. In application code, there has been no case where `Map` was used other than as an evasion.
- `WeakMap`, on the other hand, may be used freely. Associating objects by identity is `WeakMap`'s responsibility, and enumeration is unnecessary (when you want to enumerate, hold the keys in an Array and traverse through them).
- Making a `Map`'s key a primitive value (`number` / `string`, etc.) is a circumvention of the reassignment prohibition and the prohibition on mutable objects. An object-keyed `Map` is also unnecessary: if enumeration is not needed, `WeakMap` suffices, and even if it is, an Array of keys + `WeakMap` covers it — so there is no reason to choose `Map`.
- If a mutable aggregate seems necessary, reconsider the design (assemble it via a higher-order function and return it, or generate a new instance via a factory method).

## The meaning of `#alpha` notation and the treatment of native private

- `#` notation such as `#alpha` is not a JavaScript native private designation; it means "instance-private" in member notation.
  (Member notation follows "Notation of Class Members" in the documentation convention.)
- **JavaScript native private fields (`#` fields / `#` methods) are not used unless a human specifically instructs it.**
  That rule, and why it holds, belong to `/hoc-classes-principles`. It is read here for what it
  means when a property is named.
- Therefore, even if an instruction says `#alpha`, that alone is not a reason to implement it as a `#` field. It is normally defined as `this.alpha`.
