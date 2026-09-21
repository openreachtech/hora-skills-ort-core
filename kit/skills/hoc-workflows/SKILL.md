---
name: hoc-workflows
description: "Development workflow procedural rules. Defines how to proceed with implementation, which of the two to follow where a convention and the surrounding code disagree, and the steps that must always be performed before committing / before completion."
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

## The existing code is not a template

**A convention outranks the code already in the repository.** Where a skill states
how something is written and the files around it are written another way, the new
code follows the skill. Those files record what the convention was before it was
written down, and matching them propagates the very thing the convention exists to
stop.

- **The count does not change the answer.** A rule broken by one file and a rule
  broken by sixty are the same rule. A majority in the repository is evidence of
  when the code was written, never of what is correct.
  - **Counting the two spellings is itself the error.** A convention written after
    the code exists starts at zero occurrences of the spelling it prescribes, so the
    tally is guaranteed to argue against every rule on the day it is adopted.
    Reaching for `grep -c` to settle which spelling is right asks a question whose
    answer was fixed before the rule was written, and reading a count of 441 to 0 as
    "the repository says otherwise" inverts what the numbers mean.
  - **The first instance is not an anomaly needing a defence.** "The skill states
    it" is the whole justification, and 441 to 1 is what a convention looks like on
    its first day rather than evidence against it. The spelling that is about to be
    written is the one the count is missing.
- **Where a convention covers the point, the surroundings are not consulted.** The
  convention has already answered, and what the files around you happen to do adds
  nothing to that answer.
- **Where nothing is written, the surroundings inform and never authorize.** They
  show what has been done, which is not the same as what may be done. Code that is
  worse than what you already know how to write does not become acceptable by being
  nearby, and matching it writes the flaw a second time — leaving the next reader
  two examples of it instead of one, and a majority where there had been an
  exception. What stands in where nothing is written is the coding charter and the
  QA stance the tests are held to, never the nearest file.
  - **The code review convention says the same from the reviewer's side.** Where
    nothing is written it sends a reviewer to the baseline and the charter too, and
    it raises a departure from the surrounding code only where the departure is the
    worse of the two. Code written to this rule is therefore not a finding under
    that one.
- **Only a named instruction reverses this.** Being asked to match a particular
  file, or to leave a module's style alone, is that instruction. Being asked to add
  something to a file is not.
- **Leave the older files alone.** This governs what is being written; it is not
  licence to rewrite what is already there. Bringing existing files up to a
  convention is work of its own, and it is asked for separately.

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
