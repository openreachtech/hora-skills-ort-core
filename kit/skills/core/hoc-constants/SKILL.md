---
name: hoc-constants
description: "Conventions for constants. Covers naming (uppercase SNAKE_CASE / singular for enum-like objects), chopping down, and the file organization and placement of object-type constants."
---

# Constants

Conventions related to defining constants.

## Naming

- Constant names must be written in all-uppercase SNAKE_CASE.

```javascript
// NG
const maxRetryCount = 3

// OK
const MAX_RETRY_COUNT = 3
```

## Naming of enum-like objects

- When defining constants as an enum-like object, use the singular form for the name.
- Following the naming convention of enums, do not use the plural form.

```javascript
// NG: plural
const COLORS = {
  RED: 'red',
  GREEN: 'green',
  BLUE: 'blue',
}

// OK: singular
const COLOR = {
  RED: 'red',
  GREEN: 'green',
  BLUE: 'blue',
}
```

## Chopping down

- In object-type constant definitions, chop down per property (one property per line).

```javascript
// NG: multiple properties on one line
const COLOR = { RED: 'red', GREEN: 'green', BLUE: 'blue' }

// OK: one property per line
const COLOR = {
  RED: 'red',
  GREEN: 'green',
  BLUE: 'blue',
}
```

## File organization for object-type constants

- When defining an object-type constant, keep one definition per file.
- The file name must match the definition name.

```
// OK: COLOR is defined alone in COLOR.js
COLOR.js
  → const COLOR = { ... }
```

## Location

- Unless otherwise specified, define constant files under `<project-root>/constants/`.
- Subfolders may be used as appropriate, depending on the kind of constant.

```
<project-root>/constants/COLOR.js
<project-root>/constants/http/STATUS_CODE.js
```
