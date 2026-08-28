# Format: Conventional Commits

The message format used when a project resolves to `conventional`. Referenced from `SKILL.md`.

## Shape

```
<type>[(<scope>)]: <lowercase summary>
```

- **`type`** is lowercase, from the table below, and is required of every commit that carries a
  change. **A commit that carries none takes no type** — the type describes a change, and there
  is none to describe.
  - Two commits are like this, both described by the git branch convention: the empty
    branch-opening `Start …` marker, and the `Merge …` commit that closes a branch. Write
    `Start dev`, not `chore: start dev`.
  - A merge that had to resolve a conflict still takes none. What it carries is the adjustment
    joining the two lines required, not a decision of its own.
- **`scope`** is optional, lowercase, in parentheses — the area of the codebase affected.
- A **colon and a single space** separate the prefix from the summary.
- The **summary is lowercase** (unless it opens with an identifier that is itself capitalized)
  and in the **imperative mood**, with **no trailing period**.

A colon before the first space separates the type from the summary. Every other colon in a
subject belongs to the class-member notation, where it always follows `#` or `.`:
`docs: tidy up the JSDoc of BaseRestfulApiLauncher.get:ResponseBodyParser` carries one of each.
The two never collide — one opens the subject, the other sits inside a member name.

```
feat: add LockEmployeeSignInInputValidator for employee sign-in validation
fix: return 401 instead of 500 when the visa is expired
refactor: update return types in LockClientMemberSignInMutationResolver
test: add generateCommentFilesAssignments tests to client TaskCommentsQueryResolver
feat(resolver): add unlockClientMemberSignIn mutation
```

## Types

| type | use for |
| :-- | :-- |
| `feat` | a new feature added to the application or library |
| `fix` | corrected behavior |
| `refactor` | restructuring with no change in behavior |
| `test` | tests added or changed, with no production-code change |
| `docs` | documentation only |
| `chore` | build config, dependencies, tooling |
| `style` | formatting only, no code meaning changed |
| `perf` | a change made for performance |

- **A feature's internal parts take the same type as the feature.** A class only `Alpha` uses
  is still part of what `Alpha` delivers, so it is `feat`, not `chore`. Judging visibility per
  commit is the wrong moment for it — a class can become reachable later without any commit
  changing. Which feature it serves is what the branch and its merge commit say; put it in the
  summary (`feat: declare Beta for Alpha`) only where that context is missing.
- The type describes **the change**, not the file it lands in. A bug fixed inside a test
  helper is `fix`, not `test`.
- `refactor` asserts that behavior did not change. If behavior changed, it is `feat` or `fix`
  — and the two should have been separate commits in the first place. See the granularity
  detail file.
- When two types would both fit, the commit is doing two things. Split it.

## Summary

The summary follows the same substance rules as any other format: name the concrete thing that
changed, in the imperative mood, using the class-member notation given in `SKILL.md`.

**Its verb follows the shared table in `SKILL.md`** wherever that table fixes a role,
lowercased to sit after the type: `feat: declare AlphaClass`, `refactor: extract the retry
loop`. The vocabulary does not change between the two formats — only the capitalization and
what precedes it do.

The characteristic failure of this format is a **redundant trailing clause** that restates the
identifier already named:

```
Bad:  feat: add LockEmployeeSignInInputValidator for employee sign-in validation
Good: feat: add LockEmployeeSignInInputValidator

Bad:  feat: implement LockClientMemberSignInMutationResolver for locking client member sign-in accounts
Good: feat: add LockClientMemberSignInMutationResolver
```

The identifier already carries the purpose. Add a clause only when it says something the name
does not.

```
Good: fix: reject empty clientMemberId before the resolver reaches the model
```

Related: `add` and `implement` are not two different things. Use `add`.

## Breaking changes

A breaking change is marked with `!` before the colon. The `BREAKING CHANGE:` trailer that
explains it, and the body it sits under, are required of both formats — `SKILL.md` states that
rule, and this format adds the subject marker on top of it.

```
feat(resolver)!: require clientMemberId on unlockClientMemberSignIn

BREAKING CHANGE: unlockClientMemberSignIn previously resolved the member from
the visa. Callers must now pass clientMemberId explicitly.
```

- Mark the break even when it feels minor. The marker is what downstream tooling and release
  notes key on.

## Scope

- Use a scope only when the repository already uses scopes consistently. A history where a
  quarter of commits carry one is worse than a history with none.
- Scope names an **area**, not a file: `resolver`, `validator`, `model`, `job` — not
  `LockEmployeeSignInInputValidator`, which belongs in the summary.
