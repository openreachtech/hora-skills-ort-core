---
name: hoc-classes-principles
description: "Principles of class design — what a class has to hold to exist at all, and the premises the other class conventions rest on. Use when deciding whether something should be a class, when reaching for a class field or a plain object, or when the reason behind a class convention is in question. Native features the system does not use belong to the native-feature prohibition convention; specific prohibitions, to the class prohibitions convention; member order, to the class notation convention."
---

# Classes: Principles

This summarizes the principles for class property declaration and design.

## Core principle: Do not create classes without properties

- A class must hold at least one instance property.
- What the prohibition covers — static-only classes and classes without properties — and its
  exceptions and reasons belong to `/hoc-classes-prohibits`. This skill takes it as the core
  principle and builds the surrounding design conventions on top of it.

## The five points of the system (a bundle of premises)

1. **Deep immutability** — Do not reassign after construction; keep nested values (e.g. instances of other immutable classes) immutable too. **The collection types Array / Set are not deep-frozen; updates (adding/removing elements) are allowed.** The reasons are that "the set of retained items" is premised on being treated as equal regardless of count, and that it also serves to support the Builder Pattern. **However, a structure that references an individual element (e.g. pulling out a single element via `array[i]`) is prohibited** — because it is a circumvention of the prohibition on mutable objects. A collection's value must always be "used all at once" (scanned/transformed/aggregated over every element as a whole). When you need to associate by object, use `WeakMap` (if you need to enumerate, hold the keys in an Array and traverse through them). Do not use `Map` (see the property-definition convention).
2. **Property = a constructor argument of the same name** — Every property that stores a value is passed as a
   constructor argument of the same name. This makes the set of property names a class occupies appear in the
   constructor signature itself, so hidden fields cannot exist in principle.
3. **Constructor-only** — Only `this.xxx = xxx` inside the `constructor` counts as a property (see "Definition of 'property'" below).
4. **Treat references as the contract** — The published API references fix the public surface. Direct access to a member not in the references is "undocumented = outside the contract," and breaking after coupling to it is the caller's responsibility. This is the stance that **"reaching for a member outside the contract is the same as throwing a microwave at a beast"** — a use outside the contract is not the designer's responsibility.
5. **Enumerating instances is prohibited** — Enumerating a class instance that has behavior and invariants via `Object.keys` / `Object.entries` / `for...in` / spread `{...instance}` / `Object.assign` is treated as hack code. It is a category error that treats a behavior-bearing object as a POJO/dictionary, and a clearly bad design practice that drops the prototype (methods) to make a degraded copy.

### Definition of "property"

- Only `this.xxx = xxx` inside the `constructor` counts as a property.
- Do not use class fields (`x = 1`) or private fields (`#x = 1`).

Reasons:

- **Single manifest** — Anyone who wants to know "what does this class hold" reads only one place, the constructor
  (and its argument signature). If class fields are allowed, then even adding a member-ordering rule of "fields go
  first" still leaves two places to read: the leading field group and the computed `this.x =`. The real ground for
  going as far as prohibition is not "scattered and hard to find" but that it **guarantees "strictly one location"**
  (if it were merely "hard to find," an ordering rule would suffice, and someone would point that out).
- **Linear, clear initialization order** — Class fields have the semantics of being initialized "right after `super()`, before the constructor body," which breeds bugs where `super()` and field initialization interleave under inheritance. Constructor-only avoids this structurally.
- **Eliminates split-brain** — Using class fields together with the constructor falls into a double declaration of "default in a field / overridden in the constructor from an argument." The following is a property declared twice, with the field initializer killed by the constructor override — a clearly bad design practice.

```javascript
// NG: mixing default + override (split-brain)
class Foo {
  x = 1              // default declaration
  constructor (x) {
    if (x !== undefined) {
      this.x = x     // override; the field initializer dies
    }
  }
}
```

### Handling of `static` fields

- **`static` fields (`static X = ...`) may be used.** The prohibition on class fields and private fields is a convention **about instance properties**, and `static` fields are out of its scope.
- However, **`static #X` (the static form of native private) is not used** — see
  `/hoc-prohibit-native-features`.

This is because none of the three grounds for the prohibition apply to `static`.

- **Single manifest** — What the manifest answers is "what does an **instance** of this class hold." A `static` field is not held by any instance, so it does not damage the at-a-glance completeness of the constructor signature. And since no declaration site exists outside the class body, "strictly one location" is already satisfied the moment it is written as `static`.
- **Initialization order** — `static` fields are initialized in source order at class-definition time, so no problem arises from the interplay of `super()` and the constructor body.
- **Split-brain** — There is no constructor to override in, so no double declaration occurs.

Why `static #X` is not used either belongs to `/hoc-prohibit-native-features`.

#### What to put in a `static` field

- **Put accumulating associations, pools, and caches in `static` + `WeakMap`, not in an instance.** Placed on an instance, they participate in equality comparison and serialization as part of the value — but a cache is not a value. With `static` + `WeakMap`, (1) it sits outside the value semantics of instances, (2) it is non-enumerable and key-gated, so a party without the key cannot reach it, and (3) the author can look inside on demand with `util.inspect(Ctor, { showHidden: true })`.
- **Do not reassign the reference itself.** Deep immutability extends to `static` as well. Only the inside of a collection may change, and that is constrained by the property-definition convention (`Map` is not used, even for `static`).
- **Adding `static` fields does not relax the core principle of not creating classes without properties.** A `static` field is not an instance property, so a class holding only those remains prohibited as a static-only class.

## What not to use

The native features this system does not use — `Map`, `Object.freeze()`, native private members
and decorators — belong to `/hoc-prohibit-native-features`, with the reason for each. What is
left here is the plain object, which is a matter of design rather than of a language feature.

### Plain objects

A plain object is avoided wherever it can be. What holds values is a class, so that the values
arrive with a name and with the members that work on them. The rule that draws the line — an
object literal is not shared across scopes on the strength of its shape — belongs to
`/hoc-prohibit-native-features`; what follows is why.

- **What decides it is that a class can be overridden in part.** A subclass replaces one member
  and keeps every other one as it was, which is how a hotfix, a variant or a test stand-in reaches
  the one place it needs. A plain object offers nothing to override, and neither does a module of
  exported functions — which is why a module is never used in place of a class either.

- **A plain object narrows the design.** It cannot be asked, so whoever needs what is behind it
  reaches through it or takes it apart, and the knowledge of its insides spreads to every place
  that uses it. A class grows the member that answers — `it.hasActiveAccount()` in place of
  `it.account.isActive()` — and the design keeps that room.

- **A type declaration does not make a plain object a value object.** `@typedef` writes down a
  shape, and the name it gives lives in the annotation alone: at run time the value is the same
  nameless object, with nothing on it to work on what it holds. Calling a plain object something
  else through its type is writing HTML in nothing but `<div>` and `<span>` — a class attribute
  does not make a `<div>` an `<article>`.
- **This is why typing stays in JSDoc and never moves to TypeScript.** A structural type system
  accepts any object of the right shape, so declaring the shape is enough to pass the check, and
  the value object is never written. What it would have held — completing an entry, comparing two
  of them — goes to whoever uses the object, and is written again in each place.

- **Data that arrives from outside is wrapped where it arrives.** The JSON an API returns is never
  parsed and spread through the application as it is. An API is reached through the Payload /
  Capsule / Launcher structure, and its response is always capsulized: what the application holds
  is the capsule, never the parsed object. How the three are built belongs to each stack's API
  client convention.

```javascript
// NG: an entry held as a plain object
addEntry ({
  title,
}) {
  return this.Ctor.create({
    entries: [
      ...this.entries,
      {
        title,
        isDone: false,
      },
    ],
  })
}

// OK: an entry is an instance of its own class
addEntry ({
  title,
}) {
  return this.Ctor.create({
    entries: [
      ...this.entries,
      this.Ctor.TodoEntryCtor.create({
        title,
      }),
    ],
  })
}
```

## Rules

- A class holds at least one instance property (see `/hoc-classes-prohibits`)
- Properties are only `this.xxx = xxx` inside the `constructor`; class fields and private fields are not allowed
- `static` fields are allowed (`static #X` is not); put accumulating associations, pools, and caches in `static` + `WeakMap`; do not reassign the reference
- Every property that stores a value is received via a constructor argument of the same name; do not reassign it (deep
  immutability). Array / Set may be updated but referencing an individual element is prohibited (always use the whole
  at once); use `WeakMap` for association, do not use `Map`
- Do not directly access members not in the references; do not enumerate instances
- Do not use `Map`, `Object.freeze()`, native private members or decorators (see `/hoc-prohibit-native-features`)
- Do not share an object literal across scopes (see `/hoc-prohibit-native-features`); capsulize what an API returns

## Proviso

- The benefits above hold only once all "five points of the system" are in place. If any one of them (especially deep immutability) is broken while the others remain, the ground for each rule collapses. The rules' legitimacy is load-bearing on one another.
