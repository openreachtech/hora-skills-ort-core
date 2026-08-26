# Status Rules

The conditions for every status transition, what counts as evidence, and how blockers are
handled.

## Permitted transitions

```
TODO ──▶ WIP ──▶ REVIEW ──▶ DONE
  ▲       │         │         │
  └───────┘         └──▶ WIP  │      (a criterion failed; back to implementing)
          ▲                   │
          └───────────────────┘      (re-opened: a verified requirement regressed)

any ──▶ BLOCKED ──▶ (the status it came from)
any ──▶ WITHDRAWN                    (only when withdrawn in the requirement document)
```

- `TODO → DONE` and `WIP → DONE` are not permitted. Everything passes through `REVIEW`, because
  `REVIEW → DONE` is the transition where verification happens.
- `DONE → WIP` is the **only** transition out of `DONE`, it is called re-opening, and it always
  carries a change-log entry saying what was observed. A requirement that was verified and is
  now broken is a regression, and the record must show that it happened. Silently setting a
  `DONE` row back to `TODO`, or editing its evidence, erases the regression.
- `WITHDRAWN` is only set because the requirement document withdrew the requirement. Never
  withdraw a requirement to avoid implementing it.

## Conditions per transition

| Transition | Condition | Evidence recorded |
| :-- | :-- | :-- |
| `TODO → WIP` | Work on this requirement actually starts, and no other requirement is `WIP` | — (evidence is not required to start) |
| `WIP → REVIEW` | The implementation for every acceptance criterion of the requirement exists | The paths of the files that implement it |
| `REVIEW → DONE` | Every acceptance criterion has been checked and observed to hold | Per criterion: the test that covers it, or the command and its result |
| `REVIEW → WIP` | A criterion was checked and did not hold | The criterion id and what was observed instead |
| `DONE → WIP` | A requirement that was verified no longer holds | What was observed, and which acceptance criterion it breaks |
| `any → BLOCKED` | Something outside this requirement prevents progress | What is blocking, and what would unblock it |
| `BLOCKED → previous` | The blocker is resolved | What resolved it |
| `WIP → TODO` | Work is being set aside to start something else | — (the row must not be left `WIP`) |

## What counts as evidence

Evidence must be something another person can check without asking the author.

| Counts as evidence | Does not count |
| :-- | :-- |
| The path of a test that covers the criterion, and the fact that the suite it belongs to passed | "The tests pass" with no path |
| A command and the output that shows the outcome | "I ran it and it worked" |
| A stored record, response body or screen that shows the outcome | "It should work now" |
| A migration applied and the resulting schema | The migration file having been written |

- Evidence for a `REVIEW → DONE` transition must cover **every** acceptance criterion of the
  requirement, not the requirement in general. A requirement with three criteria and evidence
  for two stays at `REVIEW`.
- Where the criterion is covered by an automated test, name the test file. Where it is not,
  record the command that was run and what it produced. A criterion verified only by reading the
  code is not verified; note it as such and leave the row at `REVIEW`.
- **A requirement whose criterion has an automated test cannot reach `DONE` while that test is
  red** — no matter what a manual check shows. The test is part of what was built: a red test
  for the requirement means the requirement is not finished, even when the behavior it is
  aimed at happens to be correct. Fix the test, re-run it, then advance the row. Reaching for a
  manual command *because* the test is failing is the loophole this rule closes; the manual path
  exists for criteria that have no test, not for criteria whose test is broken.

## Blockers

- A blocked row records three things: the blocker, what would unblock it, and who or what it
  depends on.
- A dependency on another requirement in the same feature is recorded by id: `blocked by
  REQ-04`. This is what makes the ordering of the work visible.
- A blocker that is nobody's action — an unanswered question about the requirement itself —
  belongs back in the requirement document as an open question. Record it in both places: the
  progress row is `BLOCKED`, and the requirement document carries the question.
- Never work around a blocker by narrowing the requirement. Narrowing a requirement is a change
  to the requirement document, decided by the requester.

## Handling requirements discovered during implementation

Implementation regularly turns up something the requirement document does not cover. Do not
invent a progress row for it.

| What was found | What to do |
| :-- | :-- |
| A necessary behavior nobody specified | Raise it as a new requirement in the requirement document; only then does it get a progress row |
| An existing defect, unrelated to this feature | Note it and report it; it is not part of this feature's progress |
| Work that is a means to an end (refactor, fixture, tooling) | A todo item, never a progress row — it has no acceptance criteria to verify against |

## Reading a document written by an earlier session

- `WIP` on entry means "unknown". The session that set it did not close it, so its actual state
  is whatever the code happens to be. Inspect before continuing.
- `REVIEW` on entry means the code exists and nothing has verified it. Verify it before adding
  more code on top.
- `DONE` on entry is trusted only as far as its evidence. A `DONE` row whose evidence is a test
  path is trustworthy once that test still passes; a `DONE` row with no evidence recorded is not
  a `DONE` row — set it back to `REVIEW`.
