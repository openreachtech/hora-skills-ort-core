# The `import()` Type Expression

One of the two type-only-import styles, established in renchan backends. The other style is
the [`@import`](import-tag.md) block tag.

## Purpose

`import('<module>').<Type>` is the inline import-type expression — a type-only import written **directly inside** a JSDoc annotation, with no separate import block. It is the alternative to the [`@import`](import-tag.md) block tag; both solve the same problem (referencing a type from another module in a JavaScript-only project).

**Follow the repository's established style.** If neither is established, prefer `@import`. Do not mix both styles for the same type.

## Form

`import('<module>').<ExportedName>` in any type position — `@type`, `@typedef`, `@param`, `@returns`, `@extends`. Generics and unions nest as usual:

```js
import('node:http').IncomingMessage
Array<import('node:http').IncomingMessage>
import('node:http').ServerResponse | null
```

For a module's **default export**, use `.default`:

```js
import('./CustomerOrdersBk.js').default
```

Always include the filename extension in the module path — never omit it (`import('./CustomerOrdersBk.js')`, not `import('./CustomerOrdersBk')`).

## Where it appears

### `@typedef` aliases

An imported type can be aliased to a local name in one line, then used bare:

```js
/**
 * @typedef {import('node:http').IncomingMessage} IncomingRequest
 */
```

## Trade-offs vs `@import`

| | `import('…')` inline | `@import` block |
| --- | --- | --- |
| Location | inside each annotation | one block at end of file |
| Path | repeated at every use site | written once |
| Best when | a type is used once or twice | a type recurs, or many types share a module |

When the repository's convention is `@import`, promote a recurring inline `import('…')` to a named block rather than repeating the path.
