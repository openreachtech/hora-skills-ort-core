# Skills

A catalog of every skill in this package — 39 in total — with a one- or two-line summary each.

Each skill lives at `kit/skills/core/<name>/`, one level under the domain directory, and that folder name is both the skill's `name:` and the folder name it is installed under. **Skill** below is therefore all you need: it is what you invoke as `/name`, what appears under `.claude/skills/` once installed, and where the source sits. The three-character prefix is the domain — see [the flatten build convention](https://github.com/openreachtech/hora-skills-ort-core/blob/main/.claude/skills/flatten/SKILL.md) for the layout and the naming rules. Full guidance for any skill is in its own `SKILL.md`.

| Skill (= Command) | Summary |
| :-- | :-- |
| `hoc-accessors` | Getter/setter conventions — setters prohibited for immutability, `#get:Ctor` reserved for `this.constructor`, and dependency references extracted into getters. |
| `hoc-async` | Asynchronous code conventions. When writing Promises, use `async`/`await` whenever possible. |
| `hoc-charters-coding` | The ORT coding charter: write readable, unified code; avoid modification; let the code explain everything. |
| `hoc-classes-constructor` | Class constructor conventions. Constructor parameters must not have default values. |
| `hoc-classes-inflators` | The inflator (binding) method pattern — bind the class passed as an argument and return a derived subclass memoized via `BoundCtorRegistry` — plus its naming and arguments. |
| `hoc-classes-notations` | The order members are written in a class body: the eight-block placement order, plus ordering within getters and within methods. |
| `hoc-classes-principles` | Class design principles — no classes without properties, and the system underpinning it (deep immutability, constructor-only, references-as-contract). |
| `hoc-classes-prohibits` | Prohibitions in class definitions: static-only classes and classes without state are not allowed, and why. |
| `hoc-code-review` | Read-only, code-level review of a change, producing a findings report on specification compliance, correctness and convention conformance. Never fixes anything. |
| `hoc-coding-styles` | Coding style — where to chop down expressions, method/property chains, call arguments, template literals and regular-expression flags. |
| `hoc-comments` | Comments within actual code are written in English unless there is a reason otherwise. |
| `hoc-constants` | Constant conventions — uppercase `SNAKE_CASE` naming, chopping down, and the file organization and placement of object-type constants. |
| `hoc-contracts` | Type contracts for function and method arguments and return values, and how contract types are defined. |
| `hoc-dependency-defect` | Work around a bug in code this project uses but does not own — a subclass that overrides only the broken member, called by its own name, with a comment saying when it can be deleted. |
| `hoc-deployment-document` | Write a server deployment runbook through conversation — the hosting profile, the first-time build, the repeatable release, migrations, rollback, and the output that confirms each step worked. |
| `hoc-documentation` | Documentation writing conventions, including the `#instanceMember` / `.staticMember` notation used when referring to class members. |
| `hoc-errors` | Error handling — return `null` on failure from value-generating methods, and the throw-message format for abstract members. |
| `hoc-functions` | Function conventions. Parameters follow method parameters: named arguments as a principle. |
| `hoc-git-branch` | Conventions for the branches a repository carries — which one is a trunk and what that role obliges, how a general branch is named, the empty marker commit that opens a trunk, and the `--no-ff` merge that closes a sub-branch. |
| `hoc-git-commit` | Commit conventions — what belongs in a single commit and the order commits land in, the message format (imperative or Conventional Commits, chosen per project), and the verb vocabulary shared by both. |
| `hoc-implementation-progress` | Track an in-flight implementation in a progress document anchored to requirement ids, advancing a status only against recorded evidence. |
| `hoc-jest` | Write Jest unit tests for JavaScript classes, modules of exported constants, and data files such as message catalogues. |
| `hoc-jsdoc` | JSDoc writing conventions shared by backend and frontend — type annotations, `@returns`, `@typedef` and type-only imports, with the Vue/Nuxt-specific conventions in its references. |
| `hoc-license` | Write and update a project's LICENSE file. |
| `hoc-methods` | Method definition conventions — named arguments, passing properties into private methods, and factory methods. |
| `hoc-modules-exports` | Don't define files that merely named-export a function; define a class per responsibility. |
| `hoc-modules-imports` | Group imports at the top of the file, ordered from farthest to nearest to application development. |
| `hoc-naming` | Naming for classes, methods, properties and accessors — datetime suffixes (`At`/`On`, `From`/`To`), abbreviation criteria, American spelling, forbidden words, ASCII only. |
| `hoc-npm-install-scripts` | The gate deciding which packages may run an install script — denial as the resting state, the settings placed before the install they govern, and the dry run that reports a script without executing one. |
| `hoc-npm-publish` | Where a release's version bump sits among the commits, what to do when it turns out not to be last, and the audit that reads the artefact a consumer receives before anything goes out. |
| `hoc-npm-vulnerability` | Keeping a vulnerable version out — what the audit does and does not see, the release-age quarantine and the install it does not apply to, and resolving a report by raising a transitive dependency. |
| `hoc-properties` | Property conventions — set on `this` in the constructor, immutable (no reassignment, no `Map`), and no JavaScript native private. |
| `hoc-readme` | Write and update a project's README. |
| `hoc-requirement-definition` | Turn a rough request into a requirement definition document through conversation — requirements, acceptance criteria, out-of-scope list, open questions. |
| `hoc-scope` | Scope references among class members — `this` between static members, and `#get:Ctor` when referring from an instance to a static member. |
| `hoc-skill-updating` | Conventions for creating and updating skills — how to name one, placement rules, directory structure, and how to write a `SKILL.md`. |
| `hoc-statements` | Statements and control flow — no literal `undefined` in production code, higher-order functions over sequential processing, and ternary/`if` policies. |
| `hoc-test-execution` | Run a project's tests and drive them to green without weakening them — nothing skipped, deleted, loosened or waited out to make the suite pass. |
| `hoc-workflows` | Development workflow rules — how to proceed with an implementation, and the steps always performed before committing and before completion. |
