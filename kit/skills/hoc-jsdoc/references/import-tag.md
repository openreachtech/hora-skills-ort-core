# The `@import` Tag

One of the two type-only-import styles. The other style is the inline
[`import('…')`](import-expression.md) expression, established in renchan backends.

## Purpose

`@import` is the JSDoc block tag for **type-only imports**. In a JavaScript-only project there is no TypeScript syntax available, so `import type { … }` is not an option — `@import` is how a type declared in one module is made available for JSDoc annotations in another.

Never use inline `import type`. The other type-only-import style is the inline `import('…')` expression ([import-expression](import-expression.md)); `@import` is the preferred one — import each type once at the bottom of the file and reference it by bare name. Don't mix the two styles for the same type.

## Placement

- `@import` blocks live at the **end of the file** — after the code, alongside the `@typedef` blocks.
- **One `@import` per JSDoc block, one source module per block.** Do not combine two `from '…'` sources in a single block.
- Separate consecutive blocks with a blank line.

```js
/**
 * @import {
 *   IncomingMessage,
 * } from 'node:http'
 */

/**
 * @import CustomerOrdersBk from './CustomerOrdersBk.js'
 */
```

## Forms

### Named import (braced, multi-line)

The default form for named exports. Braces on their own lines, one symbol per line, **trailing comma** after every symbol — even a single one:

```js
/**
 * @import {
 *   IncomingMessage,
 *   ServerResponse,
 * } from 'node:http'
 */
```

### Default import (one-line shorthand)

For a module whose default export is the type you need — a class, most often — the single-line form is idiomatic:

```js
/**
 * @import CustomerOrdersBk from './CustomerOrdersBk.js'
 */
```

Do **not** use the braced `default as` form for a class default export — use the one-line shorthand above instead:

```js
// Do not
/**
 * @import {
 *   default as CustomerOrdersBk,
 * } from './CustomerOrdersBk.js'
 */

// Do
/**
 * @import CustomerOrdersBk from './CustomerOrdersBk.js'
 */
```

## Consuming imported types

Once imported, a type is referenced **by its bare name** inside `@typedef`, `@type`, `@param`, `@extends`, etc. — no path qualifier:

```js
/**
 * @typedef {{
 *   request: IncomingMessage
 *   customerOrders: CustomerOrdersBk
 * }} OrderRequestParams
 */
```

## Relationship to the inline `import('…')` style

The inline `import('…')` expression ([import-expression](import-expression.md)) is the alternative type-only-import style. In a repo standardized on `@import`, keep type imports in bottom-of-file blocks; if an inline `import('…')` creeps in for a one-off, prefer promoting it to a named `@import` block so the file stays in one style:

```js
// prefer this, with `IncomingMessage` from an `@import { IncomingMessage } from 'node:http'` block…
/**
 * @param {{
 *   request: IncomingMessage
 * }} params - Parameters.
 */

// …over the inline form in an `@import`-style file
/**
 * @param {{
 *   request: import('node:http').IncomingMessage
 * }} params - Parameters.
 */
```
