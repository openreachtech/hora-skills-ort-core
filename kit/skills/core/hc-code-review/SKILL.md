---
name: hc-code-review
description: >
  Run a READ-ONLY, code-level review of a change and produce a findings report — specification
  compliance against the feature's requirement definition document, correctness, and conformance
  to the project's coding conventions. Never fixes anything. Use this skill whenever the user asks
  to review a change or pull request before it is merged, or to verify that what was built matches
  what was specified.
---

# Code Review

A **read-only, code-level review** of an implementation. The goal is to report where the code
does not match what was specified, where it is likely to be wrong, and where it departs from
the project's conventions — **not to fix any of it**.

The review is written to be run against a change that has a **requirement definition document**
behind it. Without that document the first pass cannot be run; see
[Reviewing without a requirement document](#reviewing-without-a-requirement-document).

## Rules (read first)

1. **Read-only. Never edit, never fix.** Inspect, run non-mutating commands, report. State the
   fix per finding as text; do not apply it.
2. **Every requirement id appears in the report** with a verdict, and every check category
   appears as findings, `PASS` or `N/A` with a one-line reason. A gap must never look like
   "nothing to report".
3. **Leave to the linter what the linter enforces.** Run the project's lint command; report its
   result as one line. Do not turn lint-enforced formatting into findings — that noise buries
   the findings that matter. Where the project has no lint command, say so on that line; the
   formatting rules then fall to the convention pass, judged against the surrounding code.
4. **Detect, don't assume.** Read the surrounding code before judging a line. A call that looks
   wrong is often correct against a convention you have not read yet.
5. **Report the defect, not the taste.** A finding states what breaks, or which stated
   convention is violated. "I would have written this differently" is not a finding.
6. **Never claim a requirement is implemented because a file with a matching name exists.**
   Read the code path end to end, from the entry point to the effect the acceptance criterion
   describes.

## How to run

1. **Determine the target, and confirm it is not empty.** Derive the diff under review: the
   branch against its base (`git diff <base>...HEAD`, three-dot so the base's own commits are
   excluded), the staged change, or a named commit range. Resolve the base to a commit before
   using it, and **check the diff is non-empty before going further** — a mistyped base produces
   an empty diff, and every check then passes for the wrong reason. Report the target, the
   resolved base commit and the changed-file count at the top of the report.
2. **Locate the requirement document** for the feature. Search in this order and say which one
   supplied it: (a) an issue or ticket referenced by the commit messages in the range, (b) a
   path the user gave, (c) `docs/features/<feature-slug>/requirements.md` or the project's own
   specification location, (d) ask the user. Read every requirement and its acceptance criteria
   before reading any code.
3. **Profile the project.** Read `package.json` (scripts, dependencies), find the entry point,
   the configuration, the test layout and the convention set the project follows. The
   convention pass is a check against *this project's* conventions, not against generic advice.
4. **Run the three passes**, each against its detail file.
5. **Run the project's lint command** and record the result as one line — or record that the
   project has none.
6. **Print the report** in the order given under [Output](#output).

## The three passes

| Pass | Question it answers | Detail file |
| :-- | :-- | :-- |
| A. Specification compliance | Does the code do what was specified, and only that? | [specification-compliance.md](./references/specification-compliance.md) |
| B. Correctness | Where is this code likely to be wrong? | [correctness-checks.md](./references/correctness-checks.md) |
| C. Convention conformance | Where does it depart from the project's conventions — and, where the project has written none, from the baseline? | [convention-checks.md](./references/convention-checks.md) |

Run them in that order. Pass A decides what the change is *for*; passes B and C are read against
that purpose. A correctness finding in code that pass A already reported as unrequested is
worth less than the unrequested-code finding itself.

## Verdicts (pass A)

One verdict per requirement id, chosen against the acceptance criteria — never against the
requirement's title alone.

| Verdict | Meaning |
| :-- | :-- |
| `IMPLEMENTED` | Every acceptance criterion is satisfied by a code path that was read end to end |
| `PARTIAL` | Some criteria are satisfied and others are not; name which |
| `MISSING` | No code in the change satisfies the requirement |
| `WITHDRAWN` | The requirement document withdrew it; no code expected |
| `UNVERIFIABLE` | The criterion cannot be judged from the code (an environment or measurement is required); say what would be needed |

Code in the change that satisfies no requirement is not a verdict — it is reported as an
`UNREQUESTED` finding in pass A, because unrequested behavior is the failure mode a
specification-compliance review exists to catch.

## Finding format

```
### [BLOCKER|HIGH|MEDIUM|LOW|INFO] <short title>   (pass: <A|B|C> / <category>)
- Requirement: <REQ-nn, REQ-nn-ACn — or "none" for unrequested code>
- Location: <file:line, or a scope when the finding is structural>
- Evidence: <the snippet or command output that shows it>
- Why: <what breaks, or which convention it departs from>
- Recommendation: <what to change — DO NOT apply it>
```

Severity is judged by consequence, not by how much code it touches.

| Severity | Meaning |
| :-- | :-- |
| `BLOCKER` | A requirement is not met, or the change breaks something that worked. Must not merge |
| `HIGH` | Wrong under a realistic input or state — data loss, a silently swallowed failure, a broken boundary |
| `MEDIUM` | Wrong under an unusual but reachable state, or a convention departure that will mislead the next reader |
| `LOW` | A hardening or clarity gap with no reachable failure |
| `INFO` | Worth knowing; not a defect |

## Output

End the review with, in this order:

1. **Target** — the diff reviewed and the requirement document read.
2. **Requirement table** — every requirement id → verdict → one line of justification.
3. **Check table** — every pass B and pass C category → findings count, `PASS` or `N/A`. Pass C
   always includes its baseline row, so this pass can never be reported as `N/A` in full.
4. **Findings** — most severe first, in the format above.
5. **Counts by severity**, and the lint result on one line.

Write the report in the language the reader is using, as the documentation convention requires of
any document generated for a reader.
6. **A one-line statement that nothing was modified.**

If any `BLOCKER` or any `PARTIAL` / `MISSING` verdict exists, say plainly that the change does
not yet satisfy its specification. Do not soften it, and do not bury it under the passing rows.

## Reviewing without a requirement document

- Say so at the top of the report. Pass A is reported as `N/A — no requirement document found`.
- Do **not** invent the requirements from the code and then review the code against them. That
  produces a report that always passes.
- Run passes B and C, and offer to define the requirements first — the specification
  compliance pass is what makes the review a verification rather than an opinion.

## Anti-patterns

| Anti-pattern | Why it fails | Instead |
| :-- | :-- | :-- |
| Fixing what you find while reviewing | The requester loses the decision, and the report no longer describes the code | Report; let the implementer fix |
| Judging a requirement from a file name or a function name | Names lie; the effect the criterion names may not happen | Read the path from entry point to effect |
| Reporting formatting the linter already enforces | Buries real findings under noise | One line for the lint result |
| Listing only problems | The report cannot be used to decide whether to merge | Every requirement and every category gets a verdict |
| A finding without a location | Cannot be acted on | `file:line`, or a named scope |
| Grading style preferences as findings | Turns the review into a matter of taste, and gets the real findings ignored | Only convention departures that are actually written down in the project's conventions |
| Treating "the tests pass" as compliance | Tests can miss the criteria entirely | Check each acceptance criterion against a test or a read code path |

## Detail files

- [specification-compliance.md](./references/specification-compliance.md) — how to judge each
  requirement, how to detect unrequested code, and how to handle criteria that cannot be judged
  from source.
- [correctness-checks.md](./references/correctness-checks.md) — the correctness categories, what
  to look for in each, and the criteria for raising a finding.
- [convention-checks.md](./references/convention-checks.md) — the convention categories and how
  to check the change against the project's own conventions rather than generic ones.
