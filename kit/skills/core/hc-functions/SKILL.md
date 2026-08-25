---
name: hc-functions
description: "Conventions for functions. Function parameters follow method parameters: named arguments as a principle."
---

# Functions

Conventions related to defining functions.

## Named arguments as a principle

- Function parameters follow method parameters: use named arguments as a principle.
- That is, parameters are received as a single object with named arguments (destructuring), chopped down one property per line.
- For detailed policy and exceptions, follow the method-definition convention, "Receive arguments as a single named-argument object".

```javascript
// NG: positional arguments
function createColor (red, green, blue) {
  // ...
}

// OK: a single named-argument object, chopped down
function createColor ({
  red,
  green,
  blue,
}) {
  // ...
}
```
