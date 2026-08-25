# Vue Props & Ambient Globals

Frontend (Vue / Nuxt) only. The function and method annotation rules these examples follow
(single-object `@param`, always-present `@returns` with a description) are in the skill body.

## Vue prop types

Inline `@type {PropType<...>}` on the prop's `type` field:

```js
props: {
  error: {
    /** @type {PropType<NuxtError>} */
    type: Object,
    required: true,
  },
}
```

`String` is a special case, we must use type cast on the String object:

```js
props: {
  type: {
    type: /** @type {PropType<MessageType>} */ (String),
    required: true,
  },
}
```

See [[hf-nuxt]].

## Ambient global helpers (no import needed)

`RequiredExcept`, `OptionalExcept`, `NullableExcept` are declared in `types/global.d.ts` under `declare global`, so they're used **unqualified** in JSDoc with no `@import`. Same for `schema.graphql.*` ([[hf-graphql]]), `furo.*`, and `GraphqlType.*` ([[hf-nuxt]]).
