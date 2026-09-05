---
name: hoc-test-execution
description: >
  Run a project's test cases and drive them to green without weakening them — no test skipped,
  deleted, loosened, or padded with waits to make the suite pass. Use this skill whenever tests are run,
  whenever a test still fails after an implementation was believed finished, when the user asks to
  run or fix failing tests, and before claiming an implementation is complete. Writing new tests
  belongs to the project's test-writing convention; reusing a recorded pass over unchanged inputs,
  to the test-cache convention.
---

# Test Execution

A skill for **running the test cases and resolving what they report**.

The failure this skill exists to prevent is not "the tests fail". It is what usually happens next:
changing the test until it stops failing. A suite that was changed until it passed reports nothing,
and the next defect ships behind a green run.

## Core principle: a failing test is a result, not an obstacle

The test is a statement about required behavior. When it fails, exactly one of two things is true:
the behavior is wrong, or the statement is wrong. **Deciding which one — with evidence — is the
work.** Everything else is just routine detail.

- The default assumption is that the **implementation** is wrong. A test written before the
  implementation encodes what was required; the implementation is the newer, less-reviewed
  thing.
- A test's expectation may be changed **only** when a written acceptance criterion says the
  expectation is wrong. Not because the implementation disagrees with it — the implementation
  disagreeing with it is the failure.
- **Making a red test green is not the goal. Learning what the red test found is the goal.**

## The loop

```
run the whole suite
      │
      ▼
any failure? ──no──▶ report the run's own summary ──▶ done
      │yes
      ▼
take ONE failure
      │
      ▼
read its actual output (message, diff, stack, the assertion that failed)
      │
      ▼
classify it  ──────▶  see references/failure-triage.md
      │
      ▼
fix the thing the classification names — never the test unless the class is "test defect"
      │
      ▼
re-run that one test ──fails──▶ back to reading the output (do NOT try a second change on top)
      │passes
      ▼
re-run the whole suite ──▶ back to the top
```

- **One failure at a time.** Changing three things and re-running tells you nothing about which
  change did what, and routinely leaves two of them wrong.
- **Re-run the whole suite after every fix**, not only the test that was failing. A fix that
  breaks two other tests is a worse state than the one it started from, and only the full run
  shows it.
- **Never stack a second change on a failed attempt.** Revert the attempt, re-read the output,
  then change one thing.

## Before running anything

1. **Discover how this project runs its tests.** Read `package.json` scripts and the test
   configuration. Use the project's own command; do not invent one.
2. **Discover whether the project has more than one suite and whether order matters.** Many
   projects separate tests that touch no persistent state from tests that write to a store, and
   require them to run in a defined order. Running them the wrong way produces failures that
   have nothing to do with the code.
3. **Discover what the suite needs to be present** — an environment file, a database, a
   migration applied, a seed set loaded. A missing prerequisite produces failures that look like
   defects.
4. **Run the suite once before changing anything**, to establish which failures were already
   there. A pre-existing failure attributed to the current change wastes the whole session.
5. **Size the run to the machine, not to its listed specifications.** Most runners execute test
   files in **parallel workers** — one per core by default — and each worker is a full process using
   its own memory. That default assumes an idle machine; a real one is running an editor, a browser,
   a local database, and sometimes a full E2E stack, all using memory before the suite starts.
   Before a full run, account for what else is resident and how much memory and CPU are **actually
   free** — not how much is installed. **When the machine is constrained, dial the parallelism
   down** (the runner's max-workers lever, halved and re-run) **or run serially** (the runner's
   in-band / single- worker mode) rather than letting the operating system kill a worker — or
   another process — under the load. A serial run that finishes beats a parallel one that dies
   halfway. The concrete flags and the way to weigh workers against available memory belong to the
   project's test-running convention; the principle here is to **look at the machine before choosing
   the parallelism**, not after it dies.

## Read the actual output

- Read the failure message, the expected-versus-received diff, the failing assertion and the
  stack frame in the project's own code. All four.
- **Infer the cause from the output, not from the test's name.** The name says what it was supposed
  to check; the output says what happened.
- **Never act on remembered output.** If the output has scrolled away, run the single test again
  and read it.
- When the message is unclear, get more signal before changing code: run the one test in
  isolation, print the actual value, or run with the project's verbose option. Guessing costs
  more than one extra run.

## Changing a test is a specification decision

A test expectation may be changed only in these two cases, and the report must say which.

| Case | What must be true |
| :-- | :-- |
| The test contradicts a written acceptance criterion | Quote the criterion id and its text in the report. The test was wrong; the implementation may be right |
| The specification itself changed | The requirement document records the change. Until it does, the test stands |

In every other case the test stands and the implementation changes. "The implementation is
obviously right and the test is out of date" is not one of the two cases — it is the
rationalization this rule exists to stop.

## Prohibited ways to make a test pass

These are never acceptable, including "temporarily". See
[prohibited-fixes.md](./references/prohibited-fixes.md) for what to do instead in each case.

- Skipping, disabling, commenting out or deleting a failing test.
- Loosening an assertion — asserting truthiness where a value was asserted, removing a field
  from an expected object, widening a matcher until it stops discriminating.
- Widening a timeout, adding a wait, or adding a retry to make an intermittent failure stop.
- Editing fixture or seed data so that it matches the behavior the implementation happens to
  have.
- Mocking or stubbing the very thing under test, or stubbing a collaborator so that the failing
  path is no longer exercised.
- Adding a branch to the implementation that exists only to satisfy the test.
- Marking a test as expected-to-fail.

**Reusing a recorded pass is not one of these.** A cache that re-reports a pass for inputs identical
to the ones that produced it skips a second execution, not a test — nothing is skipped, loosened or
deleted, and the numbers reported are still the ones the runner measured. When that reuse is allowed,
and what a reused pass must be reported with, belong to `hoc-test-cache`.

**Breaking the letter of these rules breaks their spirit too.** A suite whose failures were silenced
is worse than no suite, because it is believed.

## Rationalization table

Every one of these appears when a suite will not go green. Each one means: stop and triage.

| Rationalization | Reality |
| :-- | :-- |
| "The test is out of date" | Then a written criterion says so. Quote it, or the test stands |
| "The implementation is obviously correct" | Obvious is what the test is there to check. Read the criterion |
| "It's flaky, just re-run it" | Flaky means order-dependent or racing. That is a defect, in the test or the code. Diagnose it |
| "I'll skip it and come back" | Nobody comes back. A skipped test is a deleted test with extra steps |
| "It only fails on CI" | Then the difference between the environments is the finding. Find it |
| "The assertion is too strict" | The assertion is the specification. Strictness is the feature |
| "I'll widen the timeout, the machine is slow" | Timeouts hide races. Find what is not being waited for |
| "Just this one test, the rest pass" | The rest passing is what makes the one failure informative |
| "It's a pre-existing failure, not mine" | Say so in the report, with the run that proves it. Do not silence it |
| "Nobody will notice" | The next defect behind this test will slip by too. That is the cost |

## Red flags — stop and triage

- Editing a test file before having classified the failure.
- Reaching for `.skip`, `.todo`, a longer timeout or a retry.
- Copying the received value from the failure output into the expected value.
- Making a change without having read the failure output in this session.
- Re-running "to see if it passes this time".
- Two or more edits since the last run.

**All of these mean: revert the edit, read the output, classify the failure.**

## Reporting the run

Build the report from the run's own output, not from memory.

1. **The command that was run**, exactly as executed, and which suite it covers.
2. **The run's own summary line** — the suite and test counts as the runner printed them.
   Paste it; do not paraphrase it.
3. **Per failure resolved**: the test, the classification, what was actually wrong, what was
   changed, and — when a test expectation was changed — the acceptance criterion id that
   justified it.
4. **Failures still open**, with their classification and what is needed. Never omit these.
5. **Pre-existing failures**, separated from the ones this work caused.

State that the suite passes only when the run's summary is in the report. **A claim of green without
the output is an unverified claim**, and the whole point of running the tests was to stop making
unverified claims.

## Detail files

- [failure-triage.md](./references/failure-triage.md) — the classification step: symptoms, what
  each class means, what to inspect, and how to confirm the classification before acting.
- [prohibited-fixes.md](./references/prohibited-fixes.md) — each prohibited move, what it hides,
  and the correct handling of the situation that tempts it.
