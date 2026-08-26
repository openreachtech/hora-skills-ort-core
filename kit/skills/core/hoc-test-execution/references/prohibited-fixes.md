# Prohibited Fixes

Each move below makes a red suite green while destroying what the suite was for. For each one:
what it hides, and the correct handling of the situation that tempts it.

None of these is acceptable "temporarily". A temporary silencing that is never revisited is the same
as a permanent one, and in practice it never gets revisited.

## Skipping, disabling or deleting a test

**Hides:** the entire behavior the test covered. Nothing will report it again.

**Tempting situation:** the failure is hard, and the rest of the suite is green.

**Correct handling:** classify the failure and fix what the classification names. If it genuinely
cannot be fixed now, the test stays failing and the report says so, with the classification and
what is needed. A visible red test is information; a skipped test is a lie the next reader
believes.

The only legitimate removal of a test is the removal of the requirement it covers — recorded as a
withdrawal in the requirement document, not decided on the spot just because the test is
inconvenient.

## Loosening an assertion

**Hides:** the difference between the value that was required and the value the code produces.

Examples: asserting that something is truthy where a value was asserted; dropping a field from
an expected object; replacing an exact match with a pattern that matches anything; asserting a
count instead of the contents; asserting "it threw" instead of which error.

**Tempting situation:** the received value is *nearly* right, and the difference looks
incidental.

**Correct handling:** the difference is the finding. Either the code produces the wrong value — an
implementation defect — or the acceptance criterion says the expected value is wrong, in which case
the expectation is corrected to the criterion's value, exactly, and the criterion id goes in the
report. Never widen an assertion; it is corrected to a different exact statement or it stands.

## Pasting the received value into the expectation

**Hides:** everything. The test now asserts that the code does what the code does.

**Tempting situation:** the diff is large and comparing it by hand is tedious.

**Correct handling:** read the diff field by field and decide, per field, which side is wrong. If
the received value is correct against the criteria, updating the expectation to it is legitimate
— but only after that judgment, and the report names the criterion. Copying first and judging
never is the failure.

## Widening a timeout, adding a wait, adding a retry

**Hides:** a race, a missing await, an unbounded operation, or a genuine performance regression.

**Tempting situation:** it passes on a fast machine and fails on a slow one.

**Correct handling:** find what is not being waited for. A test should wait for the **condition**
it needs — a record present, a status changed, a queue drained — not for an amount of time. A
duration that "usually works" fails under load, and it fails in the run nobody is watching.

A retry is worse than a wait: it makes an intermittent defect invisible while leaving it in the
product.

## Editing fixture or seed data to match the behavior

**Hides:** the defect, by moving the goalposts to where the ball landed.

**Tempting situation:** the code returns a different record than expected, and the fixture is
easier to change than the query.

**Correct handling:** decide what the fixture is *for*. Fixture data exists to put the system in
the state the acceptance criterion names. If it does not represent that state, fixing it is
correct — and the fix is derived from the criterion, not from the code's output. If it does
represent that state, the code is wrong.

Signals that the fixture is being bent rather than fixed: the change makes the data less
realistic; the change removes the case that made the test interesting; the change is to the one
record the failing test reads and to nothing else.

## Mocking or stubbing the thing under test

**Hides:** the behavior entirely. The test now checks the mock.

**Tempting situation:** the real collaborator is awkward to set up, and stubbing it makes the
failure disappear.

**Correct handling:** stub what the test is *not* about — external services, clocks, randomness,
transport. Never stub the unit under test, and never stub a collaborator in a way that removes
the path the assertion is about. If the setup is genuinely too awkward, that awkwardness is a
design finding worth reporting, not a license to stub past it.

## Adding a branch to the implementation for the test

**Hides:** nothing at first — it ships a behavior that exists only because a test asked for it.

Examples: a check for a value that only the fixture uses; a branch on an environment flag that
makes the tested path behave differently under test; a special case for the exact input the test
supplies.

**Tempting situation:** the general fix is large and the test only exercises one input.

**Correct handling:** implement the behavior the criterion states, for every input in its range.
A special case that satisfies the test and nothing else means the product does not have the
behavior — only the test run does.

## Marking a test as expected-to-fail

**Hides:** the failure, and additionally inverts the signal — when the defect is fixed, the suite
goes red.

**Correct handling:** leave it failing and report it. A failing test with a classification in the
report is exactly as visible as it should be.

## Changing the run command to exclude the failure

**Hides:** the test, the file, or the whole suite it lived in — usually without anyone noticing
that the count dropped.

Examples: narrowing the test path in the script, adding an ignore pattern, running only the
files that were touched and reporting that as the suite.

**Correct handling:** the suite the project defines is the suite that must pass. Running a subset
while working is fine; **reporting a subset as the run is not**. The report names the command
that was executed, so a narrowed command is visible in it — which is why the report format
requires the command verbatim.

## The one legitimate change to a passing bar

Raising the bar is always allowed: making an assertion stricter, adding a case, adding a test for
a criterion that had none. If a change to a test makes the suite able to detect *more*, it is not
one of these moves.
