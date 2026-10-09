---
name: hoc-prohibit-native-features
description: "The native JavaScript features this library does not use, and why each is refused. Use as the criterion whenever code is about to reach for a native feature — `Map`, `Object.freeze()`, a `#` private member, a decorator, an object literal handed across scopes — or when reviewing code that does. A class field is refused by the class design principles, which define what a property is."
---

# Prohibit Native Features

The native features below are not used. Each one does a job this library's design already does
another way — immutability held by convention, a class that can be overridden in part — and
each costs something that way of working depends on.

- **A class field (`x = 1`) is refused too**, by the class design principles convention
  (`/hoc-classes-principles`): what it rules out is part of what a property is, so it stays
  with the definition of a property.

## Object literals shared across scopes

**An object literal is not shared across scopes on the strength of its shape.** Built and used
up where it stands, a literal is fine. Handed out — returned, kept in a property, passed on to
be read elsewhere — it becomes a value whose reader trusts only that it has the right keys, and
what holds values is a class. Why a shape is not a value belongs to the class design principles
convention (`/hoc-classes-principles`).

- **A named-argument object is not sharing.** `fn({ title })` hands a literal across a call, and
  the callee destructures it on arrival into separate parameters; nothing keeps it as a value.
- **A `@typedef` naming the shape does not change this.** It describes the literal, and the
  reader still holds a nameless object.

```javascript
// NG: a literal leaves the method, and its reader relies on the shape
computeTotalPerRegion () {
  return this.extractKeptSales()
    .reduce((totals, it) => ({
      ...totals,
      [it.region]: (totals[it.region] ?? 0) + it.amount,
    }), {})
}

// OK: what leaves the method is an instance of a class
computeRegionTotals () {
  const keptSales = this.extractKeptSales()

  return [
    ...new Set(
      keptSales.map(it => it.region)
    ),
  ]
    .map(region => this.Ctor.RegionTotalCtor.create({
      region,
      amount: keptSales
        .filter(sale => sale.region === region)
        .reduce((total, sale) => total + sale.amount, 0),
    }))
}
```

## `Map`

- **`Map` is prohibited (enforced by ESLint).** Do not use it unless there is a special reason in module development. In application code, there has been no case where `Map` was used other than as an evasion.
- `WeakMap`, on the other hand, may be used freely. Associating objects by identity is `WeakMap`'s responsibility, and enumeration is unnecessary (when you want to enumerate, hold the keys in an Array and traverse through them).
- Making a `Map`'s key a primitive value (`number` / `string`, etc.) is a circumvention of the reassignment prohibition and the prohibition on mutable objects. An object-keyed `Map` is also unnecessary: if enumeration is not needed, `WeakMap` suffices, and even if it is, an Array of keys + `WeakMap` covers it — so there is no reason to choose `Map`.
- If a mutable aggregate seems necessary, reconsider the design (assemble it via a higher-order function and return it, or generate a new instance via a factory method).

## Native private members (`#x`)

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

- **Not used unless a human specifically instructs it.** An instruction that writes a member as
  `#alpha` is member notation, not a request for a native `#` field — see the property-definition
  convention.

### `static #x`

The reason not to use `static #X` is that the reason not to use native private (a subclass cannot read it, which blocks extension and substitution through inheritance — see above) holds for `static` as well, and its impact is broader: when a `static` method refers to `this.#X`, **a call through a derived class throws a TypeError.**

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

## Decorators

- A `decorator` adds no capability at all. It is purely syntactic sugar over a higher-order function + metadata (`@memoize method(){}` ≡ `method = memoize(method)`); everything a decorator can do can be written explicitly.
- This policy chooses "explicitness > brevity." A decorator's real benefits (co-locating declarative metadata, reducing DI boilerplate) are ergonomics, not capability, and they sell explicitness in return. Augmentation that is implicit, mutating, and action-at-a-distance conflicts with "explicit, immutable, single manifest."
- A field decorator attaches to a class field, so it does not fit at the syntactic level under this policy, which prohibits class fields.

## `Object.freeze()`

`Object.freeze()` is not used. Immutability is held by the conventions, so freezing guards a
hazard this system has already removed — the same reasoning that sets native private aside above.

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
