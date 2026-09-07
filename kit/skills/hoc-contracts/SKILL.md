---
name: hoc-contracts
description: "Conventions for the type contracts of function and method arguments and return values, including how contract types are defined."
---

# Shared: Contracts

Conventions related to argument and return-value contracts. Applies across both functions and methods.

## Use BooleanLike for boolean return values

- When returning a boolean, use the `BooleanLike` type.
- To do so, define the `BooleanLike` `@typedef` at the bottom of the file.
- If the type is already defined as standard, `@typedef` is unnecessary.
- Follow the JSDoc convention ("Writing `@typedef`") for the `@typedef` format.

```javascript
/**
 * Check whether the value is valid.
 *
 * @param {{
 *   value: *
 * }} params - Parameters.
 * @returns {BooleanLike} Whether the value is valid.
 */
function isValid ({
  value,
}) {
  return value != null
}

// Defined at the bottom of the file
/**
 * @typedef {*} BooleanLike
 */
```
