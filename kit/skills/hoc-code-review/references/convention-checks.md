# Pass C — Convention Conformance

Whether the change is written the way this project writes code.

## The rule that governs this pass

**Check against the project's written conventions, not against generic best practice.**

Before raising any finding in this pass, identify the convention it violates and be able to name
it. A finding of the form "this would be cleaner as X" is not a convention finding; it is a
preference, and preferences are excluded by the review rules.

- Load the project's convention set first — the shared coding conventions the project follows
  (naming, class design and members, module structure, contracts and JSDoc, comments, constants,
  errors, coding style, documentation), plus anything the project states in its own instruction
  file.
- Where the project has no written convention on a point, two things stand in for one: **what
  the surrounding code already does**, and the **baseline below**. A change that departs from
  the pattern of the files around it is a finding; a change that follows an unwritten pattern
  you personally dislike is not.
- Formatting that the linter enforces is out of scope. Run the lint command, report its result
  in one line, and do not restate its output as findings.

## C0. The baseline (applies when nothing is written down)

A project with no convention document is not a project with no conventions to check. The
following are checked on every review, so that this pass can never come back empty for lack of
a written rule. **A documented project convention overrides any row here**, and a check the
linter already enforces is skipped.

| Baseline check | It is a finding when |
| :-- | :-- |
| Mysterious name | A name does not say what the thing is or does, so the reader must open it to find out |
| Duplicated code | The same decision or expression is written in more than one place, so a change has to be made twice |
| Feature envy | A function is more interested in another object's data than its own, and the behavior belongs on that object |
| Data clump | The same group of values is passed around together everywhere but has no name of its own |
| Primitive obsession | A domain concept is carried as a bare string or number, so nothing prevents an invalid one |
| Repeated switch | The same set of cases is branched on in several places, so adding a case means finding them all |
| Shotgun surgery | One conceptual change forces edits across many files |
| Divergent change | One module changes for several unrelated reasons |
| Speculative generality | An abstraction, option or hook exists for a need nobody has stated, with one caller |
| Message chain | The caller reaches through a chain of objects and so depends on the whole path |
| Middle man | A class does nothing but delegate, adding a hop without adding meaning |
| Refused bequest | A subclass inherits members it does not want and does not use |

- Cite the baseline row by name in the finding, and say plainly that it comes from the baseline
  rather than from a project rule — so the author can answer "we do it that way on purpose",
  which is a legitimate answer to a baseline finding and not to a documented-convention one.
- Baseline findings are `MEDIUM` at most, and `LOW` when the code is small and local. They lose
  to every correctness finding.

## Categories

Each category appears in the report as findings, `PASS` or `N/A`.

### C1. Naming

- Names of classes, methods, properties, accessors, constants and files follow the project's
  naming convention.
- A name says what the thing is or does, and is not contradicted by the implementation — a
  `get*` that writes, a `validate*` that transforms, an `is*` that returns something other than
  a boolean.
- Abbreviations, spelling variants and forbidden words follow the project's naming convention
  rather than the author's habit.
- New names are consistent with the names already used for the same concept elsewhere in the
  codebase. Two names for one concept is a finding.

### C2. Class design and members

- The class has state and a reason to exist as a class, per the project's class-design
  conventions. A class that is only a namespace for functions is a finding where the project
  prohibits it.
- Construction, property assignment, immutability and factory-method conventions are followed.
- Member visibility, accessor rules and the way members refer to each other follow the
  project's conventions.
- Responsibilities are not merged: a class that both decides and performs, or that serves two
  callers for two unrelated reasons, is a finding.

### C3. Module structure

- The file is in the directory the project's structure calls for, and its name matches what it
  exports.
- Import grouping and ordering follow the project's convention.
- The export shape follows the project's convention — in particular where the project requires a
  class per responsibility rather than a file of loose functions.
- Nothing new is exported that no other module consumes.

### C4. Contracts, types and JSDoc

- Arguments and return values follow the project's contract conventions, including how arguments
  are passed (positional versus named) and what a value-producing member returns when it cannot
  produce a value.
- Type declarations exist where the project requires them, and describe the actual shape.
- JSDoc follows the project's convention and is not contradicted by the code — a documented type
  that the implementation does not honour is a finding, and the wrong one to fix is the JSDoc.

### C5. Comments

- Comments follow the project's language and content conventions.
- A comment that restates the line above it, or that describes code that no longer exists, is a
  finding.
- Commented-out code is a finding.
- A comment that explains *why* a non-obvious choice was made is the one kind that should be
  present and is often missing — note its absence where the code makes a choice a reader would
  question.

### C6. Constants and configuration

- Values that the project's constant convention requires to be declared as constants are not
  written inline at their use sites.
- Constants are declared in the place and the file form the project requires.
- Values that vary by environment are read from configuration rather than hard-coded, and a new
  configuration value is declared wherever the project declares them.

### C7. Errors

- Error creation, propagation and message form follow the project's error conventions, including
  the convention for what a member returns when it cannot produce a value.
- Error identifiers are declared where the project declares them, rather than invented at the
  throw site.
- Messages follow the project's documentation convention for referring to class members.

### C8. Coding style beyond the linter

- Expression wrapping, chain breaking and argument layout follow the project's coding-style
  convention where the linter does not enforce them.
- Control flow follows the project's convention — nesting depth, the policy on ternaries, and
  the preference between statement sequences and higher-order forms.
- Blank-line and grouping conventions are followed where the project treats them as structural
  rather than cosmetic.

### C9. Documentation

- Documentation the project requires alongside a change exists and is updated: the README
  sections the project's convention names, the API reference for a changed public member, and
  the feature's own documents.
- The requirement definition document and the progress document for the feature reflect what was
  actually built. A change that silently diverges from its own specification documents is a
  finding in this category as well as in pass A.
- Notation for referring to class members follows the project's documentation convention.

## Severity in this pass

| Severity | When |
| :-- | :-- |
| `HIGH` | The departure will produce a defect — a JSDoc contract the code violates, an error identifier a caller cannot match on |
| `MEDIUM` | The departure will mislead the next reader, or splits one concept across two conventions |
| `LOW` | A local departure with no consequence beyond consistency |
| `INFO` | The project has no convention on the point; noted so it can be decided |

Never raise a convention finding above `MEDIUM` on consistency grounds alone. Convention
findings that outrank correctness findings in the report make the report unusable.
