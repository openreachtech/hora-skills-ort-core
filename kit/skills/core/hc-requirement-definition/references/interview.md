# Interview

The rules for running the question rounds, and the checklist of what to probe.

## Running a question round

1. **Read the repository before the round.** Existing models, resolvers, screens and constants
   answer many questions for free. Bring what you found into the question: "the account table
   already stores a `status` column with `active` / `suspended` — should a suspended account be
   rejected here?"
2. **Pick the questions whose answers change the shape of the work.** A question whose two
   possible answers lead to the same implementation is not worth a round.
3. **Attach the options and a recommendation.** State the realistic choices, name the one you
   would take, and say what it costs. The requester answers a concrete choice far faster than an
   open question.
4. **Say what happens if it is left unanswered.** "If we don't decide this, I will record it as
   an open question and the feature cannot be approved" makes the cost of silence visible.
5. **Write down every answer immediately**, in the requester's terms. An answer that lives only
   in the conversation is lost by the next round.

## What not to ask

- Anything discoverable from the code, the schema, the existing screens or the project
  conventions.
- Implementation choices that are the implementer's to make (which class, which directory, which
  helper). Ask about outcomes and constraints, not about structure.
- Questions whose answer you would not act on differently either way.

## Probe checklist

Walk the topics in this order. Each row states what the answer changes — skip a row only when
the answer is already fixed by the request or by the existing system.

### Purpose and boundary

| Probe | What the answer changes |
| :-- | :-- |
| What problem does this solve, for whom? | Whether the requested solution is the right one at all |
| What happens today without it? | Whether there is an existing path to extend rather than a new one to build |
| What is deliberately not included? | The out-of-scope list, which decides what the review phase must not report |
| Is this replacing something? What happens to the old path? | Whether a migration and a removal are part of the work |

### Actors and triggers

| Probe | What the answer changes |
| :-- | :-- |
| Who performs this — which role, signed in or not? | Authorization requirements and the endpoint it belongs to |
| What starts it — a user action, a schedule, an external event, another feature? | Whether this is a request-time path or a background path |
| Can it happen concurrently for the same target? | Whether uniqueness, locking or idempotency requirements are needed |
| How often, and at what volume? | Non-functional requirements, and whether the naive approach is acceptable |

### Input

| Probe | What the answer changes |
| :-- | :-- |
| What exactly is supplied, and which parts are optional? | The input contract and its validation requirements |
| What is rejected, and what does the requester expect to happen then? | Error requirements, which are the most commonly omitted ones |
| Where does each value come from — typed by a person, chosen from a list, sent by a system? | How strictly it must be validated and whether it can be trusted |

### Behaviour and output

| Probe | What the answer changes |
| :-- | :-- |
| What does the actor see or receive when it succeeds? | The observable outcome the acceptance criteria are written from |
| What must be recorded, and must the previous value be kept? | Whether history / audit requirements exist |
| What else must happen as a consequence — notification, external call, downstream update? | Side-effect requirements, and whether they are part of the same operation |
| If a consequence fails, must the whole thing fail? | Whether the side effect is inside or outside the transaction boundary |

### Failure and edge cases

| Probe | What the answer changes |
| :-- | :-- |
| What if the target does not exist, or was deleted meanwhile? | Requirements that would otherwise be discovered during implementation |
| What if it is requested twice — is the second one an error, or a no-op? | Idempotency requirements |
| What is the behavior at the boundaries — empty, one, maximum, over maximum? | Limit requirements and their acceptance criteria |
| Which failures must the actor see, and which are internal only? | Error-exposure requirements |

### Constraints

| Probe | What the answer changes |
| :-- | :-- |
| Who is allowed to see or do this, and who must be refused? | Authorization requirements, stated as their own requirements |
| How long must the data be kept, and can it be deleted? | Retention requirements |
| Is there a required response time or throughput? | Whether a non-functional requirement with a measurable criterion is needed |
| Does an existing client depend on the current behavior? | Backward-compatibility requirements and whether a versioned path is required |

### Verification

| Probe | What the answer changes |
| :-- | :-- |
| How will the requester check this is done? | The acceptance criteria, in the requester's own words |
| Which case, if it broke, would matter most? | Which criteria the test phase must cover first |
| Is there existing data this must keep working with? | Whether the criteria must be written against real data as well as fresh data |

## Turning answers into requirements

- One decision, one requirement. If an answer covers two independently checkable outcomes, it
  becomes two requirements with two ids.
- Write the requirement in the requester's vocabulary, not in the codebase's vocabulary. The
  requester must be able to confirm it without reading the implementation.
- Write the acceptance criterion in the given / when / then shape — the state before, the
  action, and the observable outcome. It does not have to use those words, but all three parts
  must be present.
- A prohibition ("an unverified account must never receive a token") is a requirement. Give it
  its own id.

## When the requester will not decide

Some questions come back as "you decide" or stay unanswered across rounds. Handle them like
this, never by guessing silently.

| Situation | What to do |
| :-- | :-- |
| The requester defers a genuinely technical choice | Decide it, and record the decision in the assumptions section with the reason |
| The requester cannot decide yet | Record it as an open question; the document stays `DRAFT` |
| The answer only matters for a rare case | Record the rare case as out-of-scope for now, with the reason |
| Deciding it would change the whole design | Stop. Say plainly that the work cannot be scoped without it |
