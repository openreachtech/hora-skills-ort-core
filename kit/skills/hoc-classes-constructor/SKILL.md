---
name: hoc-classes-constructor
description: "Conventions for class constructors. Constructor parameters must not have default values."
---

# Classes: Constructor

Conventions related to class constructors.

## Do not assign default values to parameters

- Do not assign default values to constructor parameters.

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
