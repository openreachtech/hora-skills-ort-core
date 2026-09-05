---
name: hoc-test-cache
description: "Reuse a recorded test pass when a verification unit's inputs are unchanged, through the mentsu-testcase-cache CLI — declaring the cache units in `.hora-cache.json`, and judging when a recorded pass may stand in for an execution. Use when wiring cached test execution into a repository, or when reading a cached run's verdict. Driving a failing suite to green belongs to the test-execution convention; recording the verdict belongs to the acceptance run."
---

# Test Cache

A skill for **reusing a recorded pass instead of executing the same inputs twice**.

What is skipped is the second execution, never a test. The suite a unit runs is its whole suite, and
the numbers reported are the runner's own.

## Core principle: the cache records a pass, it never grants one

- **Only a pass is recorded.** A failing command records nothing and executes on every run.
- **The input set is derived, not declared.** It is every file git can see in the unit's directory;
  a declaration can only *add* to it. A wrong declaration costs time — it cannot manufacture a
  green.
- **A reused pass is worth what its audit is worth.** The audit is on by default, and leaving it on
  is what makes reuse defensible.

## The unit — one directory, one command

A unit is the granularity of the cache, declared in `.hora-cache.json` at the project root. The file
is committed; the directory it names is not.

```json
{
  "cacheDir": ".hora-cache",
  "units": {
    "backend": {
      "cwd": "myproject-backend",
      "command": "npm test",
      "extras": [".env*"]
    }
  }
}
```

| field | required | meaning |
| :-- | :-- | :-- |
| `cacheDir` | no, default `.hora-cache` | where records are written. Machine-local — add it to `.gitignore` |
| `units.<name>.cwd` | yes | the directory the command runs in, relative to the project root |
| `units.<name>.command` | yes | the unit's own test command, exactly as the repository already runs it |
| `units.<name>.extras` | no | files git ignores that behavior depends on, such as env files |

- **Declare the command the repository actually runs.** The command string is part of the key, so
  `npm test` and `./test.sh` are different units even when one calls the other.
- **An ordered chain is one unit.** Suites that must run in a defined order are declared as a single
  unit and never split per suite.
- **Units run serially.** Two commands that rebuild the same database collide when run at once.

Declaration examples and the extras patterns are in [unit-declaration.md](./references/unit-declaration.md).

## Wiring a repository

```sh
npm install --save-dev @openreachtech/mentsu-testcase-cache
```

```json
{
  "scripts": {
    "test": "mentsu-testcase-cache",
    "test:cold": "mentsu-testcase-cache --cold",
    "test:status": "mentsu-testcase-cache --status",
    "test:clear": "mentsu-testcase-cache --clear"
  }
}
```

- **Run it from the project root.** The CLI reads `.hora-cache.json` from the directory it is started
  in and never searches upward, and every unit's `cwd` is resolved from there. Started anywhere else
  — a sub-package's own script, a shell that wandered — it stops at exit 2 without running anything.
- **The cache directory is machine-local and never committed.** A record carries the node version and
  platform it was made on, so another machine's key never matches it anyway.
- **A missing `.hora-cache.json` stops the CLI at exit 2.** It never guesses a unit from what the
  directory looks like.

## Reading a run

| line | meaning |
| :-- | :-- |
| `━━━ unit: <name> — hashing inputs … <n> files, key <key>` | the key was computed over that many files |
| `✘ MISS — executing` | no record for this key. The command runs, streaming its own output |
| `✔ passed in <t> — recorded as <key>` | the execution passed and was recorded |
| `✘ failed in <t> — not recorded` | the execution failed. Nothing was recorded |
| `✔ REUSED — pass recorded at <time>` | a record matched, and its summary lines are replayed |
| `━━━ cache-audit: …` | the spot audit's verdict for this run |

**Report the replayed summary lines as the run's numbers.** They are the runner's own output from the
execution that produced the record, so they are transcribed and never paraphrased.

**Only a Jest-shaped summary is recorded.** What a record keeps is the output lines beginning
`Test Suites:` or `Tests:`, and nothing else. A runner that prints its counts in another shape
records no summary lines at all, so its REUSED run replays none.

- **Report a lineless reuse as what it is** — the unit, its key, the time the pass was recorded, and
  the audit's verdict. It is a statement that the inputs are unchanged, not a count.
- **Never reconstruct the counts from an earlier run.** A number typed from memory is the
  unverified claim the whole convention exists to prevent.
- **Where the report has to carry the counts, run that unit cold** and quote the execution's own
  output.

`--status` answers what the next run would do without executing anything: the cache directory, then
one line per unit — `<unit>: <n> recorded pass(es), REUSE ready` or `… stale — would MISS`, with the
key it judged that against. Keys are printed shortened, so two keys are compared by their first
characters, never assumed equal because both were abbreviated.

## Exit codes are the contract

| code | meaning | what the caller does |
| :-- | :-- | :-- |
| 0 | every unit passed, reuse included | continue |
| 1 | a unit's execution failed | triage it as an ordinary test failure |
| 2 | config error — no config file, or an unknown unit name | fix the setup. This is not a verdict on the tests |
| 3 | audit mismatch | a defect of the measuring instrument. Trust no record |

- **Never collapse 2 or 3 into "the tests failed".** Each names a different thing to fix, and only 1
  is about the product.
- **The audit outranks the tests.** A run whose audit mismatched exits 3 even when a unit also
  failed, so exit 3 is never evidence that the other units passed. Read the per-unit summary lines
  for that.

## The spot audit

When at least one unit was reused, one reused unit is picked at random and executed for real.

- **A match backs the reuse.** The run continues, and its summary carries the audited unit and
  `match`.
- **A mismatch means the key has a hole.** A recorded pass fails when executed, so every record is
  discarded and the run exits 3.
- **A run that reused nothing is not audited.** A cold run and an all-MISS run rest on executions,
  which is what the audit was there to obtain.
- **`--no-audit` belongs to a run whose verdict nobody records.** The skip is printed, so a log
  always shows which runs went unaudited.

On a mismatch:

1. Treat it as a **cache defect, not a product defect**. The audited unit's failure is evidence about
   the key, not a finding about the code.
2. Raise it as a blocking question against the key definition, naming the unit, its key, and what the
   key could not see.
3. Execute the remaining units for real before any verdict is recorded.

## When a record may not stand in for a run

- **A release-level sweep runs cold.** `--cold` executes every unit and rebuilds the records, so the
  release verdict rests on measured passes and a wrong reuse cannot outlive the version.
- **A named unit narrows the run.** `mentsu-testcase-cache backend` states that the other units were
  not checked, so a gate run always runs every unit.
- **An install the unit's directory does not hold is invisible.** The key digests
  `node_modules/.package-lock.json` under the unit's own `cwd`; where the installed tree lives at a
  workspace root instead, that path is missing and digests the same before and after an install. Run
  cold after installing dependencies in such a layout.
- **Behavior the key cannot see voids a record** — a service the suite calls, the machine's clock, a
  globally installed binary, state a previous run left in a database.

What the key covers, and what it cannot, is in [key-composition.md](./references/key-composition.md).

## A failing unit is handed on unchanged

The cache decides nothing about a red run. Exit 1 is where this skill ends and `hoc-test-execution`
begins, with its prohibitions intact: reuse of a recorded pass over identical inputs is not one of
the ways a suite is weakened, and it is never a reason to skip, loosen or delete a test.

## Prohibited

- **Narrowing a unit's command or its extras so that a key hits.** A declaration edited toward a hit
  is a fabricated green.
- **Passing `--no-audit` in a run whose verdict is recorded.**
- **Reporting a reused pass without the key it was reused under and the audit's verdict.**
- **Reporting counts a REUSED run did not replay.**
- **Reporting exit 3 as a test failure**, or re-running until the audit picks a different unit.
- **Committing the cache directory, or copying records between machines.**

## Rationalization table

| Rationalization | Reality |
| :-- | :-- |
| "It passed five minutes ago, skip it" | Then the key matches and the tool reuses it. Deciding that by hand is what the key replaced |
| "The audit costs a whole extra unit" | The audit is what separates a reused pass from a guess. Its cost is one unit, once |
| "The mismatch is flaky, re-run it" | A mismatch says a recorded pass now fails. Find what the key cannot see |
| "This file cannot affect the tests" | Then including it costs one hash. Leaving it out costs a green nobody measured |
| "The reuse printed no counts, I remember them" | A remembered count is not a measurement. Report the reuse, or run cold |
| "Cache the failures too, it is the same mechanism" | Only a wrong green is dangerous. A wrong red re-runs and corrects itself |
| "Clear the cache and move on" | Clearing hides the mismatch that was the finding. Record it first |

## Red flags — stop

- Editing `.hora-cache.json` after seeing a MISS.
- Naming units on the command line in a run whose verdict is recorded.
- A summary line typed by hand rather than transcribed from the output.
- Exit 3 reported under the same heading as a test failure.

## Detail files

- [key-composition.md](./references/key-composition.md) — everything the key digests, what it cannot
  see, and the rule for a doubtful input.
- [unit-declaration.md](./references/unit-declaration.md) — declaring units, the extras patterns, and
  the granularity rules.
