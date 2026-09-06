---
name: hoc-jsdoc
description: "JSDoc writing conventions shared by backend and frontend. Use when writing or reviewing JSDoc type annotations, in plain JavaScript or in Vue."
---

# Shared: JSDoc

Summarizes JSDoc writing conventions. All typing is JSDoc — no TypeScript syntax, no `.ts`
files — so always annotate types with JSDoc.

The rules in this body apply to every JavaScript file, backend and frontend alike. Conventions
that only apply when writing Vue / Nuxt (block placement, Furo class and factory typing, Vue
`PropType`, ambient globals) are in the reference files listed at the end, each marked with
its scope.

> How the jsdoc-plugin rules actually in effect under this project's ESLint map to
> each of this skill's conventions is summarized in
> [eslint-jsdoc-rules.md](./references/eslint-jsdoc-rules.md). If you write JSDoc as
> this skill prescribes, `npm run lint` produces no jsdoc-related errors. Note that
> lint also enforces such points as: `function` declarations and methods must have
> JSDoc (empty constructors are not exempt), each `@param` requires a type and a name,
> and a function that returns a value needs `@returns` (with a type). This skill goes
> one step further and requires `@returns` even when nothing is returned (`{void}`).

## Write object types with named parameters

- Do not use `{object}`.
- Write objects as an inline type literal, explicitly listing each property as a named parameter.

Example:

```javascript
// NG
/**
 * @param {object} params - Parameters.
 * @param {string} params.alpha - Alpha.
 */

// OK
/**
 * @param {{
 *   alpha: string
 * }} params - Parameters.
 */
```

## Name a single object argument `params`

- For a method that takes a single object argument, name the parameter `params` unless there is a special reason not to.

```javascript
// OK
/**
 * @param {{
 *   alpha: string
 * }} params - Parameters.
 */
```

## Always write `@returns`

- Every function and method gets `@returns`, even one that returns nothing.
- For a function that returns nothing, write `@returns {void}` (`@returns {Promise<void>}`
  when it is async). Never omit the tag.
- Lint (`jsdoc/require-returns`) only demands `@returns` when a value is actually returned,
  so this skill is stricter than lint here (see
  [eslint-jsdoc-rules.md](./references/eslint-jsdoc-rules.md)). The reason is that "returns
  nothing" should be stated, not inferred from the absence of a tag.

```javascript
// NG: returns nothing, so @returns is omitted
/**
 * @param {{
 *   message: string
 * }} params - Parameters.
 */

// OK
/**
 * @param {{
 *   message: string
 * }} params - Parameters.
 * @returns {void}
 */
```

## Attach a description to `@returns`

- Attach not just a type but also a description to `@returns`.
- The sole exception is `@returns {void}` / `@returns {Promise<void>}`. There is no
  returned value to describe, so the type stands on its own.
- Do not put a hyphen between the type and the description. The leading `- ` is a
  `@param` convention (`jsdoc/require-hyphen-before-param-description`) and does not
  apply to `@returns`.
- Lint (`jsdoc/require-returns-description`) is off and does not force a description,
  but this skill always attaches one for QA reasons (see
  [eslint-jsdoc-rules.md](./references/eslint-jsdoc-rules.md)).

```javascript
// NG: no description
/**
 * @returns {string}
 */

// NG: a hyphen before the description
/**
 * @returns {string} - Default characters to pick from.
 */

// OK
/**
 * @returns {string} Default characters to pick from.
 */
```

## Add `@public` to entry points

- Add `@public` to the JSDoc of any method that serves as an entry point accessed from outside.

```javascript
/**
 * Generate a random text of the given length.
 *
 * @param {{
 *   length: number
 * }} params - Parameters.
 * @returns {string | null} Generated random text, or null if it cannot be generated.
 * @public
 */
generate ({
  length,
}) {
  // ...
}
```

## Write array types as `Array<T>`

- Trailing-`[]` array notation such as `string[]` is prohibited. Always write it as `Array<string>`.
- Reason: writing something like `{ alpha: number, beta: string }[]` causes confusion for the reader.

```javascript
// NG
/**
 * @param {string[]} values - Values.
 */

// OK
/**
 * @param {Array<string>} values - Values.
 */
```

## Avoid vague types; write the most specific type you can

- Vague types such as `object` / `Object` / `any` / `*` are not used unless clearly necessary.
- Investigate the type and write it as a type literal with the most detailed properties possible,
  or as a concrete type name.
- Make generics concrete as far as possible. Do not leave the element type vague as
  `Array<object>`; write `Array<UserEntity>` instead.

```javascript
// NG: vague element type and return type
/**
 * @param {{
 *   users?: Array<object>
 * }} params - Parameters.
 * @returns {object} Users and their total count.
 */

// OK: investigate and write the detailed type
/**
 * @param {{
 *   users?: Array<UserEntity>
 * }} params - Parameters.
 * @returns {{
 *   users: Array<UserEntity>
 *   total: number
 * }} Users and their total count.
 */
```

### `object` / `Object` is prohibited; use `Record<string, *>`

- `object` / `Object` is prohibited regardless of the reason.
- Reason: `object` is dangerous in that it tolerates ambiguity including `null`. Even when you are
  forced to represent an object with unknown keys, `Record<string, *>` is better.
- Still, `Record<string, *>` is a last resort too; write a type literal with explicit properties
  wherever possible.

```javascript
// NG
/**
 * @param {object} config - Config.
 */

// BETTER: when the keys are genuinely unknown
/**
 * @param {Record<string, *>} config - Config.
 */

// BEST: when you can list the properties
/**
 * @param {{
 *   host: string
 *   port: number
 * }} config - Config.
 */
```

## Do not write undefined type names (define custom types before referencing them)

- For type names written in JSDoc, anything other than built-in types (`string` / `number` /
  `boolean` / `Array` / `Object` / `*` / `null`, etc.) and TS utility types (`Record` / `Partial` /
  `Pick` / `Omit` / `ReturnType`, etc.) must be **defined before** it is referenced, via one of
  `@typedef` / `@class` / `@interface` / import.
- Do not write undefined type names. Reason: they are a common source of typos and missing imports.
- For example, to write `Array<UserEntity>` from the previous section, a `@typedef` (or the like) for
  `UserEntity` must exist in the same file (or its import source).

```javascript
// NG: UserEntity is not defined anywhere
/**
 * @param {{
 *   users?: Array<UserEntity>
 * }} params - Parameters.
 */

// OK: define it with @typedef before referencing it
/**
 * @typedef {{
 *   id: number
 *   name: string
 * }} UserEntity
 */

/**
 * @param {{
 *   users?: Array<UserEntity>
 * }} params - Parameters.
 */
```

> The undefined-type-name check (`jsdoc/no-undefined-types`) is **off** by default in
> `@openreachtech/eslint-config`, so this skill enforces it rather than relying on lint.

## Do not write `undefined` as a type (use `null`)

- Do not write `undefined` in a type, as in `@returns {string | undefined}` or `@param {undefined}`.
- Express "no value" with `null`, not `undefined` (e.g. `@returns {string | null}`).
- For a function that returns nothing, write `@returns {void}`, not `@returns {undefined}`
  (see "Always write `@returns`" above).
- Exception: when a third-party module or the like requires `undefined`, you may write `undefined` in
  `@returns` and so on. If there is a way to avoid `undefined`, prefer that.

```javascript
// NG
/**
 * @returns {string | undefined} Generated text.
 */

// OK
/**
 * @returns {string | null} Generated text.
 */
```

## Write the any type as `*`

- To express an arbitrary (any) type, use `*`. Do not use `any`.
- Still, `*` (any) itself is not overused, per "Avoid vague types; write the most specific type you
  can" above. Write `*` (not `any`) only when the type cannot be narrowed and any is unavoidable.

```javascript
// NG
/**
 * @param {any} value - Value.
 */

// OK
/**
 * @param {*} value - Value.
 */
```

## Do not place a delimiter after each chopped-down property

- When chopping down the `@param` for named arguments, do not place a semicolon or comma after each property.

```javascript
// OK
/**
 * @param {{
 *   alpha: number
 *   beta: number
 * }} params - Parameters.
 */
```

## Writing `@typedef`

- Write `@typedef` as a block comment of at least three lines. A single-line
  `@typedef` is also prohibited by lint (`jsdoc/multiline-blocks`).
- Define one `@typedef` per block.
- Separate `@typedef` block comments from one another with a blank line.

```javascript
// NG: written on a single line
/** @typedef {*} BooleanLike */

// NG: multiple @typedef in one block
/**
 * @typedef {*} BooleanLike
 * @typedef {{
 *   id: number
 *   name: string
 * }} UserEntity
 */

// OK: at least three lines, one per block, separated by a blank line
/**
 * @typedef {*} BooleanLike
 */

/**
 * @typedef {{
 *   id: number
 *   name: string
 * }} UserEntity
 */
```

## Type-only imports

- To reference a type declared in another module, use a type-only import. Never use the
  TypeScript `import type` statement — it is not available in a JavaScript-only project.
- Two styles exist: the `@import` block tag and the inline `import('…')` expression. **Follow
  the style the repository has established, and do not mix both for the same type.** Furo /
  Nuxt apps use `@import`; renchan backends use the inline `import('…')` expression. If a
  repository has established neither, prefer `@import`.
- One `@import` tag per JSDoc block, one source module per block — the same rule as `@typedef`
  above.

```javascript
// OK: the @import block tag
/**
 * @import {
 *   Reactive,
 * } from 'vue'
 */

// OK: the inline import('…') expression
/**
 * @returns {import('./CustomerOrdersBk.js').default} Backup model declaration.
 */
```

See [import-tag.md](./references/import-tag.md) and
[import-expression.md](./references/import-expression.md) for each style in full.

## References

| Reference | Scope | Topic |
| --- | --- | --- |
| [eslint-jsdoc-rules.md](./references/eslint-jsdoc-rules.md) | Common | ESLint (jsdoc plugin) mapping — follow it and `npm run lint` passes / where this skill is stricter than lint / intentionally relaxed rules |
| [import-tag.md](./references/import-tag.md) | Common | Type-only import via the `@import` block tag — placement, named/default forms, source modules, consuming by bare name |
| [import-expression.md](./references/import-expression.md) | Common | Type-only import via the inline `import('…')` expression — forms, use sites, trade-offs vs `@import` |
| [placement.md](./references/placement.md) | Frontend | Where `@typedef` / `@import` blocks go, inline `@type` on reactive declarations, Params / FactoryParams naming |
| [class-typing.md](./references/class-typing.md) | Frontend | Params / FactoryParams typedef pair, `create()` factory template idiom, `@template` / `@extends` / `@override` / `@property` |
| [vue-props-and-globals.md](./references/vue-props-and-globals.md) | Frontend | Vue `PropType` on prop definitions, ambient globals used unqualified |
