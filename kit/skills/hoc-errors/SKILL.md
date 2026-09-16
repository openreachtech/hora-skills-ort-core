---
name: hoc-errors
description: "Error-handling conventions. Covers returning null on failure from value-generating methods, and the two throws an abstract member declares itself unimplemented with — a plain Error carrying the fixed wording, or the error class the module declares for its own failures — along with the member notation and the run-time class name both of them carry."
---

# Shared: Errors

Summarizes conventions for error handling.

## Generation methods return null on failure

- For a method that returns a generated value, return `null` rather than throwing an exception "when generation is not possible."
- However, if there is a specific instruction to do so, implement it to throw an exception.

```javascript
// OK: return null when generation is not possible
/**
 * @param {{
 *   length: number
 * }} params - Parameters.
 * @returns {string | null} Generated text, or null if it cannot be generated.
 */
generate ({
  length,
}) {
  if (!Number.isInteger(length) || length < 0) {
    return null
  }

  // ...
}
```

## How an abstract member declares itself unimplemented

An abstract method or getter that requires an override in a subclass throws where it is
reached without one. **Two throws express that, and a module takes one of them throughout.**

| Route | What is thrown | Where it belongs |
| :-- | :-- | :-- |
| The plain error | `new Error()` carrying the wording below | The default, and the whole of it for a module that declares no error class of its own |
| The module's own error | The class that module declares for its failures | A module whose errors are part of what it publishes |

**The choice belongs to the module and is made once.** A module mixing the two leaves a caller
unable to say what to catch: half of its abstract members arrive as something the caller can
identify and half as a bare `Error`, and nothing about a member says which it will be.

Two rules hold whichever route is taken — how the member is named, and that the class name is
resolved rather than written down. Both are below the two routes.

### The plain error

- Unify the `new Error()` message to the following format.

```
`${<class name>}<member-notation> must be inherited`
```

- Unify the wording as `must be inherited`, with no trailing period (`.`).

```javascript
// OK: instance method (override required in subclass)
normalizeValue () {
  throw new Error(`${this.constructor.name}#normalizeValue() must be inherited`)
}

// OK: static getter
static get rawSchema () {
  throw new Error(`${this.name}.get:rawSchema must be inherited`)
}

// OK: static method
static generateCredential () {
  throw new Error(`${this.name}.generateCredential() must be inherited`)
}
```

### The module's own error

**A module that declares an error class so its exceptions can be told apart from anything
else's has already said how its failures arrive.** An abstract member left unimplemented is
that module's own contract failing, so it arrives the same way — and a bare `Error` thrown
beside it would be the one failure of that module a caller cannot place.

- **The message is the error class's, not this convention's.** Where the class builds one from
  an error code and a value, the code and the value are what a reader sees; the
  `must be inherited` wording belongs to the plain route and is not restated here.
- **The member goes in under the key `memberName`.** Fixing the key buys what fixing the wording
  buys on the other route — whoever searches for where a member is declared abstract has one
  string to search for, rather than a shape that differs per module.

```javascript
// OK: the module's own error, with the member named and the class resolved
static get config () {
  throw ConcreteMemberNotFoundError.create({
    value: {
      memberName: `${this.name}.get:config`,
    },
  })
}
```

- **Declaring the class is the module's decision, not this convention's.** What settles it is
  whether callers are meant to catch that module's failures by type; a module nobody catches
  that way gains nothing from the class and takes the plain route.

### The member is named as the documentation convention names it

- `<member-notation>` follows "Notation of Class Members" from the documentation convention
  (instance method `#instanceMethod()` / static getter `.get:staticGetter` / static method
  `.staticMethod()`, etc.).
- This holds in the plain error's message and in the `memberName` the module's own error
  carries. The notation is what a reader matches against the source, so it cannot differ by
  route.

### The class name is resolved at run time

- `<class name>` is resolved dynamically, embedding the actual runtime class (subclass) name.
  - Instance member: `this.constructor.name`
  - Static member: `this.name` (in a static context, `this` is the class itself)
- Do not hard-code the class name. Hard-coding it will display the wrong class name when
  inherited by a subclass — which is the one case the throw exists to report, since a base
  class reporting its own name says nothing about which subclass failed to override.
- **The module's own error hides the mistake more easily**, because the name sits inside a value
  object rather than in a template literal next to the member. Resolve it there too.

```javascript
// NG: class name is hard-coded (still displays "CompositeScalar" even in a subclass) + wording and trailing period are not unified
static get boundSchema () {
  throw new Error('CompositeScalar.get:boundSchema must be overridden.')
}

// OK: dynamically resolved, wording unified, no trailing period
static get boundSchema () {
  throw new Error(`${this.name}.get:boundSchema must be inherited`)
}
```

```javascript
// NG: the module's own error, with the class name written down
static get config () {
  throw ConcreteMemberNotFoundError.create({
    value: {
      memberName: 'BaseServerEngine.get:config',
    },
  })
}
```
