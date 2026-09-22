---
name: hoc-type-errors
description: "How to read what a type checker reports, before deciding what to change: the reported count is not the size of the work while casts are still silencing errors, and an error may be pointing at a value that is wrong rather than an annotation that is missing. Use when a `tsc` or editor diagnostic is being cleared. Which annotation to write, and which casts are refused, belong to the JSDoc convention; what a test's fixtures hold belongs to the Jest convention."
---

# Shared: Type Errors

Two things a checker's output does not say, and both decide what happens next. What is then
written is `/hoc-jsdoc`'s, and this skill hands over to it.

## The reported count is not the size of the work

**A cast silences every error beneath it, so what the checker reports is only what nothing
covered.** The number is a measure of the gaps left uncovered, never of the gaps.

Measured on one test file: 36 errors reported, and 33 casts sitting elsewhere in the same file.
Taking those casts out surfaced 55 more. Nothing in the first number pointed at the second, and
an estimate built on it would have planned for a fraction of the work.

- **Take out what hides errors first, then read the output.** The count is worth having once
  nothing is suppressing it, and not before.
- **Removing a cast is a measurement, not a fix.** It reports what was already wrong. Treating
  the errors it surfaces as damage the removal caused gets the order backwards, and is the usual
  reason a cast goes back in.
- **Say the count after, not before, when the work is handed over.** A checklist written against
  the reported number understates what it is asking for.

## An error may be reporting the value, not the annotation

**Three parties meet at a type error** — the declaration, the type it names, and the value
standing under it — **and any of the three can be the one that is wrong.** The reflex is to read
every error as a declaration that was never written, which is right often enough to become a
habit and wrong in the case that matters most.

Measured on the same file: fixtures held error *instances* where the class under test builds its
hash of error *classes* and then calls `.create()` on what it finds. The declaration was right
and the implementation was right; the fixture had recorded a misunderstanding, and a cast had
kept it unreported. Writing an annotation there would have declared the misunderstanding rather
than corrected it.

**Read the implementation before the annotation.** What the code under test does with the value
settles which of the three dissents:

| What the checker met | What to change |
| :-- | :-- |
| The type and the implementation agree; the value does not | The value. It encodes a misunderstanding |
| The value and the implementation agree; nothing declares it | The annotation |
| The value and the implementation agree; the type contradicts both | The type — and only on that evidence |

**A value that dissents is never fixed by widening the type to accept it.** That records the
misunderstanding in the contract, where every later reader inherits it.

## Where this stops

**Which annotation to write, how far it may be reduced, and which casts are refused belong to
`/hoc-jsdoc`.** What a test's fixtures hold, and what may stand in for a collaborator, belong to
`/hoc-jest`. This skill settles only what the output is reporting, which is the question asked
before either of them.
