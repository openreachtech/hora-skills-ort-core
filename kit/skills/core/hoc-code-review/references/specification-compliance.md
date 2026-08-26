# Pass A — Specification Compliance

Judging the change against the requirement definition document. This pass answers two
questions: **is everything that was specified there**, and **is anything there that was not
specified**.

## Procedure

1. List every requirement id in the requirement document, including withdrawn ones.
2. For each requirement, list its acceptance criteria ids.
3. For each criterion, find the code path that produces the outcome it names, and read it from
   the entry point to the effect. Record the path as the evidence for the verdict.
4. Assign the requirement's verdict from its criteria, using the rules below.
5. Walk the change once more from the other direction: for each changed file, name the
   requirement it serves. Anything that serves none becomes an `UNREQUESTED` finding.

## Judging a criterion

A criterion is satisfied when the code path that produces its outcome has been read and does
produce it — for the state the criterion names, not for the happy path in general.

| Situation | Verdict for the criterion |
| :-- | :-- |
| The path exists and produces the stated outcome | Satisfied |
| The path exists but produces a different outcome (different error, different field, different record) | Not satisfied — this is the most common real finding |
| The path exists only for the success case; the criterion names a failure case | Not satisfied |
| The outcome depends on configuration that is not set | Not satisfied; name the missing configuration |
| The outcome can only be judged by running the system (timing, volume, concurrency at scale) | `UNVERIFIABLE`; state what would be needed |

- **A test that asserts the criterion is evidence, but not a substitute for reading the path.**
  A test can assert against a stub and pass while the real path is wrong.
- **A criterion satisfied only under an input the code never receives is not satisfied.** Check
  that the validation layer in front of the path actually admits the input the criterion names.

## From criteria to the requirement verdict

| Criteria state | Requirement verdict |
| :-- | :-- |
| All satisfied | `IMPLEMENTED` |
| Some satisfied, some not | `PARTIAL` — list the unsatisfied criterion ids |
| None satisfied and no code addresses it | `MISSING` |
| The requirement document marks it withdrawn | `WITHDRAWN` |
| At least one criterion is unverifiable and the rest are satisfied | `UNVERIFIABLE` — say which criterion and what is needed |

Never round `PARTIAL` up to `IMPLEMENTED` because the remaining criterion is small. The size of
the gap belongs in the severity of the finding, not in the verdict.

## Unrequested code

Code in the change that serves no requirement is reported, with one of these classifications.

| Classification | Example | Severity guidance |
| :-- | :-- | :-- |
| Unrequested behavior | An endpoint, field, flag or side effect nobody asked for | `MEDIUM`, or `HIGH` when it is reachable from outside and changes data |
| Scope creep into unrelated code | A refactor of a module the feature does not touch | `MEDIUM` — it widens the blast radius of the change |
| Speculative generality | An abstraction, option or hook with exactly one caller and no stated need | `LOW` |
| Leftovers | Debug output, commented-out code, an unused import or export, a file nothing references | `LOW` |

Two things are **not** unrequested code, and must not be reported as such:

- Anything the requirement document's out-of-scope section names as handled elsewhere.
- The mechanics an implementation needs to exist at all — the migration behind a stored value,
  the validator in front of an input, the type declaration for a contract. These serve the
  requirement even though no criterion names them.

## Requirements the change was not supposed to touch

A change frequently alters behavior that an *earlier* requirement already fixed. Check the
requirement documents of the feature being changed for requirements marked done that this diff
could break — a changed shared validator, a changed error code, a changed default.

- A change that breaks a previously satisfied requirement is a `BLOCKER`, regardless of how
  small the diff is.
- Name the earlier requirement id in the finding, so it is clear this is a regression against a
  specification and not a matter of preference.

## Reporting this pass

The requirement table comes first in the report and has one row per id.

```
| ID | Requirement | Verdict | Justification |
| :-- | :-- | :-- | :-- |
| REQ-01 | Sign-in with email and password | IMPLEMENTED | AC1–AC3 each traced from the mutation entry point to the token issue |
| REQ-02 | Failed attempts are limited | PARTIAL | AC1, AC3 satisfied; AC2 not — the lock never expires, see finding 1 |
| REQ-03 | Sign-in response time | UNVERIFIABLE | Needs a 100-concurrent-request measurement; no load environment |
```

Each `PARTIAL`, `MISSING` and `UNREQUESTED` row has a corresponding finding in the findings
section. A verdict without a finding to explain it is not actionable.
