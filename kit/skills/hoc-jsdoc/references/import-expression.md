# The `import()` Type Expression

One of the two type-only-import styles, established in renchan backends. The other style is
the [`@import`](import-tag.md) block tag.

## Purpose

`import('<module>').<Type>` is the inline import-type expression — a type-only import written **directly inside** a JSDoc annotation, with no separate import block. It is the alternative to the [`@import`](import-tag.md) block tag; both solve the same problem (referencing a type from another module in a JS-only Furo app).

**Follow the repository's established style.** Some Furo apps use `import()` inline everywhere; others use `@import` blocks. If neither is established, prefer `@import`. Do not mix both styles for the same type.

## Form

`import('<module>').<ExportedName>` in any type position — `@type`, `@typedef`, `@param`, `@returns`, `@extends`. Generics and unions nest as usual:

```js
import('vue').Ref<HTMLFormElement | null>
import('#app').NuxtError
import('~/stores/customer.js').CustomerStore
```

For a module's **default export**, use `.default`:

```js
import('~/components/units/AppDialog.vue').default
```

Always include the filename extension (`.js`/`.vue`) in the module path — never omit it (`import('@openreachtech/furo-nuxt/lib/contexts/BaseFuroContext.js')`, not `import('@openreachtech/furo-nuxt/lib/contexts/BaseFuroContext')`).

## Where it appears

### Reactive declarations

Above the `ref()` / `shallowRef()` / `reactive()` call:

```js
/** @type {import('vue').Ref<HTMLFormElement | null>} */
const formRef = ref(null)

/** @type {import('vue').ShallowRef<HTMLInputElement | null>} */
const inputShallowRef = shallowRef(null)
```

### `@typedef` aliases

An imported type can be aliased to a local name in one line, then used bare:

```js
/**
 * @typedef {import('@openreachtech/furo-nuxt/lib/contexts/BaseFuroContext.js').BaseFuroContextParams} ComponentContextParams
 */

/**
 * @typedef {ComponentContextParams} ComponentContextFactoryParams
 */
```

## Trade-offs vs `@import`

| | `import('…')` inline | `@import` block |
| --- | --- | --- |
| Location | inside each annotation | one block at end of file / `<script>` |
| Path | repeated at every use site | written once |
| Best when | a type is used once or twice | a type recurs, or many types share a module |

When the repository's convention is `@import`, promote a recurring inline `import('…')` to a named block rather than repeating the path.
