---
name: hc-classes-notations
description: "Convention for the order members are written in a class body — the eight-block placement order (fields, constructor, factory methods, inflators, getters, methods) plus the ordering within getters and within methods, falling back to source order where undetermined. What a class may hold belongs to the class design principles convention; this covers only writing order. Refer to it when arranging or reviewing member order."
---

# Classes: Notations

Collects the conventions for how a class definition is written in source. Design-level judgments — what a class may hold, what it must not use — are governed by the class design principles convention; this skill establishes only the **way it is written**.

## Placement order of members

Write a class body in the following order.

1. `static` fields
2. `constructor`
3. The factory methods (`.create()` → `.createAsync()`)
4. static inflator methods
5. static getters
6. static methods
7. instance getters
8. instance methods

Item 3 means the class's own factory methods that are published as API. The convention of placing `.create()` immediately after the `constructor` is governed by the method-definition convention, and the definition and naming of the inflator methods in item 4 by the inflator-methods convention.

### Why `static` fields go at the top

- This follows the Java convention.
- **The order is arranged to mirror the order in which memory is secured.** `static` fields are secured and
  initialized at class-definition time, and an instance's properties are secured afterwards in the `constructor`.
  Matching the source order to that order makes the order of allocation readable by simply reading the source from top
  to bottom.
- The conditions for using `static` fields, and what belongs in them (do not use `static #X`; put accumulating associations, pools, and caches in `static` + `WeakMap`, etc.), are governed by the class design principles convention.

### Order among getters

Within the getter categories (5 and 7), use the following order.

1. Getters that return a fixed value. Among them, place the ones denoted by `~Ctor` (static getters for delegation) first.
2. The rest (getters that compute the value they return).
3. List abstract getters last, together.

The naming of `~Ctor` is governed by the accessor-definition convention.

### Order among methods

Within the method categories (6 and 8), the order is **the order of appearance in an outline that writes out the nesting structure of calls**. Immediately after a method, place the methods that it calls (depth-first, first appearance). Members belonging to another category (getters and the like), and anything that is not an own member of the class (methods of a delegate or of a dependency class), are left out of this enumeration.

**For instance methods, the call outline of the private methods takes priority.** Therefore abstract members are not collected and placed at the end; they go at their first-appearance position in the outline.

An outline example for a `Sample` class.

```
// --- Static Methods ---

- .create()
  - constructor

- .createAsync()
  - .buildAlpha()
  - .generateBeta()
  - .createAlphaClient()
  - .create()

- .buildAlpha()
  - .get:AlphaCtor

- .generateBeta()
  - .buildGamma()

- .buildGamma()
  - .get:delta

- .createAlphaClient()
  - AlphaClient.create()

// --- Instance Methods ---

- #buildAlpha()
  - #generateBeta()
  - .get:beta
  - #defineDelta()

- #generateBeta()
  - #normalizeBeta()
  - #defineGamma()
  - .get:gamma

- #defineGamma()
  - #alphaClient.saveDelta()

- #defineDelta()
  - #alphaClient.loadDelta()
```

The class that corresponds to the outline above. Member bodies are folded into `{ ... }` and arguments into `(...)`, written on a single line.

```javascript
export default class Sample extends BaseSample {
  // static field
  static pool = new WeakMap()

  constructor (...) { ... }

  // factory methods
  static create (...) { ... }
  static async createAsync (...) { ... }

  // static inflator method
  static as (...) { ... }

  // static getters
  static get AlphaCtor () { ... }
  static get beta () { ... }
  static get gamma () { ... }
  /** @abstract */
  static get delta () { throw new Error('...') }

  // static methods
  static buildAlpha (...) { ... }
  static generateBeta (...) { ... }
  static buildGamma (...) { ... }
  static createAlphaClient (...) { ... }

  // instance getters
  get Ctor () { ... }

  // instance methods
  buildAlpha (...) { ... }
  generateBeta (...) { ... }
  /** @abstract */
  normalizeBeta (...) { throw new Error('...') }
  defineGamma (...) { ... }
  defineDelta (...) { ... }
}
```

### What the order does not determine

For whatever this skill's order does not determine, use **the order in which they are written from the top within the class**. When enumerating or documenting members, follow that same order.
