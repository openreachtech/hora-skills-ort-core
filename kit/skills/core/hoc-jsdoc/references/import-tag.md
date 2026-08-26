# The `@import` Tag

One of the two type-only-import styles. The established style in Furo / Nuxt apps, so the
examples below are frontend ones; the tag itself is not frontend-specific. The other style is
the inline [`import('…')`](import-expression.md) expression, established in renchan backends.

## Purpose

`@import` is the JSDoc block tag for **type-only imports**. In a JavaScript-only Furo app there is no TypeScript syntax available, so `import type { … }` is not an option — `@import` is how a type declared in one module is made available for JSDoc annotations in another.

Never use inline `import type`. The other type-only-import style is the inline `import('…')` expression ([import-expression](import-expression.md)); `@import` is the preferred one — import each type once at the bottom of the file and reference it by bare name. Don't mix the two styles for the same type.

## Placement

- `@import` blocks live at the **end of the file**, or at the **end of the `<script>` block** in a `.vue` file — after the code, alongside the `@typedef` blocks.
- **One `@import` per JSDoc block, one source module per block.** Do not combine two `from '…'` sources in a single block.
- Separate consecutive blocks with a blank line.

```js
/**
 * @import {
 *   Reactive,
 * } from 'vue'
 */

/**
 * @import {
 *   useRouter,
 * } from 'vue-router'
 */

/**
 * @import {
 *   BaseFuroContextParams,
 * } from '@openreachtech/furo-nuxt/lib/contexts/BaseFuroContext.js'
 */
```

## Forms

### Named import (braced, multi-line)

The default form for named exports. Braces on their own lines, one symbol per line, **trailing comma** after every symbol — even a single one:

```js
/**
 * @import {
 *   Reactive,
 *   ShallowRef,
 * } from 'vue'
 */
```

### Default import (one-line shorthand)

For a module whose default export is the type you need (composables, GraphQL `*Payload` / `*Capsule` classes, module classes), the single-line form is idiomatic:

```js
/**
 * @import useAppGraphqlClient from '~/composables/useAppGraphqlClient.js'
 */

/**
 * @import SignOutMutationGraphqlCapsule from '~/app/graphql/client/mutations/signOut/SignOutMutationGraphqlCapsule.js'
 */
```

### Default import via `default as` (braced)

The braced equivalent of the shorthand, aliasing a module's default export to a local name. Reserve this form for single-file components (`.vue`, whose only export is the default):

```js
/**
 * @import {
 *   default as AppDialog,
 * } from '~/components/units/AppDialog.vue'
 */
```

Do **not** use the braced `default as` form for a class default export (`.js` composables, GraphQL `*Payload` / `*Capsule` classes, module classes) — use the one-line shorthand above instead:

```js
// Do not
/**
 * @import {
 *   default as UpdateEmailMutationGraphqlCapsule,
 * } from '~/app/graphql/client/mutations/updateEmail/UpdateEmailMutationGraphqlCapsule.js'
 */

// Do
/**
 * @import UpdateEmailMutationGraphqlCapsule from '~/app/graphql/client/mutations/updateEmail/UpdateEmailMutationGraphqlCapsule.js'
 */
```

## Common source modules

| Source | For |
| --- | --- |
| `'vue'` | `Reactive`, `Ref`, `ShallowRef`, `PropType`, `ComponentCustomProps`, … |
| `'vue-router'` | `useRoute`, `useRouter` (consumed via `ReturnType<typeof …>`) |
| `'#app'` | Nuxt types such as `NuxtError` |
| `'@openreachtech/furo-nuxt'` / `'@openreachtech/furo-nuxt/lib/contexts/BaseFuroContext.js'` | `BaseFuroContextParams`, furo base types |
| `'~/composables/*.js'` | `use*` / `useApp*` composable defaults |
| `'~/stores/*.js'` | `use*Store` return types (e.g. `CustomerStore`) |
| `'~/app/graphql/client/**/*.js'` | GraphQL `*Payload` / `*Capsule` classes |
| `'~/components/**/*.vue'` | component classes and their local typedefs |
| `'./Component.vue'` / `'./index.vue'` | sibling local typedefs |

Use the `~/` alias for repo-root paths and `./` for siblings — the same aliases Nuxt resolves at runtime.

## Consuming imported types

Once imported, a type is referenced **by its bare name** inside `@typedef`, `@type`, `@param`, `@extends`, etc. — no path qualifier:

```js
/** @type {Reactive<ErrorMessageHash>} */

/**
 * @typedef {BaseFuroContextParams & {
 *   router: ReturnType<typeof useRouter>
 *   customerStore: CustomerStore
 *   dialogComponentShallowRef: ShallowRef<AppDialog | null>
 * }} SignOutSubmitterContextParams
 */
```

Sibling local typedefs (`ErrorMessageHash`, `ComponentProps`, `FormField`, …) are declared at the bottom of one `<script>`/file and re-imported across sibling files via `@import { ErrorMessageHash } from './Component.vue'`.

## Relationship to the inline `import('…')` style

The inline `import('…')` expression ([import-expression](import-expression.md)) is the alternative type-only-import style. In a repo standardized on `@import`, keep type imports in bottom-of-file blocks; if an inline `import('…')` creeps in for a one-off (typically a single Vue ref), prefer promoting it to a named `@import` block so the file stays in one style:

```js
// prefer this, with `Ref` from an `@import { Ref } from 'vue'` block…
/** @type {Ref<ReturnType<typeof setTimeout> | null>} */
const timeoutIdRef = ref(null)

// …over the inline form in an `@import`-style file
/** @type {import('vue').Ref<ReturnType<typeof setTimeout> | null>} */
const timeoutIdRef = ref(null)
```

## Do not `@import` ambient globals

Types declared under `declare global` in `types/*.d.ts` are used **unqualified, with no `@import`**: `RequiredExcept`, `OptionalExcept`, `NullableExcept`, and the `schema.graphql.*`, `furo.*`, and `GraphqlType.*` namespaces. Importing them is redundant. See the parent `hoc-jsdoc` skill and [[hof-nuxt]].
