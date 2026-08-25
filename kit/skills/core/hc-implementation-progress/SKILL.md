---
name: hc-implementation-progress
description: >
  Make the state of an in-flight implementation visible, in a progress document anchored to the
  requirement ids of the feature being built — a status per requirement, advanced only against
  recorded evidence, mirrored by the session's todo list. Use this skill whenever implementing a
  defined feature spans more than one work item or session, or whenever the user asks how far
  along the work is or what is blocked.
---

# Implementation Progress

A skill for keeping the progress of an implementation **visible and truthful** while the
implementation is happening.

Progress is tracked in two places at once, and they serve different purposes.

| Where | Lifetime | Purpose |
| :-- | :-- | :-- |
| `docs/features/<feature-slug>/progress.md` | Survives sessions, reviewed in git | The record of what is done and on what evidence |
| The session's todo list | The current session only | The working slice — what is being touched right now |

The document is the source of truth. The todo list is a view of the part of it that is
currently active.

## Core principle: progress is measured from evidence, not from effort

A status describes **what has been verified**, never how much work has gone in. "Nearly done"
is not a state; either the acceptance criteria hold and there is something that shows it, or
they do not.

- Advancing a status requires a recorded piece of evidence — a passing test, a command output,
  a file path, a screen that was exercised.
- Effort spent, code written and lines changed are not evidence and never move a status.
- **A status that was advanced without evidence is a false report.** Everything downstream —
  the review phase, the release decision — is made from this document.

## Status set

| Status | Meaning | Entered when |
| :-- | :-- | :-- |
| `TODO` | Not started | The requirement is in the approved document and nothing has been written for it |
| `WIP` | Being implemented now | Work on it has actually begun in this session |
| `BLOCKED` | Cannot proceed | Something outside the current work prevents it; the blocker is recorded |
| `REVIEW` | Implemented, not yet verified | The code exists but its acceptance criteria have not all been checked |
| `DONE` | Implemented and verified | Every acceptance criterion of the requirement has been checked and the evidence is recorded |
| `WITHDRAWN` | No longer required | The requirement was withdrawn upstream; kept so the id is never reused |

- The full transition rules, including what counts as evidence for each transition, are in
  [status-rules.md](./references/status-rules.md).
- `DONE` is the only status that counts as complete. `REVIEW` is not "basically done" — it is
  the status of code nobody has verified yet.

## One requirement in progress at a time

- At most one requirement is `WIP`. Starting a second means the first goes back to `TODO` or
  forward to `REVIEW` — not silently left half-open.
- The exception is a requirement that cannot be finished without another one; in that case the
  dependent one is `BLOCKED` with the blocking id recorded, not `WIP`.
- The reason is not tidiness: several simultaneous `WIP` entries make the document unable to
  answer "what would be lost if this stopped right now", which is the question it exists for.

## Update at the moment the status changes

- The document is edited **when the status changes**, not at the end of the task, not at the end
  of the session.
- The cost of the rule is one small edit; the cost of breaking it is a session that ends with
  work that nothing records.
- A batch of status changes written at the end is written from memory, and memory is exactly
  what this document replaces.

## Keeping the todo list in step

- One todo item per requirement that is `WIP` or `BLOCKED`, named with the requirement id first:
  `REQ-02 — failed attempt limit`.
- Work that is not a requirement (setting up a fixture, reading a library) may be a todo item,
  but never becomes a row in the progress document. The document's rows are requirement ids and
  nothing else.
- When a todo item completes, update the document first, then the todo list. In that order the
  two can never disagree in the direction that matters.

## Reporting progress to the user

When asked how far along the work is, answer from the document, not from recollection. Read the
file, then give:

1. One line — `REQ done / total (percent)`, plus the count blocked if any.
2. The status table.
3. What is happening right now, and what is blocking anything blocked.

Write the document, and this summary, in the language the reader is using, as the documentation
convention requires of any document generated for a reader.

```
Progress: 7/12 done (58%), 1 blocked

| ID | Requirement | Status | Evidence |
| :-- | :-- | :-- | :-- |
| REQ-01 | Sign-in with email and password | DONE | tests/__tests__/.../SignInMutationResolver.js |
| REQ-02 | Failed attempts are limited | WIP | — |
| REQ-03 | Sign-in response time | BLOCKED | needs the load-test environment |
```

- The percentage counts `DONE` over the total of requirements that are not `WITHDRAWN`.
- **Never round a percentage upward and never report a partially-implemented requirement as
  done.** A requirement is one unit; there is no half.
- If the document does not exist yet for a feature that is already being implemented, say so
  rather than reconstructing a plausible history — then create it from the requirement
  document's ids, with every unverified requirement at `REVIEW` or `TODO`.

## Resuming in a later session

1. Read `docs/features/<feature-slug>/progress.md` before touching any code.
2. Re-verify anything at `WIP` — its state is unknown, because the session that set it ended
   without closing it. Treat `WIP` on entry as "unknown, inspect it".
3. Re-create the todo list from the `WIP` and `BLOCKED` rows.
4. Only then continue.

Never reconstruct progress by reading the diff or the commit log. Those show what changed, not
what was verified.

## Anti-patterns

| Anti-pattern | Why it fails | Instead |
| :-- | :-- | :-- |
| Marking `DONE` when the code is written | Written is not verified; the review phase then finds "finished" work that does not work | `REVIEW` until every acceptance criterion is checked |
| Marking `DONE` on a manual check because the requirement's test is failing | The red test is part of the unfinished work; the row then reads green while the suite reads red | Fix the test, re-run it, then advance the row |
| Percentage estimated by feel ("about 80%") | Cannot be checked, and is always optimistic | Count `DONE` rows over total rows |
| A progress row that is not a requirement id | Nothing downstream can point at it, and the total stops meaning anything | Requirement ids only; other work lives in the todo list |
| Updating the document at the end of the session | The one session that ends unexpectedly loses everything | Update at each status change |
| Several requirements left `WIP` at once | The document can no longer say what is actually in flight | One `WIP`; the rest go back to `TODO` or on to `REVIEW` |
| Deleting a withdrawn requirement's row | Its id looks free and gets reused, invalidating older references | `WITHDRAWN` with a reason; the row stays |
| Answering "how far along?" from memory | The answer drifts from the record, and the record is what others read | Read the file, then answer |

## Detail files

- [status-rules.md](./references/status-rules.md) — the transition rules, what counts as
  evidence for each transition, and how to handle blockers.
- [document-template.md](./references/document-template.md) — the progress document template
  and the rules for each field.
