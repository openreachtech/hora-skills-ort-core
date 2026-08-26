# Failure Triage

Classifying a failure before changing anything. Triage is the step that decides **what** gets
changed; skipping it is how a test ends up edited to match a defect.

## The four classes

| Class | The failure means | What changes |
| :-- | :-- | :-- |
| Implementation defect | The behavior is wrong | The implementation |
| Test defect | The statement of required behavior is wrong | The test — only against a written acceptance criterion |
| Environment or fixture problem | Neither is wrong; the run was not set up as the test requires | The setup, the fixture or the command |
| Order-dependent or racing | The test passes and fails depending on what ran before it or on timing | Whichever of the two owns the shared state or the missing wait |

**Default to implementation defect** — leave it only with the evidence each other class requires
below.

## Confirming each class

### Implementation defect

Symptoms: the expected-versus-received diff shows the code produced a different value, a
different error or no effect at all; the test is unchanged in this diff; the code path in the
stack is code the current work touched.

Confirm by tracing the path from the test's entry point to the value it asserts, and finding the
step where the value diverges from the requirement. **Naming the step is the confirmation.** A
failure attributed to the implementation without a named step is a guess.

### Test defect

Symptoms: the implementation produces what the acceptance criterion states, and the test asserts
something else.

Confirm by quoting the acceptance criterion. There is no other confirmation.

| Situation | Class |
| :-- | :-- |
| The criterion says X, the test asserts Y, the code does X | Test defect — fix the test, cite the criterion id |
| The criterion says X, the test asserts X, the code does Y | Implementation defect |
| No criterion covers the point | **Not a test defect.** The specification is silent; get it decided before changing either side |
| The criterion is ambiguous between the two readings | Not a test defect. Raise the ambiguity as an open question against the requirement |

A test defect also covers tests that are wrong in a way unrelated to the expectation: a fixture
built inconsistently inside the test, an assertion on the wrong subject, an `await` missing so
the assertion runs before the effect. These are fixed in the test, and the report says what was
wrong with it.

### Environment or fixture problem

Symptoms: the failure names a connection, a missing table or column, a missing configuration
value, a missing file, an authentication error, or a value that is absent rather than wrong;
many unrelated tests fail together in the same way; the same test passes on another machine or
after a setup step.

Confirm by finding the missing prerequisite: a migration not applied, a seed set not loaded, an
environment file not present, a service not running, a stale build artefact, a dependency not
installed after a lockfile change.

- Fix the setup, not the test. A test changed to tolerate a missing prerequisite stops checking
  the thing the prerequisite is for.
- If the prerequisite is undocumented, that is itself a finding worth reporting — the next
  person will lose the same time.
- If the fixture data is genuinely wrong for the test's purpose, fixing the fixture is
  legitimate. Editing the fixture so the assertion matches the code's current output is not; see
  the prohibited-fixes reference.

### Order-dependent or racing

Symptoms: the test passes alone and fails in the suite, or the reverse; the failure moves
between runs; the failure involves a value from a previous test's data; the failure mentions a
timeout; the run order changed and the failures changed with it.

Confirm by running the test alone, then running it after the suite that precedes it. Two runs
with different outcomes confirm the class.

Then find the actual owner of the problem:

| Cause | Where the fix belongs |
| :-- | :-- |
| A test leaves data behind that another test reads | The test that leaves it — clean up what it created |
| Two tests write the same record, row or key | The tests — give each its own identifiers |
| A test depends on running after another | The dependency itself — make each test set up what it needs |
| The assertion runs before the effect completes | The test — wait for the actual condition, not for a set amount of time |
| The implementation races with itself | The implementation — this is a real defect that the suite happened to expose |

**A test that fails intermittently has found something.** The only wrong response is to re-run
until it passes.

## Multiple failures at once

- Look for the shared cause first. Twenty failures with the same message are one problem, and
  fixing it clears all twenty.
- Fix in dependency order: environment problems first (they mask everything), then the
  implementation defect that the most tests point at, then the rest one at a time.
- Do not open twenty investigations. Classify all of them, group them, then work one group.

## Recording the triage

For each failure, record before changing anything:

```
- Test: <the test's full name>
- Output: <the assertion that failed, expected vs received>
- Class: <implementation defect | test defect | environment | order-dependent>
- Confirmation: <the diverging step, the criterion id, the missing prerequisite, or the two runs>
```

The confirmation line is what makes the change afterwards defensible. A change made without it
looks, later, exactly like editing the test until it passed.
