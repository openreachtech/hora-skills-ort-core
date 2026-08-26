# Pass B — Correctness

Where the code is likely to be wrong, independently of whether it matches the specification.

Each category below states what to look for and when it becomes a finding. Every category
appears in the report as findings, `PASS` or `N/A`.

## 1. Input handling

| Look for | Finding when |
| :-- | :-- |
| Values that reach a store, a query or an external call without being validated | An input crosses a boundary unchecked and a malformed value would be persisted or forwarded |
| Validation that admits values the code below cannot handle | The validator's accepted range is wider than what the implementation handles |
| Optional values treated as always present | An absent value would produce an unclear failure rather than the specified one |
| Numeric and boundary values (zero, negative, maximum, empty collection) | A boundary value takes a path that was clearly not intended |

## 2. Error and failure paths

| Look for | Finding when |
| :-- | :-- |
| A failure caught and discarded | A `catch` produces no error, no log and no state change — the caller cannot tell it failed |
| A failure turned into a success-shaped value | An error path returns the same shape as success, so the caller proceeds on bad data |
| The wrong error surfaced | The error returned differs from the one the specification names, or leaks internal detail outward |
| A failure path that was never given a value to return | A branch falls through and returns `undefined` implicitly |
| An `await` missing on a call that can reject | A rejection becomes unhandled and the failure surfaces somewhere unrelated |

## 3. Boundaries: transactions and side effects

| Look for | Finding when |
| :-- | :-- |
| Several writes that must succeed or fail together | They are not inside one transaction, and a partial failure leaves the store inconsistent |
| An irreversible side effect inside a transaction that can roll back | A mail, a payment, an external call or a queue push happens and the transaction then rolls back |
| A side effect that must survive the response | It is fired without being handed to the mechanism that guarantees it runs |
| A read used to decide a write | The read is outside the transaction that performs the write, so the decision can be stale |

## 4. Concurrency and repetition

| Look for | Finding when |
| :-- | :-- |
| Check-then-write on a value another request can change | Two concurrent requests would both pass the check |
| An operation that can be delivered twice (a retry, a re-submitted form, a redelivered job) | The second delivery produces a second effect where it should produce none |
| A counter, a balance or a status updated by reading and then writing | The update is not atomic and concurrent updates would be lost |
| Uniqueness enforced only in application code | The store permits the duplicate, so a race writes it |

## 5. Data access

| Look for | Finding when |
| :-- | :-- |
| A query inside a loop over rows | The row count is not bounded by the data model, so the query count grows with the data |
| An unbounded read (no limit, no pagination) | The result set grows with the data and nothing caps it |
| A filter applied after fetching | Rows are fetched and discarded in memory that the store could have excluded |
| A write of a value that another column already derives | The two can disagree, and nothing keeps them in step |
| A query built from a value that came from outside | The value is interpolated rather than bound as a parameter |

## 6. Resources and lifecycle

| Look for | Finding when |
| :-- | :-- |
| An acquired connection, handle, timer, subscription or lock | There is a path — especially a failure path — on which it is not released |
| An unbounded accumulation (a cache, a map, a list keyed by request) | Nothing removes entries, so it grows for the lifetime of the process |
| A long operation with no ceiling | An external call or a query has no timeout and can hold a resource indefinitely |

## 7. Test coverage of the acceptance criteria

This category is about whether the change's tests can detect a regression against the
specification. It is not a review of test style.

| Look for | Finding when |
| :-- | :-- |
| A criterion with no test | An acceptance criterion is satisfied by code but nothing would notice if it broke — `MEDIUM`, or `HIGH` for a criterion about a failure or a permission |
| A test that asserts less than the criterion states | The criterion names a value and the test only asserts that something was returned |
| A test that would pass against a stub of the thing it tests | The assertion does not depend on the behavior under test |
| Failure cases tested only for "it throws" | The specification names which failure, and the test does not distinguish it |
| A test whose expectation contradicts the specification | Report as a `BLOCKER` — either the test or the implementation encodes the wrong behavior, and one of them is shipping |

## 8. Change safety

| Look for | Finding when |
| :-- | :-- |
| A changed shared function, constant or type | A caller outside the feature relies on the old behavior and was not updated |
| A changed response shape, error code or field name | An existing client would break and no compatibility path exists |
| A schema change | It is not reversible, or existing rows do not satisfy the new constraint |
| A removed export, route or operation | Something still references it |

Trace at least one caller for every shared thing the change touches. A change reviewed only
within its own files cannot see the failure it causes elsewhere.
