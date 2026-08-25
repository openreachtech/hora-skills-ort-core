---
name: hc-requirement-definition
description: >
  Turn a rough, incomplete request into a requirement definition document through conversation
  with the requester — identified requirements, observable acceptance criteria, an out-of-scope
  list, and the points still undecided. Use this skill whenever the user brings a feature request,
  ticket or rough idea and asks to define or write up the requirements, and before implementing
  anything whose scope is not already written down.
---

# Requirement Definition

A skill for converting a rough specification into a **requirement definition document**
through conversation with the requester.

The output is not prose that describes an idea. It is a document that a later phase can
**verify against**: every requirement carries a stable id and at least one acceptance
criterion, so that implementation progress, code review and test results can each point back
to the exact requirement they satisfy.

## Core principle: the document records decisions, it never records guesses

A rough specification is rough because the requester has not yet decided everything. The value
of this skill is in **surfacing what has not been decided**, not in producing a complete-looking
document.

- Anything the requester actually stated → a requirement.
- Anything the requester did not state, and that changes the work → **ask**.
- Anything asked but not yet answered → an **open question** in the document, never a guessed
  answer written as if it were decided.
- A document that looks finished but silently contains the author's assumptions is worse than
  an obviously unfinished one, because the verification phase will then confirm the assumption
  instead of the requirement.

**Filling a gap quietly is the failure this skill exists to prevent.**

## The skill is not finished until the requester approves

The document has two states, recorded in its header.

| State | Meaning |
| :-- | :-- |
| `DRAFT` | Written, but not yet confirmed by the requester. Implementation must not start. |
| `APPROVED` | The requester has read it and confirmed it. Implementation may start. |

- Never mark a document `APPROVED` on the requester's behalf.
- A document with any unanswered open question cannot be `APPROVED`. Either the question is
  answered, or the affected requirement is moved to out-of-scope.

## Procedure

Work through the five phases in order. Do not skip to drafting.

| # | Phase | What happens |
| :-- | :-- | :-- |
| 1 | Restate | Read everything given (message, ticket, existing code) and restate the request in your own words in 3–5 lines. Get it corrected before going further. |
| 2 | Probe | Ask the questions that change the shape of the work. See [interview.md](./references/interview.md). |
| 3 | Decide | For each answered point, write the decision down. For each unanswered point, write an open question. |
| 4 | Draft | Produce the document from the template. See [document-template.md](./references/document-template.md). |
| 5 | Approve | Present the document, ask for confirmation, then set the state to `APPROVED`. |

- Phases 2 and 3 repeat. One round of questions is almost never enough; two or three rounds
  is normal.
- Phase 1 is not optional. A restatement that gets corrected early saves the whole document
  from being written against the wrong problem.

## Rules for the conversation

- **Group questions by topic and ask a round at a time.** Roughly five questions per round is
  the upper bound. A list of twenty questions gets one answer.
- **Order questions by how much the answer changes the work.** Ask what splits the design first
  (does this write to storage at all? is it synchronous?), and ask details last (label text,
  ordering).
- **Never ask what the code already answers.** Read the repository first. Asking a requester
  something discoverable from the existing implementation wastes the round.
- **Offer a recommendation with each question.** Present the realistic options and say which one
  you would pick and why. An open-ended question returns a vague answer.
- **Ask about what must not happen**, not only about what must happen. Prohibitions are
  requirements and are the ones most often left unstated.
- **Do not negotiate the scope down.** If the request is larger than expected, record it fully
  and let the requester decide what to cut. Cutting scope silently is the same failure as
  guessing.

## Requirement ids

- Ids are `REQ-01`, `REQ-02`, … zero-padded to two digits, unique within one document.
- Ids are **assigned in the order requirements are written and never renumbered.** Progress
  entries, review findings and test names refer to these ids; renumbering invalidates all of
  them.
- A withdrawn requirement keeps its id with the status `WITHDRAWN` and a one-line reason. **Its
  id is never reused** for a different requirement.
- A requirement added after approval takes the next unused number, even if it belongs at the top
  of the document logically. Order in the document is only for presentation; the id is the
  identity.

## Every requirement must be verifiable

- Each requirement carries at least one acceptance criterion, numbered `REQ-nn-AC1`,
  `REQ-nn-AC2`, ….
- An acceptance criterion states an **observable outcome** — something a test, a screen or a
  stored record can be checked against. If nobody can tell from the outside whether it holds,
  it is not a criterion.
- A requirement whose criteria cannot be written is a requirement that is still not understood.
  Go back to phase 2 and ask, rather than writing a vague criterion.

| Not a criterion | A criterion |
| :-- | :-- |
| "Sign-in works properly." | "Given a registered email and its correct password, the response contains an access token and the account's `lastSignInAt` is updated." |
| "Handles errors." | "Given a password that does not match, the response is the `invalid-credentials` error and no token is issued." |
| "Should be fast." | "A sign-in request returns within 1 second with 100 concurrent requests." |

## The out-of-scope list is mandatory

- Every document has an out-of-scope section, and it is never empty.
- List what a reader would reasonably assume is included but is not, with a one-line reason
  (deferred / not requested / handled elsewhere).
- This section is what prevents the review phase from reporting deliberate omissions as defects.

## Where the document lives

```
docs/features/<feature-slug>/
└── requirements.md      the requirement definition document
```

- `<feature-slug>` is lower-case kebab-case naming the feature (`user-sign-in`,
  `invoice-export`).
- One feature, one directory. Later phases place their own documents in the same directory, so
  that everything about one feature is found together.
- If the project already has an established location for specifications, follow the project and
  keep the file name `requirements.md`.
- Write it in the language the reader is using, as the documentation convention requires of any document generated for a reader.

## Amending an approved document

- **Never silently rewrite an approved requirement.** Later phases have already been verified
  against the old text, and a silent edit makes a passing review meaningless.
- Editorial fixes (typo, clearer wording, same meaning) may be applied directly.
- A change in meaning follows one of these two forms, and is recorded in the document's change
  log with the date and the reason.

| Change | How to record it |
| :-- | :-- |
| A requirement is dropped | Keep the id, set its status to `WITHDRAWN`, give the reason. |
| A requirement changes meaning | Set the old id to `SUPERSEDED BY REQ-nn`, add the new text under a new id. |

- Adding a new requirement to an approved document returns the document to `DRAFT` until the
  requester approves the addition.

## Anti-patterns

| Anti-pattern | Why it fails | Instead |
| :-- | :-- | :-- |
| Writing the document before asking anything | The document becomes the author's design, and the requester reviews their own idea back to front | Restate first, probe second, draft third |
| A plausible answer written in place of an unanswered question | The verification phase confirms the guess and reports success | Record it as an open question |
| Requirements phrased as implementation ("add a `SignInResolver`") | Locks the design and cannot be verified as an outcome | Phrase the outcome; leave the design to the implementation phase |
| One giant requirement covering a whole feature | Cannot be marked partially done, cannot be reviewed piece by piece | Split until each requirement has its own acceptance criteria |
| Renumbering ids to tidy the document | Breaks every progress entry, review finding and test that refers to them | Ids are permanent; only the order of presentation may change |
| Leaving out-of-scope empty | Every omission later looks like a defect | Always list what was deliberately excluded |

## Detail files

- [interview.md](./references/interview.md) — the probe checklist (what to ask per topic and
  why it changes the work) and the rules for running a question round.
- [document-template.md](./references/document-template.md) — the document template and the
  rules for each field.
