---
name: hoc-git-commit-order
description: "The order commits land in once the work has been split: which of a test and the change it checks goes first, and how the rest of a branch is sequenced. Use when deciding which commit comes first, when tests and implementation change together, or when something is removed or a limit is lowered. What belongs in a single commit belongs to the git commit convention; which test cases fail against the old code, to the Jest convention."
---

# Git Commit Order

What belongs in each commit is settled first, by the git commit convention. This settles which
of them comes first.

## The first commit is the one that turns the suite red

**Where a change and the tests that check it are separate commits, whichever of the two makes
the suite fail goes first.** The red commit states the claim on its own, and the commit after it
is what turns the suite green. A reader stands at the red commit and watches the test fail for
the reason it exists; taken the other way round, every step is green and nothing distinguishes a
test that checks the change from one that asserts nothing.

### Adding puts the tests first

**A class, a member or a behavior that is being added has its tests committed before its
implementation.** The test commit states what the code is expected to do, and the implementation
commit is the one that makes the statement true. Committed the other way round, the tests only
confirm what already worked, and there is no commit at which the claim stands on its own to be
reviewed. This is test-driven development written into the history rather than into the editor.

- **A test commit in which nothing fails asserts nothing.** It may hold cases that pass —
  covering behavior the change leaves alone — but not only those. Which cases fail against the
  unchanged code, and which pass, is read as the Jest convention (`/hoc-jest`) describes.

### Reducing puts the implementation first

**Where the work takes something away, the implementation changes first and the tests follow.**
Removing a method, removing a class, lowering a cap the implementation enforces, such as the
length it allows a `description:` — taking the tests out, or bringing them into line, first turns
nothing red, because the code they check still satisfies them. So the implementation goes first and fails the suite, and
the commit that brings the tests into line is the one that clears it.

- **In this order the red persists until the correct tests come out.** Remove
  the wrong ones and the suite is still failing, because the tests for the
  member just deleted are still sitting there. That is the check doing its
  work.
- **The reverse order is green at every step**, because a suite with fewer
  tests still passes. Removing tests first and the member second reports
  nothing at either commit, whether or not the right tests were taken.
- **It is silent exactly where it matters most.** Take a member that never had
  tests of its own. Remove tests first, take another member's by mistake, then
  remove the member: both commits pass, the other member is now uncovered, and
  nothing anywhere said so. There was no red to go missing, because there was
  never a test to turn red.

### A test that holds a ceiling

**A byte budget, a size limit, a threshold recorded per file: which side goes first follows from
the same rule.** Raising the ceiling first turns nothing red, because nothing is over it yet. So
the change it guards goes first and fails the suite, and the raised ceiling after it is what
passes. Shrinking runs the other way — the lowered ceiling first, failing, then the cut that fits
under it. **Commit whichever side turns the suite red first**; that is the rule above, read
against the kind of test rather than the word "test".

- Measured on a kit of skills held to per-file byte budgets: with each budget committed ahead of
  the text it made room for, none of eight budget commits failed. Reordered, all eight text
  commits failed and every budget after them passed.

## The rest of the line: behavior, structure, addition

Splitting settles what each commit holds, not which one comes first. **The sequence is behavior
change, then structural change, then addition.**

- **A change to how an existing rule behaves leads the branch**, however small it is. One line
  earns a commit of its own here. At the head it stands directly against the state it changed,
  so a reviewer compares the two with nothing structural in between, and it reverts on its own
  if the behavior turns out wrong.
- **Structural change comes next** — a section renamed, two of them folded into one, a module
  moved. It carries the tree from one shape to another and claims no new ground.
- **Addition comes last**, into the shape that is settled by then.

**What separates the three is nature, not size.** A single line that changes what an existing
entry matches is a behavior change; a hundred lines of new entries are an addition. Ordering by
how much a commit touches buries the one risky line in the middle of the branch, which is the
one place it must not be.

**An addition does not go inside a structural change.** A new section is structure rather than
addition, so whatever belongs in it waits for a later commit. Filled as it is created, the
structural change comes apart around its contents, and a reader watches the same shape being
assembled twice.

```
Anchor dist to the repository root                                behavior
Move .env into a new Secrets section                              structure
Rename environment to Regenerable output and absorb Build output  structure
Add coverage, tgz and eslintcache to Regenerable output           addition
Add key and certificate patterns to Secrets                       addition
```

The first commit is one line of a `.gitignore`, and it leads because it is the only one that
changes what an existing entry matches. The two `Add` commits fill sections the two structural
commits put there, and they wait until both are in place.

**A set of fields filled in one configuration file is ordered inside itself.** The sequence
above ranks them all equally, because every one of them is an addition. Order them by what they
depend on instead: the identifier the rest follow from first, the fields derived from it next,
and the field nothing else decides last.

```
Fulfill name: in package.json                             the identifier
Fulfill repository:, bugs: and homepage: in package.json  derived from it
Fulfill description: in package.json                      decided on its own
```

**The derived fields travel together and the independent one does not.** Three URLs naming one
repository are a single decision written as a list, which the one-line test lets through. A
description is a judgement that could be accepted while those URLs are rejected, so it takes a
commit of its own — and a subject reaching for an umbrella over all five, such as *fulfill the
placeholders*, is the "and" in disguise that the test turns away.

**A test and the implementation it covers are not ordered by this sequence.** The implementation
is the behavior change, so the sequence would lead with it, and the red-first rule above says
otherwise. That rule governs the pair; the sequence orders whatever else the line holds.

So work that also brings the existing tests up to convention lands in three commits, with the
behavior change last of the three:

```
Tidy up the existing test for <the class>    structure
Update the test for <the class>              addition
Update <the class>                           behavior
```

The middle commit is red where it sits, and that is what it is for — the claim stands there on
its own, and the commit after it is the one that makes the claim true.

**Coherence bounds the sequence.** Where this order would leave a commit referring to what is
not there yet, the seam is what is wrong, not the order — find the seam first, and sequence
what comes out of it.
