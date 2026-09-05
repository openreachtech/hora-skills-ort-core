# Directory (Directory Structure and Imports)

Conventions for where test files are placed and how import paths are written in
jest. Referenced from `SKILL.md`.

## Why We Don't Co-locate (Reasons for Isolating Tests in a Dedicated Tree)

Tests do not use **co-location** (placing `Foo.test.js` next to `Foo.js` in the
same directory as the source); instead they are isolated in a **dedicated
`tests/` tree** (placement details are below in
[Directory Structure](#directory-structure)). The answer to "why not
co-location" comes down to four points, all of which share the same root:
**with a directory boundary, you can state "everything inside the boundary is X"
as a positive allowlist, but co-location forces you to state "everything,
except tests" as a negative denylist everywhere** — and a denylist is always a
source of bugs from **falling out of sync with test naming conventions**.

Note that test discoverability (where the tests for a given member live) is
guaranteed by the describe syntax (the core principle in [SKILL.md](../SKILL.md),
"index the 1st and 2nd levels by definition name"). Since discovery does not
rely on placement, placement can be chosen freely, letting us capture the
benefits listed below.

1. **Configuration can be strongly scoped per directory.**
   ESLint overrides / tsconfig / jest environment / coverage targets, etc. can
   all be applied **wholesale** to `tests/**`. You can guarantee "loosen or
   tighten rules for tests only" by **location**, without relying on file-name
   glob discipline (`*.test.js`). With co-location, you end up relying on globs,
   and suffix inconsistencies let rules **slip through or misfire on production
   code**.

2. **It doesn't pollute loaders that walk a directory (directory-as-registry).**
   Many implementations, like a `DeepCtorsLoader` that loads every class under
   `server/resolvers/`, **use the folder itself as a registry (declaration)**.
   Co-location degrades the contract "everything under here = every resolver"
   into "everything under here, **except tests**," forcing the
   **production/runtime loader to carry exclusion logic** (a layering
   violation). Any gap in that filter becomes a production bug where a test
   file gets **loaded as a resolver at runtime**. With a dedicated tree, the
   implementation folder stays pure, and the loader can walk it
   **unconditionally**.

3. **npm publish can be handled with a simple allowlist.**
   If `lib/` is pure, the publish spec is a one-liner: `"files": ["lib"]`. With
   co-location, you either publish the tests along with everything else,
   **polluting the package** (bloated installs, broken references to
   devDependencies, information leaks via fixtures), or you're stuck
   **endlessly maintaining exclusions** in `.npmignore` (e.g.
   `**/*.test.js`) — and any gap leaks tests into the production package.

4. **It complements a build that skips compilation.** This package **publishes
   `lib/*.js` as-is** (ESM, no build; types are checked only via JSDoc +
   `tsc --noEmit`). This gives real benefits: no build toolchain needed, faster
   CI, and "what you debug is what you ship" (no source maps needed, stack
   traces point to real file lines). At the same time, this setup has **no emit
   step** (the filter a TS compile configuration would use to filter `.test.ts`
   out of build artifacts). A dedicated tree keeps `lib/` pure from the start,
   which is **consistent** with the absence of that filter. Co-location combined
   with no-compile is the **worst combination** — tests sit in `lib/` with no
   filtering boundary at all, forcing you to manually fill in all of the
   denylists from points 2 and 3 above.

## Directory Structure

The location of tests mirrors the directory structure under the source's
`lib/` directly under `tests/__tests__/`.

- `lib/` is the mirroring base point. Do not include `lib/` in the test-side
  path.
- Place the remaining path, with `lib/` stripped off, directly under
  `tests/__tests__/`.

Example:

| Source | Test |
| --- | --- |
| `lib/tools/PathnameBuilder.js` | `tests/__tests__/tools/PathnameBuilder.js` |
| `lib/Foo.js` | `tests/__tests__/Foo.js` |
| `lib/a/b/Bar.js` | `tests/__tests__/a/b/Bar.js` |

Incorrect example: `tests/__tests__/lib/tools/PathnameBuilder.js`
(must not include `lib/`)

A test whose subject is a **data file** mirrors that file's path the same way,
with the varying segment — written `*` — collapsed into a single test file. The
describe name at level 1 then matches the source path it mirrors
([structure.md](./structure.md#a-data-file-is-indexed-by-its-path)).

| Source | Test |
| --- | --- |
| `lib/i18n/locales/*/message.json` | `tests/__tests__/i18n/locales/message.js` |
| `lib/fixtures/*/user.json` | `tests/__tests__/fixtures/user.js` |

## Splitting Large Test Files

If a single test file grows too large, it may be **split by method**. In that
case, use **the class name as a directory** and place per-method files
underneath it.

- Before splitting: `tests/__tests__/tools/PathnameBuilder.js`
- After splitting: turn the class name `PathnameBuilder` into a directory.

```
tests/__tests__/tools/PathnameBuilder/
  constructor.js
  create.js
  buildPathname.js
```

- Directory mirroring (not including `lib/`, etc.) stays the same after
  splitting: `lib/tools/PathnameBuilder.js` ↔
  `tests/__tests__/tools/PathnameBuilder/`.
- Splitting is a remedy for bloat and is not mandatory. A single file is fine
  while it stays small.

## Test Tools (Where to Place Helper Functions)

Do not define helper functions inside a test file
([anti-pattern.md](./anti-pattern.md)). If one is truly necessary, **define** it
under `tests/tools/`, and **write its test at that same path mirrored under
`tests/__tests__/`** — the path carried across whole, `tests/` included.

| Helper | Its test |
| --- | --- |
| `tests/tools/makeSample.js` | `tests/__tests__/tests/tools/makeSample.js` |
| `tests/tools/foo/makeSample.js` | `tests/__tests__/tests/tools/foo/makeSample.js` |

- **`lib/` is stripped; nothing else is.** The mirroring base is the source root, and
  `lib/` drops out because it *is* that root's name (Directory Structure, above). A
  helper does not live under `lib/`, so nothing drops out of its path — which is why
  the test path repeats `tests/`.
- **This is what keeps the two trees from colliding.** `lib/tools/` and `tests/tools/`
  are different directories sharing a tail. Strip `tests/` and both of their tests land
  on `tests/__tests__/tools/`, where the second one written has to be renamed to fit.
  Carrying the path whole gives each its own place, with no extra rule to remember.
- Only helper functions that have tests written for them may be used inside
  test files.

## Import Path

Import the subject under test using a **relative path**, not an alias like `~`.
The relative path traces the directory structure directly.

```js
// Correct (from tests/__tests__/tools/PathnameBuilder.js)
import PathnameBuilder from '../../../lib/tools/PathnameBuilder.js'

// Incorrect
import PathnameBuilder from '~/lib/tools/PathnameBuilder.js'
```

## Import Order

Among classes imported from this package (`lib/`), write **the subject under
test first**. Separate subsequent imports (base classes, etc.) from the subject
import with a **blank line**.

- Placing the subject under test first makes "what this file tests" clear from
  the very first line.
- Subsequent imports (base classes, collaborator classes, etc.) are grouped
  together after the blank line.

```js
// Correct (testing a subclass: subject first, base class separated by a blank line)
import BasicAuthorizationBuilder from '../../../../../lib/request/authorization-builder/concretes/BasicAuthorizationBuilder.js'

import BaseAuthorizationBuilder from '../../../../../lib/request/authorization-builder/BaseAuthorizationBuilder.js'
```

```js
// Incorrect (subject and base class listed with no blank line between them)
import BasicAuthorizationBuilder from '.../concretes/BasicAuthorizationBuilder.js'
import BaseAuthorizationBuilder from '.../BaseAuthorizationBuilder.js'
```
