# Document Template

The template for `docs/features/<feature-slug>/requirements.md`, and the rules for each field.

## Template

````markdown
# <Feature name>

| Field | Value |
| :-- | :-- |
| State | DRAFT |
| Feature slug | `<feature-slug>` |
| Requester | <who asked for this> |
| Last updated | YYYY-MM-DD |

## Background

<2–5 lines: the problem being solved and who has it. What happens today without this feature.>

## Scope

<3–6 lines: what this feature does, in the requester's vocabulary. No implementation terms.>

## Out of scope

| Not included | Reason |
| :-- | :-- |
| <thing a reader would assume is included> | <deferred / not requested / handled elsewhere> |

## Requirements

### REQ-01 — <short title>

<One or two sentences stating the required outcome.>

- **REQ-01-AC1** — Given <state>, when <action>, then <observable outcome>.
- **REQ-01-AC2** — Given <state>, when <action>, then <observable outcome>.

### REQ-02 — <short title>

...

## Non-functional requirements

### REQ-nn — <short title>

<Volume, response time, retention, compatibility. Written the same way, with measurable criteria.>

- **REQ-nn-AC1** — <measurable outcome>.

## Assumptions

| # | Assumption | Why it was decided this way |
| :-- | :-- | :-- |
| A-1 | <a decision made on the requester's behalf> | <reason, and that it was deferred to us> |

## Open questions

| # | Question | Blocks | Status |
| :-- | :-- | :-- | :-- |
| Q-1 | <the unresolved point> | REQ-nn | OPEN |

## Change log

| Date | Change | Reason |
| :-- | :-- | :-- |
| YYYY-MM-DD | Initial draft | — |
````

## Field rules

### State

- `DRAFT` until the requester confirms; `APPROVED` afterwards.
- Any `OPEN` row in the open questions table forces `DRAFT`.
- Adding a requirement to an approved document returns it to `DRAFT`.

### Out of scope

- Never empty. If nothing was excluded, the scope was not probed.
- Each row names something a reader would otherwise assume is included.

### Requirements

- Heading form: `### REQ-nn — <short title>`. The title is a noun phrase naming the outcome, not
  a task.
- Ids are assigned in writing order, never renumbered, never reused.
- The body states the outcome. It must not name a class, a file or a library — the design is
  decided by the implementation phase, not here.
- Requirements that no longer apply keep their heading and gain a status line directly under it.

```markdown
### REQ-03 — Suspended accounts are refused

**Status: WITHDRAWN** — the suspension feature was canceled (2026-08-02).
```

```markdown
### REQ-04 — Sign-in token lifetime

**Status: SUPERSEDED BY REQ-11** — the lifetime became role-dependent (2026-08-02).
```

### Acceptance criteria

- At least one per requirement. Numbered `REQ-nn-AC1` upward within the requirement, never
  renumbered.
- Each states the state before, the action, and the outcome that is observable from outside.
- Cover the failure paths, not only the success path. A requirement with only a happy-path
  criterion is under-specified.
- Do not state the implementation ("the resolver calls the validator"). State what a check can
  see (the response, the stored record, the screen).

### Non-functional requirements

- Use the same id sequence as the functional requirements — the sequence is document-wide, so
  that no two requirements ever share an id.
- Only include one when it has a measurable criterion. "Fast" is not a requirement; "returns
  within 1 second at 100 concurrent requests" is.

### Assumptions

- Only for decisions the requester explicitly deferred. A decision the requester never saw is an
  open question, not an assumption.
- Each row says why the decision went the way it did, so it can be revisited.

### Open questions

- `Blocks` names what cannot be settled until the question is answered. That is one of two
  things, and never blank:

| The question is about | What `Blocks` names |
| :-- | :-- |
| A requirement that already has an id | The requirement ids — `REQ-02`, or `REQ-02, REQ-05` |
| Whether something should become a requirement at all | `a new requirement: <what it would cover>` |

- A question that blocks neither is a note, not an open question, and it does not belong in the
  document.
- A question of the second kind is the one most often dropped, because there is no id to point
  at yet. Dropping it is how unrequested work gets built: nobody decided it, so nothing recorded
  the decision, and the review phase later reports the code as unrequested.
- Status is `OPEN` or `ANSWERED: <the answer>`. Answered rows stay in the table as a record. When
  a question of the second kind is answered yes, the new requirement is added and the row
  becomes `ANSWERED: added as REQ-nn`.

### Change log

- One row per meaning-changing edit, with the date and the reason.
- Editorial edits are not logged.

## Worked example

````markdown
# Account Sign-in

| Field | Value |
| :-- | :-- |
| State | APPROVED |
| Feature slug | `account-sign-in` |
| Requester | Product team |
| Last updated | 2026-08-02 |

## Background

Accounts can be registered but there is no way to sign in, so every screen behind
authentication is unreachable. Support currently issues tokens by hand.

## Scope

An account holder signs in with the email address and password used at registration and
receives an access token that identifies them on subsequent requests. Failed attempts are
limited so that a password cannot be guessed by repetition.

## Out of scope

| Not included | Reason |
| :-- | :-- |
| Password reset | Deferred to the next release |
| Social sign-in | Not requested |
| Multi-factor authentication | Handled by a separate feature already in design |

## Requirements

### REQ-01 — Sign-in with email and password

An account holder supplies an email address and a password and, when they match a registered
account, receives an access token.

- **REQ-01-AC1** — Given a registered account, when its email and correct password are
  supplied, then an access token is returned and the account's last-sign-in time is updated to
  the time of the request.
- **REQ-01-AC2** — Given a registered account, when the password does not match, then the
  `invalid-credentials` error is returned and no token is issued.
- **REQ-01-AC3** — Given an email address that is not registered, when sign-in is attempted,
  then the `invalid-credentials` error is returned — the same error as a wrong password, so
  that registered addresses cannot be discovered.

### REQ-02 — Failed attempts are limited

Repeated failures for the same email address are refused for a period.

- **REQ-02-AC1** — Given five consecutive failures for one email address within 15 minutes,
  when a sixth attempt is made, then the `too-many-attempts` error is returned even if the
  password is correct.
- **REQ-02-AC2** — Given the lock is in effect, when 15 minutes have passed since the fifth
  failure, then a correct password succeeds again.
- **REQ-02-AC3** — Given a successful sign-in, then the failure count for that email address is
  cleared.

## Non-functional requirements

### REQ-03 — Sign-in response time

- **REQ-03-AC1** — A sign-in request returns within 1 second at 100 concurrent requests.

## Assumptions

| # | Assumption | Why it was decided this way |
| :-- | :-- | :-- |
| A-1 | The failure count is kept per email address, not per source address | The requester deferred the choice; per-address is what protects the account itself |

## Open questions

| # | Question | Blocks | Status |
| :-- | :-- | :-- | :-- |
| Q-1 | Should an administrator be able to clear a lock manually? | REQ-02 | ANSWERED: no, out of scope for this release |

## Change log

| Date | Change | Reason |
| :-- | :-- | :-- |
| 2026-08-02 | Initial draft | — |
| 2026-08-02 | Added REQ-01-AC3 | The requester confirmed unknown addresses must not be distinguishable |
````
