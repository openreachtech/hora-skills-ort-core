---
name: hoc-jest
description: "Write Jest unit tests for JavaScript classes, and for the modules, data files and reconciliations a project tests alongside them. Use this skill whenever the user asks to create or update Jest tests — for a class, for a module of exported constants, for a data file such as a message catalogue, or for a test that two collections still agree with each other."
---

# Jest Testing

A skill for writing Jest unit tests for JavaScript classes, and for the modules
and data files a project tests alongside them.
The conventions are split across the detail files below. Refer to them as needed.

> **Notation convention**: Throughout this skill, when we simply write
> `describe()` / `test()` / `expect()`, each is a **generic term that implies**
> the corresponding `.each()` variant (`describe()` includes `describe.each()`,
> `test()` includes `test.each()`, and `expect()` includes `expect.each()`).
> Where the context is limited to one or the other specifically, that will be stated explicitly.
>
> Also, use `test()` / `test.each()` for test blocks and never `it()` / `it.each()`
> (`it()` is easy to misread as `if()`; lint also enforces `test` via
> `jest/consistent-test-it`; see [eslint-jest-rules.md](./references/eslint-jest-rules.md)).

## Core principle: don't cut coverage using implementation knowledge (QA stance)

Tests must be written so that they **notice even if the implementation is wrong**.
Using knowledge gained from reading the implementation to judge "this will always
be this value" or "this member is obvious" and then **omitting cases or members,
or collapsing them into a single `test()`, is a QA mistake**. What a test must
guarantee is not the implementation's internal circumstances, but the member's
**contract (externally observable behavior) as a black box**.

- Do not omit something because "the implementation is written this way so it's
  fine." If you do, hard-coding or regressions can slip in without the test
  noticing, which defeats the purpose of having a test at all.
- Example: for the instance getter `get Ctor () { return this.constructor }`,
  collapsing it into a single `test()` that says "in the base class it's always
  `BoundCtorRegistry`" would still pass even for the hard-coded implementation
  `return BoundCtorRegistry`. Only by running the variable element (the
  instance's type) through `test.each()` can you verify the contract
  ([structure.md](./references/structure.md#instance-getters-require-testeach-the-variable-element-is-the-instance)).
- Example: don't skip testing a static property because "the memoization
  pool is an implementation detail." Members that appear public should be
  verified as part of the contract.
- Example: for a member that **transforms a property/argument before using it**
  (escape / normalize / encode, etc.), don't test only with values for which the
  transformation is a no-op. Always include **a value that actually exercises the
  transformation** so that a broken transformation would be noticed
  ([test-cases.md](./references/test-cases.md#choose-values-that-exercise-the-internal-transformation-dont-fill-cases-with-no-op-values-only)).
- When in doubt, err **on the side of coverage**. When trimming a member,
  branch, or variable element, the justification must be "this input/state
  cannot exist under the contract," never "because I know the implementation."

## Core principle: don't put logic in test files

A test must consist **only of assertions** (`expect()`) that **check simple
input and output values**. Writing logic inside a test to prepare it is a
well-known anti-pattern. Test code itself is not verified, so if raw logic
written inside a test is wrong, nobody notices, and the reliability of the test
collapses.

- The only things allowed for use inside a test are things that **have already
  been tested elsewhere and have guaranteed behavior** (already-tested classes,
  factory methods, literal data, etc.). Do not write unverified transformations,
  branches, or helper definitions.
- Inside `describe()`, do not use `if` / ternary / `??` / short-circuit
  evaluation / higher-order functions / `forEach`. Express repetition with
  `expect.each()` from `@openreachtech/jest-expect-each`.
- See [anti-pattern.md](./references/anti-pattern.md) for details and examples.

## Core principle: index the first and second levels by definition name (class/member)

The nesting of `describe()` is fixed as **level 1 = class name**,
**level 2 = member name**. Do not write behavior (`should ...` / `when ...`)
directly at levels 1 or 2. Behavior must always go at **level 3 or deeper**,
following the order `describe(class) > describe(member) > describe(behavior) >
test()`.

- **Why**: One of the main purposes of a test is that **a human can later
  look up the test for a given target**. When fixing a member in the
  implementation, one must be able to reach the corresponding test with the
  shortest path possible. Fixing levels 1 and 2 to definition names makes the
  **implementation structure (class → member) directly the index key of the
  tests**, so code and tests can be looked up one-to-one, in either direction.
- A behavior-first style (BDD-leaning writing that starts with
  `describe('should return 401 when ...')`) reads well as a specification, but
  it is slow to look up "where is the test for `FooResolver#resolve()`."
  Because the class → member correspondence has no syntactic anchor, finding
  the corresponding test from the implementation requires grepping the whole
  file or tree exploration. This skill prioritizes lowering this **search
  cost** through syntax.
- **Prevents duplicate tests**: if you cannot look up "where" the test for a
  target member is, you cannot check whether an equivalent test already
  exists, and **similar tests end up added redundantly**. This is especially
  true **when multiple people work on the same codebase** — each person adds
  tests without being able to find existing ones, which easily produces
  duplication and coverage gaps. Indexing by definition name concentrates
  "the test for this member is in exactly one place," letting you check
  existing tests before adding new ones.
- As a consequence of this policy, the class-name `describe()` is **repeated**
  per member (one class `describe()` holds only one member). Even when
  fast-scrolling a long file, the class name always sits directly above the
  member `describe()`, so "which class, which member am I looking at" always
  stays on screen (a syntactically achieved sticky header). We accept that the
  class name is duplicated in Jest's output, in favor of human explorability.
- **When the subject is not a class** — a module of exported constants, or a
  data file such as a message catalogue — the index rule still holds, but what
  sits at each level changes. **A module reads level 1 off whatever it
  default-exports**, and indexes below that by the kind of export and its name; a
  data file is indexed by its path. **A reconciliation between two collections is
  the one case where the rule cannot hold**, because neither side is the subject;
  it is indexed by the relation instead. See
  [structure.md](./references/structure.md#when-the-subject-is-not-a-class).
- For details on nesting structure and notation, see
  [structure.md](./references/structure.md#describe-structure) /
  [naming.md](./references/naming.md#notation-of-class-members).

## Write comments in the code in English

Comments written inside test code (`.js`) must be written in **English**. This
unifies the comment language to English, aligning with other comments in the
codebase.

- This applies to **comments inside the test code generated under tests/**
  (`// same reference` / `// neutral value; not under test` /
  `// all omitted → default; keep last`, etc. — `//` or `/* */` written in
  `.js` files). These are generated artifacts, hence English.
- Comments inside **this skill's own examples** (` ```js ``` ` blocks) follow the
  same rule, since they illustrate the very code the rule governs.

## Detail files

- [directory.md](./references/directory.md) — directory layout, import paths
- [structure.md](./references/structure.md) — structure of describe / test
- [test-cases.md](./references/test-cases.md) — data conventions for `cases`
- [aaa-pattern.md](./references/aaa-pattern.md) — Arrange / Act / Assert
- [mocks.md](./references/mocks.md) — mocks/stubs (inside Arrange, overridden with `jest.spyOn()`)
- [naming.md](./references/naming.md) — naming of variables and case properties
- [types.md](./references/types.md) — type annotations and type resolution
- [anti-pattern.md](./references/anti-pattern.md) — don't put logic in tests (forbidden syntax, extracting helpers)
- [prohibit.md](./references/prohibit.md) — prohibited items (forbidden matchers, etc.)
- [eslint-jest-rules.md](./references/eslint-jest-rules.md) — ESLint (jest plugin) mapping (follow it and `npm run lint` passes; intentionally relaxed rules)
