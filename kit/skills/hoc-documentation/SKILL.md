---
name: hoc-documentation
description: "Documentation writing conventions. Referenced when updating or writing READMEs, design documents, comments, etc. Defines what a document may state as fact and what it must reach for instead, the language a document generated for a reader is written in, and the notation used when referring to class members, among other things."
---

# Documentation

This gathers the conventions for writing documentation (READMEs, design documents, comments, etc.).

- When writing or updating documentation, follow the conventions in this skill.
- Follow this skill when writing or updating `SKILL.md` as well (referenced from the skill-updating convention).

## A document states only what it can check

**A fact a document cannot verify is a copy, and a copy rots.** Where the fact lives somewhere
else, the document holds a second record of it, and nothing tells a second record when the first
one moves.

Measured across four sibling packages, each carrying the same four-row table of what the others
hold: sixteen claims, five of them wrong. **Every wrong one was a row about a package other than
the one whose document it sat in.** The row each document made about itself was derivable from
the repository holding it, and not one of those had drifted.

- **Annotating the rot does not stop it.** The convention covering that table already said, in
  plain words, that those rows go stale and that no check it ran could catch them. The note stood
  while the rows were wrong: being right about the decay arrested none of it.
- **The remedy is removing the restatement, never maintaining it.** What replaced the copied
  column was a link to each package's own list. The fact did not disappear — it stopped being
  written down twice and became something the reader reaches instead.
- **A check makes a restatement correct and still does not make it worth keeping.** The places a
  repository-local audit gated never went wrong, and they were removed along with the rest. The
  check existed only because the document was holding a copy, so deleting the copy deleted the
  machinery with it.

**The test is not whether the fact is true today.** It is whether the repository holding the
document owns the fact. Where it does not, write what the reader can follow to the source; where
it does, ask whether the document needs to say it at all.

This is the same instinct as naming one governing source for a rule, turned on facts. **A rule
restated elsewhere is aligned to its source; a fact restated elsewhere is deleted in favour of
reaching it.**

## Notation of Class Members

- When referring to a class member within documentation, use the following notation.
- Instance members are prefixed with `#`, static members with `.`.
- **This skill is the governing source for this notation.** When another skill restates the same table, it aligns with the content here.

| notation | members |
| :-- | :-- |
| `#instanceProperty` | instance property |
| `#instanceMethod()` | instance method |
| `#get:instanceGetter` | instance getter |
| `#set:instanceSetter` | instance setter |
| `.staticProperty` | static property |
| `.staticMethod()` | static method |
| `.get:staticGetter` | static getter |
| `.set:staticSetter` | static setter |

- **Where the kind of member does not matter, write `#instanceMember` / `.staticMember`.** A rule that holds for every member alike turns on the prefix alone, and naming a kind it does not depend on would narrow it.
- When attaching the class name, write it as in `SampleClass#extractValue()`.

| notation | member |
| :-- | :-- |
| `SampleClass#extractValue()` | instance method of `SampleClass` |
| `SampleClass.createValue()` | static method of `SampleClass` |

## What language a document is written in

- **A document written for a person to read is written in the language that person is using.** If
  someone asks for a requirement definition in Japanese, the requirement definition is in Japanese.
  If they ask in English, it is in English.
- **An explicit instruction wins.** If the requester asks for a particular language, use that one,
  whatever language the conversation is in.
- **This skill is the governing source for this rule.** Other skills that produce a document refer
  to it in one line rather than restating it, so there is one place to change.

This covers anything generated for a reader — a requirement definition, a progress document, a
review or audit report, an acceptance report, a deployment runbook.

It does **not** cover the following, each of which has its own rule elsewhere.

| Not covered | Rule, and where it lives |
| :-- | :-- |
| Code and identifiers | ASCII only, in the naming convention |
| Comments in real code | English unless there is a reason otherwise, in the comment convention |
| `LICENSE` | The original text, unchanged |
| `SKILL.md` and its `references/` | English, so that every skill in a package reads the same way |

**A README follows the project.** Some projects keep one file, some keep one per language
(`README.md` alongside `README.ja.md`). Match what the repository already does rather than
introducing a second pattern.

**Why the rule is worth stating.** A document nobody can read has not been delivered. Writing a
requirement definition in English for a team that works in Japanese means the one person who has to
approve it reads it slowest — and approval is the step the document exists for.

## Scope of application (applies beyond prose)

- This notation applies not only to Markdown prose, but to **any text within implementation code that refers to a class member**. Specifically, this includes the following.
  - **Error messages** (message strings in `throw new Error(...)`, etc.)
  - **JSDoc / comments** (places within a member's description that refer to a member)
- When dynamically embedding a class name, use the same notation. Prefix instance members with `#` and static members with `.`.

```javascript
// OK: instance method (the class name is resolved via this.constructor.name)
throw new Error(`${this.constructor.name}#normalize() must be inherited`)

// OK: static getter / static method (the class name is resolved via this.name)
throw new Error(`${this.name}.get:rawSchema must be inherited`)
throw new Error(`${this.name}.generateCredential() must be inherited`)
```
