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
