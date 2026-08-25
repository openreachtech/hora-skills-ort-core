---
name: hc-errors
description: "Error-handling conventions. Covers returning null on failure from value-generating methods, the throw-message format for abstract members, and related policies."
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

## Throw message format for abstract members

- When expressing, via `throw`, an abstract method or abstract getter that requires an override (inherited implementation) in a subclass, unify the `new Error()` message to the following format.

```
`${<class name>}<member-notation> must be inherited`
```

- `<member-notation>` follows "Notation of Class Members" from the documentation convention (instance method `#method()` / static getter `.get:name` / static method `.method()`, etc.).
- `<class name>` is resolved dynamically, embedding the actual runtime class (subclass) name.
  - Instance member: `this.constructor.name`
  - Static member: `this.name` (in a static context, `this` is the class itself)
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

- Do not hard-code the class name. Hard-coding it will display the wrong class name when inherited by a subclass.

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
