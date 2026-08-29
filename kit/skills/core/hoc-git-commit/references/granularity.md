# Granularity (What Belongs in One Commit)

Conventions for deciding which changes are staged together. Referenced from `SKILL.md`.

This is decided **while working**, not at commit time. By the time the work is finished and
the tree holds six unrelated edits, the cheap moment to have made the decision has passed.

## Principle: one commit is one decision

A commit is the unit a reviewer accepts or rejects. It should therefore contain **exactly one
decision** — one behavior change, one rename, one new artifact — and everything required to
make that decision coherent, and nothing else.

Two consequences follow.

- A reviewer who disagrees with one decision can reject that commit **without** undoing
  unrelated work that happened to travel with it.
- A later reader bisecting for a regression lands on a commit that changes **one** thing, so
  the answer to "what broke it" is the commit itself, not a subset of it.

## The "and" test

If an accurate subject line needs the word "and", the commit is two commits.

```
Bad:  Add LockEmployeeSignInInputValidator and tidy up an unrelated JSDoc typo
Good: Add LockEmployeeSignInInputValidator
      Tidy up the JSDoc of EmployeeSignInMutationResolver#resolve()
```

**Unless the two sides it joins belong to one decision.** The word is a symptom often enough to
be worth noticing, and it settles nothing on its own.

```
Good: Add coverage, tgz and eslintcache to Regenerable output
Good: Rename environment to Regenerable output and absorb Build output
```

The first is one decision — everything confirmed regenerable goes in — written as a list. The
second is one structural change, where the rename and the absorption cannot be taken apart:
half of it leaves a section named for what it no longer holds.

The same applies to a subject that reaches for a vague umbrella noun to cover several
changes — `Update auth handling`, `Various fixes`, `Cleanup`. The umbrella is the "and" in
disguise.

## Scale

Keep commits small. **1 to 4 files** is the working range, and a single-file commit is a
perfectly normal unit of work. A commit touching more than a handful of files is something to
justify, not a default to settle into.

Large commits are legitimate in a few cases — a mechanical rename across many files, a
generated artifact, an initial scaffold. What makes them legitimate is that they are still
**one decision**, and the reviewer only has to agree with that one decision once.

## What to split

| Split these apart | Why |
| :-- | :-- |
| Implementation and its tests | The test commit states what the implementation is expected to do, and reads as its own reviewable claim. Reviewing them separately keeps a weak test from being waved through on the strength of the code beside it. |
| Refactor and behavior change | A refactor is reviewed by confirming behavior did **not** change. Mixed together, the reviewer cannot tell which lines were meant to alter behavior. |
| Mechanical rename and logic edit | A rename is verified by scanning that it is uniform. One hand-edited line hidden among 200 renamed ones is invisible. |
| Formatting or lint fixes and substance | Whitespace churn buries the two lines that matter. |
| Dependency bumps and code that uses them | The bump is a separate risk with a separate rollback. |
| Generated artifacts (`package-lock.json`, generated types) and hand-written source | Generated diffs are large and unreviewable; keeping them separate keeps the source commit readable. |
| Unrelated files that happen to be dirty | They are unrelated. This is the most common cause of an accidental umbrella commit. |

Registering a new artifact in an index or export barrel is its own commit as well. Adding the
class and exporting it are two decisions, and the export is the one with a public-surface
consequence.

**A class's tests are committed before its implementation.** The order is what makes the split
worth having: the test commit states what the class is expected to do, and the implementation
commit is the one that makes the statement true. Committed the other way round, the tests only
confirm what already worked, and there is no commit at which the claim stands on its own to be
reviewed. This is test-driven development written into the history rather than into the editor.

## What to keep together

- A change and the **type annotations or JSDoc that describe it**. A signature and its
  documented contract are one decision; splitting them leaves a commit whose documentation
  contradicts its code.
- A change and whatever is **required for the commit to hold together on its own** at that
  point. If splitting would produce a commit that refers to something that is not there yet —
  a call to a member the next commit defines, an import of a file it adds — the split is in the
  wrong place. Find a different seam rather than committing the dangling reference.
  - **A seam is not only a line between files. An order is a seam too.** One breaking order
    proves nothing about the rest: the same change split the other way round often passes
    through every intermediate state intact. Before concluding that no seam exists, reverse the
    order and try again.
  - **What counts as broken is what the work actually runs at that point, not everything that
    could be run there.** A commit that would fail a command nobody issues between it and the
    next one has not broken anything. Turning a check on before the entries that satisfy it are
    recorded looks like the wrong order, and is the right one where the install that would trip
    over it comes after both — the setting and its entries are configured together, and the
    command runs once they are. Judging the seam against an imagined invocation rules out
    orders that hold perfectly well, and pushes the split somewhere worse.
  - **Take a removal apart from the outside in** — the callers first, then the registration,
    then the thing itself. Each step deletes something nothing else points at any more, so no
    intermediate state refers to what is gone. Going the other way breaks at the first commit,
    which is what makes a removal look unsplittable when it is not.
  - Retiring a check that a CI workflow runs, an npm script registers, and a script file
    implements is three commits, and taken in this order not one of them leaves a dangling
    reference behind:

    ```
    Kick out the levers reference check from the CI workflow
    Kick out check:levers from package.json
    Purge scripts/check-levers.mjs
    ```

    The subjects come apart as cleanly as the commits do, because `SKILL.md` gives a removal
    two verbs instead of one — `Kick out <what> from <where>` for the file that stays without
    it, and `Purge <path>` for the file that goes. That pair is the tool for splitting one
    removal across several commits; a single `Remove` would hide the seam it makes visible.
- A rename and **every call site it touches**. Half a rename is a broken tree.
- **One document's translations, where the edit is the same edit.** A `README.md` and its
  `README.ja.md` carry one decision written twice, so a reviewer given them apart has to take
  both or neither, and the split has bought nothing while doubling the commits. The subject
  drops the extension and names the document: `Kick out the registry setting from README`.
  - **What stays split is a different edit that happens to land in the same pair of files.**
    Correcting a default that one example states wrongly, and supplying an option that another
    example omits, are two decisions in both languages — that is two commits, each touching two
    files, not one commit touching four.

## Staging a mixed working tree

When the tree already holds several unrelated changes, do not resolve it by committing
everything at once.

```bash
git add -p            # stage one decision's hunks at a time
git diff --cached     # confirm what is actually staged before committing
git diff              # confirm what is being left for the next commit
```

- Never `git add -A` or `git add .` without first checking what that sweeps in. Untracked
  scratch files, editor artifacts, and `.env` variants are picked up this way.
- When hunks for two decisions are interleaved in the same file, stage the first, commit, and
  then stage the second. `git add -p` splits hunks with `s` and edits them with `e`.

## Anti-patterns

- **Checkpoint commits.** `wip`, `save progress`, `saving` — a commit holding real changes
  whose message records that time passed rather than what changed. If a checkpoint is needed
  mid-work, use `git stash` or a local branch, and squash before the work is shared.
  - This does **not** apply to the branch-opening `Start …` marker, which the git branch
    convention describes. That commit is deliberately empty, so it makes no claim about
    granularity at all — there is no change in it to have scoped correctly.
  - Nor to the `Merge …` commit that closes a branch. It carries no change of its own either.
    What a reviewer weighs there is the branch it brings in, and that was already scoped commit
    by commit inside the branch.
  - Nor to a commit made **on the premise that it is deleted right away** — a save point taken
    only so that an operation needing a clean tree can run, undone by `git reset --soft`, by
    `git commit --amend`, or by a squash the moment it has served that purpose. **Its message
    is free.** Every convention on a subject exists to tell a later reader what changed, and
    this commit is gone before there is one: `saving-20260821-1530` is as good a subject as
    any, no verb from the vocabulary in `SKILL.md` has to fit it, and the changes in it are
    left undivided.
    - **The premise is what buys the freedom, so the premise has to hold.** A commit still
      there when the work is shared was never one of these, whatever was intended when it was
      made. Delete it before the branch goes out, or write it as any other commit.
- **End-of-day dumps.** A single commit holding everything touched since morning is the
  default outcome of never deciding granularity. Decide it while working.
- **Typo-fix follow-ups on unpushed work.** A `Fix typo` commit immediately after the commit
  that introduced the typo is noise. Fold it in with `git commit --amend` — but only while
  the commit is **unpushed**. Once pushed, a separate fix commit is correct.
- **Splitting past the point of coherence.** A commit left referring to what is not there yet,
  so that the "one decision" rule could be honored more purely, has traded a real property for
  a cosmetic one.
