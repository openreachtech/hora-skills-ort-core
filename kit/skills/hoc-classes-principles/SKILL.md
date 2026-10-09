---
name: hoc-classes-principles
description: "Principles of class design — what a class has to hold to exist at all, and the premises the other class conventions rest on. Use when deciding whether something should be a class, when reaching for a `#` private member (a static one included), a decorator, a class field, `Object.freeze()` or a plain object, or when the reason behind a class convention is in question. Specific prohibitions belong to the class prohibitions convention; member order, to the class notation convention."
---

# Classes: Principles

This summarizes the principles for class property declaration and design.

## Core principle: Do not create classes without properties

- A class must hold at least one instance property.
- What the prohibition covers — static-only classes and classes without properties — and its
  exceptions and reasons belong to `/hoc-classes-prohibits`. This skill takes it as the core
  principle and builds the surrounding design conventions on top of it.

## The five points of the system (a bundle of premises)

1. **Deep immutability** — Do not reassign after construction; keep nested values (e.g. instances of other immutable classes) immutable too. **The collection types Array / Set are not deep-frozen; updates (adding/removing elements) are allowed.** The reasons are that "the set of retained items" is premised on being treated as equal regardless of count, and that it also serves to support the Builder Pattern. How a collection's value may be used, and why `Map` gives way to `WeakMap`, belong to `/hoc-properties`.
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

The reason not to use `static #X` is that the reason not to use native private (a subclass cannot read it, which blocks extension and substitution through inheritance — see `#private` below) holds for `static` as well, and its impact is broader: when a `static` method refers to `this.#X`, **a call through a derived class throws a TypeError.**

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

## What not to use

### `#private`

Under an immutable property design, **there is no work left that is unique to `#private`**. One might initially evaluate abandoning it as "the biggest cost," but within this system it is not even a cost.

- **It blocks extension through inheritance** — A native private member cannot be read even from an inheriting subclass. A patch that changes behavior temporarily through inheritance, such as a hotfix that overrides in a subclass, is then blocked by the parent class's private member.
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

### `Object.freeze()`

`Object.freeze()` is not used. Immutability is held by the conventions, so freezing guards a
hazard this system has already removed — the same reasoning that sets `#private` aside above.

- **A frozen instance cannot be stood in for.** `jest.spyOn()` replaces a method by adding a
  property of the same name to the instance, and a frozen object takes no new property: the spy
  fails with `TypeError: Cannot add property …, object is not extensible`. A subclass patching the
  instance is shut out the same way.
- **Freezing a collection contradicts how one is built.** A collection is added to while it is
  assembled (see `/hoc-properties`); a frozen one cannot be.

```javascript
// NG: the instance is frozen, so a test cannot spy on it
constructor ({
  value,
}) {
  this.value = value

  Object.freeze(this)
}
```

### Plain objects

A plain object is avoided wherever it can be. What holds values is a class, so that the values
arrive with a name and with the members that work on them.

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
  immutability). Array / Set may be updated; how their value is used, and `Map` against `WeakMap`, follow
  `/hoc-properties`
- Do not directly access members not in the references; do not enumerate instances
- Do not use `#private` or `decorator` (except when a human explicitly specifies it)
- Do not use `Object.freeze()`
- Avoid plain objects; capsulize what an API returns

## Proviso

- The benefits above hold only once all "five points of the system" are in place. If any one of them (especially deep immutability) is broken while the others remain, the ground for each rule collapses. The rules' legitimacy is load-bearing on one another.
