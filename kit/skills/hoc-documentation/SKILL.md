---
name: hoc-documentation
description: "Documentation writing conventions for READMEs, design documents and any other prose a project carries. Use when writing or updating a document, and when a change moves a fact a document states. The README's own structure belongs to the README convention; which language a document is written in, to the writing-language convention."
---

# Documentation

This gathers the conventions for writing documentation (READMEs, design documents, etc.).

- When writing or updating documentation, follow the conventions in this skill.
- Follow this skill when writing or updating `SKILL.md` as well (referenced from the skill-updating convention).

## What language a document is written in

- Which language a document is written in belongs to `/hoc-writing-language`. It is read here for
  the default it gives a document written for a reader — the language that reader is using — which
  holds wherever no instruction, and no language code in the file's name, has settled it first.

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

### A fact the repository owns moves with every copy of it

**Owning a fact decides whether a document may state it. It does not keep the statement in
step.** A document stating a fact its own repository holds is still carrying a copy, and that copy
rots exactly as the copied rows above did. So a change that moves such a fact moves every copy of
it, in the same piece of work.

Measured: a change moved a parameter's default by editing the one line of code that held it. The
API reference, kept in two languages, had stated the old default for less than two hours, and was
still stating it in both when the change reached a release branch.

- **What obliges the update is a moved fact, not a touched file.** A default, a return value, what
  is thrown — anything the document states as the behaviour. Code a document merely mentions can
  be rewritten underneath it without any of that moving, and then the document owes nothing.
- **Find the copies by searching for the old value, as the document writes it**, across `docs/`
  and every `README*`. A value the document sets in markup — in backticks, in a table cell — is
  searched for with that markup around it, because the bare value can fail to match the line that
  holds it.
- **Every language version is a copy.** A document kept in two languages states the old value
  twice, and repairing one leaves the other asserting it.

## Notation of Class Members

- How a class member is written when a document refers to it belongs to
  `/hoc-classes-member-notation`.
