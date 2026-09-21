---
name: hoc-classes-constructor
description: "Conventions for class constructors. Constructor parameters must not have default values."
---

# Classes: Constructor

Conventions related to class constructors.

## Do not assign default values to parameters

- Do not assign default values to constructor parameters.
- **The constructor's whole responsibility is to hold what its parameters receive.** Each one is
  assigned to the property of the same name, and no value is decided here.
- **Deciding a value the caller did not supply belongs to the factory methods**, `static create
  (...)` first among them. That division is settled by the method-definition convention.

```javascript
// NG: assigning a default value to a parameter
constructor ({
  delimiter = ',',
}) {
  this.delimiter = delimiter
}

// OK: no default value
constructor ({
  delimiter,
}) {
  this.delimiter = delimiter
}
```

## Separate what goes up to the base from what this class keeps

**Where a constructor hands part of its parameters to the base class and keeps the rest,
a blank line divides the two in the parameter list.** The reader then sees, without
following `super()`, which of the properties this class is responsible for.

```javascript
constructor ({
  ErrorCtor,

  maxDocumentDepth,
}) {
  super({
    ErrorCtor,
  })

  this.maxDocumentDepth = maxDocumentDepth
}
```

- **The factory methods repeat the same division**, in their own parameter list and in the
  `new this({ ... })` they assemble. A class states the split in one shape wherever the
  list appears, so a reader meeting `.create()` first learns the same thing.
- **A call that hands over the whole list carries no division**: `super({ ErrorCtor })`
  lists only the base's share, and nothing is separated inside it.
- **The JSDoc above it does not repeat the division.** A JSDoc block has no blank line to
  divide with, and the `*`-only line that stands in for one is spent between the description
  and the first tag — see the JSDoc convention. The type literal stays a flat list of the
  same properties.
- **Tests do not repeat it either.** A test assembling the constructor's argument is
  building a value, not declaring what the class is made of, so the argument object it
  writes carries no blank line.
