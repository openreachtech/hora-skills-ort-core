---
name: hc-classes-principles
description: "Principles of class design. Establishes the overarching principle of not creating classes without properties (no-properties class), the system that underpins it (deep immutability, constructor-only, references-as-contract, etc.), and the reasons for not using #private / decorator."
---

# Classes: Principles

This summarizes the principles for class property declaration and design.

## Core principle: Do not create classes without properties

- A class must hold at least one instance property.
- Do not create a class that is nothing but a collection of static methods / static fields (a static-only class). Design it as a state-holding instance class, or delegate to a single-responsibility class.
- The class prohibitions convention is the canonical source for the detailed reasons and exceptions of this prohibition (no-properties / static-only). This skill sets that as the core principle and then systematizes the surrounding design conventions on top of it.

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
- However, **do not use `static #X` (the static form of native private).**

This is because none of the three grounds for the prohibition apply to `static`.

- **Single manifest** — What the manifest answers is "what does an **instance** of this class hold." A `static` field is not held by any instance, so it does not damage the at-a-glance completeness of the constructor signature. And since no declaration site exists outside the class body, "strictly one location" is already satisfied the moment it is written as `static`.
- **Initialization order** — `static` fields are initialized in source order at class-definition time, so no problem arises from the interplay of `super()` and the constructor body.
- **Split-brain** — There is no constructor to override in, so no double declaration occurs.

The reason not to use `static #X` is that the reason not to use native private (a subclass cannot read it, which blocks extension and substitution through inheritance — see the property-definition convention) holds for `static` as well, and its impact is broader: when a `static` method refers to `this.#X`, **a call through a derived class throws a TypeError.**

```javascript
// NG: a static native private cannot be used from a derived class
class Base {
  static #pool = new WeakMap()

  static ensure (key) {
    return this.#pool.has(key)
  }
}
class Derived extends Base {}

Derived.ensure(key)
// TypeError: Cannot read private member #pool from an object whose class did not declare it
```

#### What to put in a `static` field

- **Put accumulating associations, pools, and caches in `static` + `WeakMap`, not in an instance.** Placed on an instance, they participate in equality comparison and serialization as part of the value — but a cache is not a value. With `static` + `WeakMap`, (1) it sits outside the value semantics of instances, (2) it is non-enumerable and key-gated, so a party without the key cannot reach it, and (3) the author can look inside on demand with `util.inspect(Ctor, { showHidden: true })`.
- **Do not reassign the reference itself.** Deep immutability extends to `static` as well. Only the inside of a collection may change, and that is constrained by the property-definition convention (`Map` is not used, even for `static`).
- **Adding `static` fields does not relax the core principle of not creating classes without properties.** A `static` field is not an instance property, so a class holding only those remains prohibited as a static-only class.

### Handling of derived classes

- A class that has `extends` is out of scope even if it only overrides static members. The responsibility belongs to the base class.

## What not to use

### `#private`

Under an immutable property design, **there is no work left that is unique to `#private`**. One might initially evaluate abandoning it as "the biggest cost," but within this system it is not even a cost.

- **The write axis that private protected evaporates** — Private answers "who may touch it" (access control); immutability answers "can it change at all" (mutation control). Private's historical main purpose is preventing "invariants being broken by external rewrites," but under deep immutability there is not a single write to protect, inside or out. Private is the strategy of "guarding the hazard (mutable state)," immutability is the strategy of "removing the hazard itself"; with no hazard, the guard is redundant.
- **The remaining read axis is not `#`'s job either** — The read axis splits in two. (1) Decoupling (hiding the
  representation to refactor) is carried by references = the contract. Coupling to a member not in the references is
  outside the contract, so the author's freedom to refactor is guaranteed by the contract without waiting for `#`. (2)
  Secrecy (hiding the value itself, e.g. keys/passwords) is independent of immutability, but JS's `#` is not a
  security boundary against memory dumps/debuggers, so it is not a guarantee `#` can reliably provide — that is the
  job of a separate layer (encryption, not retaining).
- **Debug visibility actually favors soft-private** — In Node, `#` fields appear neither in `console.log` nor in `util.inspect(obj, { showHidden: true })` (DevTools shows `#` specially, but that is a Node-specific handicap). Soft-private (`this._x`) shows up in logs with zero extra code. For an immutable value object, the `_secret` shown there is not "dirt you want to hide" but "the very state you want to see," so being visible is correct. If you want to shape it, you can curate with `[util.inspect.custom]()` (opt-in).
- **`#` does not save the incompetent** — A user who does not respect boundaries will, even with `#`, touch public things mutably and break something elsewhere ("a lock only keeps out the honest," "a caveman cannot use a microwave"). In closed / application code, stripping visibility and straightforward description from competent users for the sake of the thin band that accidentally couples is putting the cart before the horse.
- **Name collisions also vanish via the system** — `#`'s last redeeming value, "safety against name collisions under inheritance," vanishes for your own hierarchy via "Property = a constructor argument of the same name." Since everything that holds a value appears in the arguments = the public signature, no hidden field exists, and the inheriting side necessarily confronts it via `super({...})`. That safety is a value for "the author of the class being inherited (the base)," not something the inheriting side gains by writing `#` in its own code.

Therefore soft-private (`this._x`) is visible and correct. Do not use `#private` unless a human explicitly specifies it.

### `decorator`

- A `decorator` adds no capability at all. It is purely syntactic sugar over a higher-order function + metadata (`@memoize method(){}` ≡ `method = memoize(method)`); everything a decorator can do can be written explicitly.
- This policy chooses "explicitness > brevity." A decorator's real benefits (co-locating declarative metadata, reducing DI boilerplate) are ergonomics, not capability, and they sell explicitness in return. Augmentation that is implicit, mutating, and action-at-a-distance conflicts with "explicit, immutable, single manifest."
- A field decorator attaches to a class field, so it does not fit at the syntactic level under this policy, which prohibits class fields.

## Rules

- A class holds at least one instance property (if it cannot, replace it with a method of a state-holding class / delegation to a single-responsibility class)
- Properties are only `this.xxx = xxx` inside the `constructor`; class fields and private fields are not allowed
- `static` fields are allowed (`static #X` is not); put accumulating associations, pools, and caches in `static` + `WeakMap`; do not reassign the reference
- Every property that stores a value is received via a constructor argument of the same name; do not reassign it (deep
  immutability). Array / Set may be updated but referencing an individual element is prohibited (always use the whole
  at once); use `WeakMap` for association, do not use `Map`
- Do not directly access members not in the references; do not enumerate instances
- Do not use `#private` or `decorator` (except when a human explicitly specifies it)
- A class that has `extends` is out of scope

## Proviso

- The benefits above hold only once all "five points of the system" are in place. If any one of them (especially deep immutability) is broken while the others remain, the ground for each rule collapses. The rules' legitimacy is load-bearing on one another.
