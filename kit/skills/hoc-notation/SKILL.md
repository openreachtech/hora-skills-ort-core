---
name: hoc-notation
description: "How a text that names several things of one kind is written: never as an enumeration, whatever the connector. Use when writing or reviewing any text that names more than one thing, or when no single name for the whole can be found. Which language a text is written in belongs to the writing-language convention; what a document states, to the documentation convention."
---

# Notation

This gathers the convention for how a written text names several things of one kind. It holds for
every kind of text, so any other convention may refer to it, and it holds in the text of a skill as
much as anywhere else.

## An enumeration is never written

**An enumeration is several things of one kind named side by side in one place of a sentence.** It
is never written. The kind of text decides nothing, and there is no exception.

| Where it is written | Example |
| :-- | :-- |
| An item of a list | `- [ ] Raise alpha and beta` |
| A title | `Retire the alpha and beta endpoints` |
| The subject of a commit | `Ignore node_modules, dist and coverage` |
| The name of a branch | `update/alpha-and-beta` |
| A sentence of prose | `The script reads the lock and the manifest` |

**The connector is not what is refused.** Whatever carries the list makes an enumeration.

| Shape refused | Example |
| :-- | :-- |
| `and` | `Raise alpha and beta` |
| `or` | `Remove alpha or beta` |
| `&` | `Raise alpha & beta` |
| `+` | `Raise alpha + beta` |
| `/` | `Raise alpha/beta` |
| `plus` | `Raise alpha plus beta` |
| `as well as` | `Raise alpha as well as beta` |
| a bare comma list | `Raise alpha, beta, gamma` |

**The word `and` is not the target, and refusing it mechanically is not the aim.** A sentence that
joins two clauses, each with a subject of its own, lists nothing. What decides is whether the
connector puts several things of one kind into one position of one frame.

```
Good  The script reads the lock, and the report names each action
Bad   The script reads the lock and the manifest
```

The first sentence makes two statements about two subjects. The second names two things the script
reads, and that is the enumeration.

## One covering name, or two pieces of work

**Where the parts belong to one piece of work, one name at the altitude they sit beneath states the
whole, and that name is the one written.**

```
Bad   Ignore node_modules, dist and coverage
Good  Ignore the regenerable output
```

**Where no one name can state the whole, the parts are different pieces of work mixed together,
and the text is split.** Failing to find the covering name is evidence about the work, never an
obstacle to be written around. Reaching for a connector to carry two parts is how two pieces of
work pass as one: a commit joining a new class with the repair of an unrelated comment holds two
hunks, and any other text does the same.

| Written | What it holds | Fix |
| :-- | :-- | :-- |
| `Add AlphaValidator and tidy an unrelated comment` | Two pieces of work. No name covers both | Two lines: one for each |
| `Raise alpha, beta and gamma` | One operation on three targets | One line for each target |
| `Ignore node_modules, dist and coverage` | One piece of work. A name covers all three | `Ignore the regenerable output` |

**The test is whether the covering name still holds when the parts change.** A name that
survives is the altitude of the work. A name that has to be rewritten each time is a list written
as a noun, and the work under it has not been named yet.

## Two files carrying one fix

**A document kept in two language versions is one document.** A line about a fix to it names the
document, and never the two files.

```
Bad   Add the quick start to guide.md and its translation guide.ja.md
Good  Add the quick start to the guide
```

Which files the fix touched belongs to the diff. Naming them in the line turns one fix into an
enumeration of files.

## What is written instead

**The convention owns the replacement as much as the refusal.** A refusal with nothing to put in
its place sends the writer to a different connector, which is the same enumeration.

| What the enumeration was carrying | Write instead | Why |
| :-- | :-- | :-- |
| Several targets of one operation, each of which could be done while the others stay undone | One line for each target, with the operation repeated on every line | One checkbox beside a joined line can only be whole or untouched. It cannot say that half is done |
| Several things compared on what differs between them | A table: the rows are the things, the columns are what differs | A reader checking one case scans a column. Parsing a sentence costs more |
| A value that every line repeats | A heading above the lines that holds the value. Each line keeps its verb | What the lines share is not what the reader came for |
| The parts of one piece of work | One name at the altitude the parts sit beneath | The name is what a reader scanning a list of titles meets |
| Parts that have no covering name | Separate pieces of work: separate commits, separate issues, separate branches | A part that cannot be named with the others was never one thing |

```
Bad   - [ ] Raise alpha and beta
Good  - [ ] Raise alpha to ^1.2.0
      - [ ] Raise beta to ^3.0.0
```

**Lines that differ only in which thing they name are one subject.** Writing each on a line of its
own does not call for a heading over each. A heading earns its place where the lines repeat one
value.

## Where it stops

This convention settles how several things are named, and nothing beyond that.

- Which natural language a text is written in belongs to `/hoc-writing-language`.
- What a document may state as fact belongs to `/hoc-documentation`.
- How a class member is written when it is named belongs to `/hoc-classes-member-notation`.
- Which sections a written artefact carries belongs to the convention for that artefact. The rules
  above hold inside every one of them.
