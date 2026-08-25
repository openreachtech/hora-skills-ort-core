---
name: hc-workflows
description: "Development workflow procedural rules. Defines how to proceed with implementation and the steps that must always be performed before committing / before completion."
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
