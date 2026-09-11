---
name: hoc-workflows
description: "Development workflow procedural rules. Defines how to proceed with implementation, the steps that must always be performed before committing and before completion, and how to establish that a tool reporting nothing was actually measuring. Use when starting an implementation, before a commit, and before calling work complete."
---

# Workflows

Procedural rules related to the development workflow.

## How to proceed with implementation

- Before proceeding with an implementation, consult the skills it requires.
- Follow this order.

1. Design the class composition of the feature as a whole
2. Design the member composition of each class
3. Write the tests
4. Implement the class members
5. Commit the tests one class at a time
6. Commit per class

## Before committing

- Before committing, pass `npx eslint <path> <path> …`, naming the files the commit touches.
  Every commit leaves the tree lint-clean. Lint the files, not the repository — `npm run lint`
  walks the whole tree and is too slow to sit in front of every commit.
- **`npm test` is not a gate on every commit.** Step 5 commits a class's tests before step 6
  commits its implementation, so that commit fails the suite when it is checked out on its own.
  That failure is the assertion the test commit makes. Staging the implementation alongside the
  tests to keep the suite green destroys both the assertion and the split.
- Run the commit's own tests all the same — `npm test -- <path> <path> …` — and **read the
  failure**: it must fail on the behavior the tests assert, not on a typo, a bad import or a
  missing fixture. A test commit that is red for the wrong reason is a defect; a test commit
  that is green asserts nothing. Note that jest reads those arguments as **regular expressions**
  matched against the whole test path, not as literal paths, so a pattern selects every file it
  matches — unlike the eslint arguments above, which are paths.
- Follow the git commit convention before writing a commit message, and before deciding how to split working-tree changes into commits. It resolves which message format the project uses and defines what belongs in a single commit.

## Before completing implementation

- Pass `npx eslint <path> <path> …` over every file the work touched, and pass `npm test` over
  the **whole suite**, before completing the implementation. Narrowing the run to the files just
  changed here would hide the tests this work broke elsewhere. This is where the suite must be
  green — step 6 is the commit that turns the tests of step 5 green. Do not consider the
  implementation complete while either one is failing.
- The branch structure the commits land on is decided here, once the work is complete, rather
  than before it starts; see the git branch convention.

## A command that ran is not a command that worked

**A zero comes out of two states: nothing was wrong, and nothing was measured.** The output reads
the same either way — no findings, no failures, a clean exit — and every gate above is read off
exactly that.

**So before believing a zero, make the tool report a failure it cannot miss.** Plant a defect of
the kind it exists to catch, confirm it appears, and take it back out. A count that cannot be made
to move is not a count.

Measured on a type-checker invoked through a variable holding its flags: the run reported no errors
before a refactor and none after it, and the conclusion drawn was that the refactor had cost
nothing. The tool had not run at all. What exposed it was an earlier recorded run over the same
files at 435 errors — **a zero that contradicts a number somebody wrote down is the only kind that
announces itself.** Re-measured with the flags written out, the same comparison came back 435
against 431.

- **A variable holding a command and its flags is one command name, not a command with
  arguments.** A shell that does not word-split an unquoted parameter looks for a program whose
  name contains the spaces and the dashes; measured side by side, the identical script ran the
  command under one shell and reported `command not found` under another. **A variable may hold a
  path or a filename. It may not hold the command.**
  - Wrapped in `> file 2>&1 || true`, that failure lands in the output file and the exit status is
    discarded, so the `grep -c` that reads the file afterwards returns a clean zero. **The error
    message is inside the file being counted and does not match what is being counted for.**
- **Reaching the end of a script is not evidence that its steps succeeded.** An abort-on-error
  setting is not a substitute for looking: a command joined by `||`, one inside a condition, one
  on the left of `&&` are all outside its reach, and a `|| true` written to keep a step quiet
  removes it on purpose. **Check the end state the steps were supposed to produce**, not the fact
  that the script finished.
- **This is the general form of rules stated elsewhere in narrower terms** — that a test is run
  against the unchanged code first, and that progress is recorded from evidence rather than from
  effort. Both are this check applied to one instrument.
