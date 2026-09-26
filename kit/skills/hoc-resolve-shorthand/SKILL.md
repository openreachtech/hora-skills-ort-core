---
name: hoc-resolve-shorthand
description: "Resolve a shorthand where the reader meets it, so that following it costs no lookup. Covers the handle standing in for something longer — a phase or issue number, a rule number, a letter invented for an entry in a list — the name to write instead where one already exists, the content a real identifier has to carry with it, and the origin an invented label owes. Applies to anything written for a reader. The `#instanceMember` notation belongs to the documentation convention."
---

# Resolve Shorthand

**A handle the reader cannot resolve where they meet it is a question you have made them ask.**
A phase number, an issue number, a rule number, a letter standing for an entry in a list — each
is short because something longer stands behind it, and the writer is the one who can see both
halves. Whatever the reader would otherwise have to open, scroll back through or search for
belongs beside the handle.

**This governs anything written for a reader** — a reply, an issue or pull request body, a
document, a commit message. The reader is whoever meets the sentence, and they are never assumed
to be holding what the writer was holding.

## Where a name already exists, do not invent a handle

Measured: a queue of three items awaiting a decision was presented as `A`, `B` and `C`, each row
naming what it covered. The reply was a question asking which of the three the reader had just
supplied. **Every item already had a name** — the convention it was bound for — and the letters
replaced names that would have answered that question by themselves.

- **An invented label costs a lookup that the real name does not.** `A` has to be carried from the
  table to the sentence using it; a name carries itself.
- **A letter earns its cost only where nothing shorter is available**, which is rare. A list the
  reader answers by number is a different case, and the convention owning that list settles it.

## Where the handle is the name, carry its content

Some handles are not inventions. A phase number, a section number, an issue number and a rule
number are what the thing is called, and replacing them loses the reference. Resolve them in
place instead.

```
Bad:  dropping phases 6 and 7 leaves those rules with nowhere to live
Good: dropping phase 6 (Write) and phase 7 (Verify) leaves those rules with nowhere to live
```

**Measured: a proposal to remove two numbered phases was argued through to its consequences
without either phase being stated.** The reply asked what the two were. Everything else in that
answer was correct, and none of it could be weighed.

## A label invented here carries where each entry came from

Where a list is handed back to the person who supplied its entries, each row says what put it
there. **The subject alone does not answer *which of these is mine*,** because the reader is
matching against what they asked for rather than against what it is about. Two rows can differ
by one word in their subject and by a week in their origin.

## The test is the question you were asked

**By the time the reader asks *what is that?* or *which one?*, it has already failed.** The repair
is not to answer; it is to have written it. A question of that shape is the measurement, and it
names the handle that went unresolved.

## What resolving costs

**Enough that the reader does not leave the sentence, and no more.** A parenthetical usually does
it. Restating a whole section because one of its numbers was mentioned is the opposite failure,
and the documentation convention governs that: what the reader can follow to the source stays at
the source.

## Where this stops

- **The `#instanceMember` and `.staticMember` form** for referring to a class member is the
  documentation convention's.
- **What a document may state as fact**, and the restatement it deletes in favour of a link,
  belong there too. This convention governs the reference too short to follow, never the copy too
  long to keep.
- **Whether a list carries numbers at all** is decided by the convention owning that list. A
  checklist refuses them and a report written to be replied to requires them.
