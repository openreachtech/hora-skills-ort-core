---
name: hc-modules-exports
description: "Conventions for module exports. Don't define files that merely named-export a function; define a class per responsibility."
---

# Modules: Exports

Conventions related to module exports.

## Don't define files that merely named-export a function

- Don't define a file whose only purpose is to named-export a function.
- If each function has a single responsibility, defining a class one by one is the correct approach.

```javascript
// NG: a file that merely named-exports a function
export function normalizeColor ({
  value,
}) {
  // ...
}

// OK: define a class per responsibility
export default class ColorNormalizer {
  // ...
}
```
