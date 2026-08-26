# Document Template

The template for `docs/features/<feature-slug>/progress.md`, and the rules for each field.

## Template

````markdown
# <Feature name> — Progress

| Field | Value |
| :-- | :-- |
| Feature slug | `<feature-slug>` |
| Requirements | `./requirements.md` |
| Done | 0 / 0 (0%) |
| In progress | — |
| Last updated | YYYY-MM-DD |

## Status

| ID | Requirement | Status | Evidence |
| :-- | :-- | :-- | :-- |
| REQ-01 | <short title, copied from the requirement document> | TODO | — |

## Blockers

| ID | Blocked by | What would unblock it |
| :-- | :-- | :-- |
| REQ-nn | <the blocker> | <the action or decision needed> |

## Change log

| Date | ID | Change | Evidence |
| :-- | :-- | :-- | :-- |
| YYYY-MM-DD | REQ-01 | TODO → WIP | — |
````

## Field rules

### Header

- `Requirements` points at the requirement definition document in the same directory. It is a
  relative path within the feature directory, so it survives the directory being moved.
- `Done` is `<count of DONE> / <count of requirements that are not WITHDRAWN>` with the
  percentage in parentheses, rounded down. Recomputed on every edit — a stale header is the one
  thing a reader will trust without checking.
- `In progress` names the single `WIP` requirement id, or `—`.

### Status table

- One row per requirement id in the requirement document, including `WITHDRAWN` ones. No other
  rows.
- The order follows the requirement document, and never changes.
- `Requirement` repeats the short title so the table is readable on its own. If the title
  changes upstream, update it here.
- `Evidence` holds the evidence for the current status, in the form given by the status rules —
  a test path, a command and its result, or `—` for statuses that require none.
- A row whose evidence does not fit in the cell keeps a one-line summary in the cell and the
  detail in the change log.

### Blockers table

- Only rows currently at `BLOCKED`. Resolved blockers move to the change log and leave the
  table.
- `Blocked by` uses a requirement id when the blocker is another requirement in this feature.

### Change log

- One row per status transition, in chronological order. This is the audit trail: the status
  table says where things stand, the change log says how they got there.
- The `Evidence` column repeats what justified the transition. For `REVIEW → DONE`, list every
  acceptance criterion id and what verified it.

## Worked example

````markdown
# Account Sign-in — Progress

| Field | Value |
| :-- | :-- |
| Feature slug | `account-sign-in` |
| Requirements | `./requirements.md` |
| Done | 1 / 3 (33%) |
| In progress | REQ-02 |
| Last updated | 2026-08-02 |

## Status

| ID | Requirement | Status | Evidence |
| :-- | :-- | :-- | :-- |
| REQ-01 | Sign-in with email and password | DONE | tests/__tests__/server/graphql/resolvers/customer/actual/mutations/SignInMutationResolver.js |
| REQ-02 | Failed attempts are limited | WIP | — |
| REQ-03 | Sign-in response time | BLOCKED | — |

## Blockers

| ID | Blocked by | What would unblock it |
| :-- | :-- | :-- |
| REQ-03 | No environment to run 100 concurrent requests against | The load-test environment, or the requester accepting a single-request measurement |

## Change log

| Date | ID | Change | Evidence |
| :-- | :-- | :-- | :-- |
| 2026-08-02 | REQ-01 | TODO → WIP | — |
| 2026-08-02 | REQ-01 | WIP → REVIEW | SignInMutationResolver.js, SignInInputValidator.js |
| 2026-08-02 | REQ-01 | REVIEW → WIP | AC3 failed — an unregistered address returned `account-not-found`, not `invalid-credentials` |
| 2026-08-02 | REQ-01 | WIP → REVIEW | The unknown-address path now returns the same error |
| 2026-08-02 | REQ-01 | REVIEW → DONE | AC1, AC2, AC3 each covered by a case in SignInMutationResolver.js; suite passed |
| 2026-08-02 | REQ-02 | TODO → WIP | — |
| 2026-08-02 | REQ-03 | TODO → BLOCKED | No load-test environment available |
````
