# ESLint (jsdoc plugin) mapping

This maps the jsdoc rules **actually in effect** under this project's ESLint
configuration (`eslint-plugin-jsdoc`, bundled by `@openreachtech/eslint-config`) to
how this skill writes JSDoc. **If you write JSDoc as this skill prescribes,
`npm run lint` produces no jsdoc-related errors.** Items that overlap with
conventions in other files reinforce one another.

> The settings are layered. The base is `@openreachtech/eslint-rules-default-jsdoc`
> (nearly every rule is `error`); on top of it, `configurations/plugins/jsdoc.js` in
> `@openreachtech/eslint-config` turns some rules `off` or swaps their options. For
> the actual values, see those two files.

## Must follow (rules that error)

### Presence and targets of blocks

- **`function` declarations and methods (`MethodDefinition`) must have JSDoc.** Empty
  constructors and empty functions are not exempt (`exemptEmptyConstructors: false` /
  `exemptEmptyFunctions: false`). Arrow functions, function expressions, and class
  declarations themselves are not targeted. — `jsdoc/require-jsdoc`
  (`contexts: ['FunctionDeclaration', 'MethodDefinition']`)
- **Do not leave empty JSDoc blocks or empty descriptions.** —
  `jsdoc/no-blank-blocks` / `jsdoc/no-blank-block-descriptions`
- **Single-line blocks (`/** ... *&#47;`) are prohibited**; always write them across
  multiple lines. The only exceptions are `lends` / `type`, plus `extends` /
  `inheritdoc` / `override` added by this project. → This is what lint-enforces this
  skill's "write `@typedef` on at least three lines." — `jsdoc/multiline-blocks`
  (`noSingleLineBlocks: true`)

### Types, `@param`, `@returns`

- **`@param` is required for every argument, with a type and a name.** Destructured
  (named) parameters are targeted too, and the documented name must match the actual
  argument name. — `jsdoc/require-param` / `jsdoc/require-param-name` /
  `jsdoc/require-param-type` / `jsdoc/check-param-names`
- **Using `object` as a `@param` type forces you to document its sub-properties**
  (`checkTypesPattern` matches `object` / `Object` / `Array`, etc.). → This skill's
  "write it as an inline type literal instead of `{object}`" aligns with this and
  avoids the extra reports. — `jsdoc/require-param` / `jsdoc/check-param-names`
- **If you write a `@param` description, prefix it with a hyphen `- `** (the
  description itself is not required; see "Intentionally relaxed" below). The rule
  targets `@param` only, so a `@returns` description takes **no** hyphen. —
  `jsdoc/require-hyphen-before-param-description` (`'always'`)
- **A function that returns a value needs `@returns`, with a type, consistent with the
  actual return.** A function that returns nothing is not targeted → **this skill is
  stricter and asks for `@returns {void}` there too** (see the table below). —
  `jsdoc/require-returns` / `jsdoc/require-returns-type` / `jsdoc/require-returns-check`
- **A function that throws needs `@throws`; a generator needs `@yields`.** —
  `jsdoc/require-throws` / `jsdoc/require-yields` / `jsdoc/require-yields-check`
- **Write `@public` / `@private` / `@protected` / `@access` correctly and without
  duplicates.** → This skill's "add `@public` to entry points" is validated under this
  rule. — `jsdoc/check-access`
- **Type syntax must not be broken (matched `{`–`}`, etc.).** — `jsdoc/check-syntax` /
  `jsdoc/check-property-names` (when using `@property`)

### Layout (rules, formatting, tag order)

- **Every line must start with `*`.** — `jsdoc/require-asterisk-prefix` (`'always'`)
- **Keep the `*` column aligned and the spacing after tags (one space after the tag /
  type / name / hyphen).** — `jsdoc/check-alignment` / `jsdoc/check-line-alignment`
- **Exactly one blank line between the description and the first tag; no blank lines
  between tags; no blank line before the closing.** — `jsdoc/tag-lines`
  (`'never'`, `startLines: 1`, `endLines: 0`, `applyToEndTag: true`)
- **Order tags by the default `tagSequence`** (roughly
  `@param` → `@returns` → `@throws` → … → `@public`/`@access` → `@example`), with no
  blank lines between tags (`linesBetween: 0`). — `jsdoc/sort-tags`
- **Do not place stray `*` (`**`) in the middle or at the end of a line** (leading
  whitespace is allowed). — `jsdoc/no-multi-asterisks` (`allowWhitespace: true`)
- **`@description` / `@param` / `@returns` and others are excluded from the indentation
  check**, but the body indentation of other tags is inspected. —
  `jsdoc/check-indentation`

## How each of this skill's conventions maps to the rules

| This skill's convention | How lint treats it |
|---|---|
| Object types as named parameters (no `{object}`) | `require-param` / `check-param-names` demand sub-property documentation for an `object` type, so writing an inline type literal aligns with lint (reinforced) |
| Name a single object argument `params` | This skill's own convention (not lint-enforced) |
| Always write `@returns` (including `{void}`) | `jsdoc/require-returns` fires only when a value is actually returned, so a function returning nothing passes lint without the tag → **this skill is stricter than lint** |
| Attach a description to `@returns` (except `{void}`) | **`jsdoc/require-returns-description` is off**; lint does not require a description → **this skill is stricter than lint** (QA stance). The presence of `@returns` and its type are lint-enforced |
| Add `@public` to entry points | Adding `@public` is a skill convention; `check-access` validates that the notation is correct |
| Write array types as `Array<T>` (no `T[]`) | This skill's own convention (not lint-enforced) |
| Avoid vague types (`object`/`Object`/`any`/`*`); write the most specific type and concrete generics | This skill's own convention (not lint-enforced). However, using `object`/`Object` as a type makes `require-param`/`check-param-names` demand sub-property documentation, so a detailed type literal aligns with lint (reinforced). See also the "object types as named parameters" row |
| Prohibit `object`/`Object` (use `Record<string, *>`) | This skill's own convention (not lint-enforced). The reason is that `object` tolerates ambiguity including `null` |
| Write the any type as `*` (no `any`) | This skill's own convention (not lint-enforced). `*` itself is not overused either; it is a last resort for when the type cannot be narrowed |
| Do not write undefined type names (define custom types via `@typedef`/import) | `jsdoc/no-undefined-types` is **off** in the base default (overridden to `error` by this project's `eslint.config.js`). Built-in types and TS utility types (`Record`, etc.) count as defined. Enforced by this skill rather than relying on lint |
| Do not write `undefined` as a type (use `null`) | This skill's own convention (not lint-enforced). `undefined` is a defined type, so `no-undefined-types` allows it. Exception: permitted when a third party requires it |
| Do not place a delimiter after each chopped-down property | Inside a type literal, so lint does not parse it. This skill's own convention |
| Write `@typedef` on at least three lines | **Lint-enforced** by `multiline-blocks` (`noSingleLineBlocks: true`); a single-line typedef errors |
| One `@typedef` per block, separated by a blank line | This skill's own convention (not lint-enforced) |

## Intentionally relaxed rules (off)

These are `error` in the base (`@openreachtech/eslint-rules-default-jsdoc`) but are
turned **off** by `jsdoc.js` in this project. This skill's patterns rely on their
being off.

- **`jsdoc/no-types` = off** → you **can write type annotations** such as
  `@param {{ alpha: string }}` (the original recommendation forbids function type
  annotations). This skill's way of writing types depends on it.
- **`jsdoc/require-returns-description` = off** → lint does not require a description on
  `@returns` (this skill still asks for one; see the table above).
- **`jsdoc/require-description` = off** (unnecessary because `no-blank-blocks` exists) /
  **`jsdoc/require-param-description` = off** → block descriptions and `@param`
  descriptions are optional as far as lint is concerned.
- **`jsdoc/require-example` = off** → `@example` is not required.
- **`jsdoc/require-file-overview` = off** → `@file` / `@fileoverview` is not required.
- **`jsdoc/check-tag-names` = off** → unknown / custom tags (e.g. `@note`) may be used.
- **`jsdoc/match-description` / `require-description-complete-sentence` = off** →
  description style, leading capital, and trailing period are not enforced.
- **`jsdoc/valid-types` / `imports-as-dependencies` / `informative-docs` /
  `text-escaping` = off** → strict type syntax validation, import-based type
  dependency checks, and the like are not performed. Note that
  `jsdoc/no-undefined-types` is off in the base default but is overridden to `error`
  by this project's `eslint.config.js` (see the "do not write undefined type names"
  row above).
