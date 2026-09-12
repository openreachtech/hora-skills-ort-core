---
name: hoc-git-branch
description: "Conventions for the branches a repository carries: which branch is a trunk and what that role obliges, how a general branch is named, and how work is split off a trunk and merged back with `--no-ff`. Use before cutting a branch, before merging one back, and before deciding whether work needs a branch structure at all. What goes inside a commit, and how a subject is worded, belong to the git commit convention. Every `git rebase` here takes `-r`."
---

# Git Branch

Conventions for the branches a repository carries — which one is a trunk, what a general
branch is called, the commit that opens a trunk and the commit that closes a sub-branch.

What belongs in a single commit, and how a subject is worded, are settled by the git commit
convention. The two commits described here are the exception it points at: their subjects are
specified in full below, because what they say is inseparable from what they are for.

## Every rebase takes `-r`

**`git rebase` without `-r` (`--rebase-merges`) is forbidden. There is no ordinary case that
omits it.** Not preferred, not recommended where a branch has merges in it: the flag goes on
every invocation, and a command written without it is wrong whether or not this particular
branch happens to survive the omission.

```bash
git rebase -r <trunk> <branch>
git rebase -r --onto <trunk's new tip> <the commit this branch was cut from> <branch>
```

Without `-r`, `git rebase` drops every merge commit it replays. A branch that carried its own
sub-branches arrives flattened, and the `--no-ff` merges inside it are gone — the exact thing
`--no-ff` was used to keep. The loss is silent: the rebase reports success, the working tree
matches, and what is missing is structure no diff reports.

**Whether the branch holds a merge commit right now is not the test.** A branch with none loses
nothing today, but deciding case by case means re-examining the question at every rebase and
getting it wrong on the one branch that did carry a merge. The flag costs nothing when there is
nothing to preserve.

- The single thing that lifts it is a deliberate decision to flatten a branch's internal merges,
  taken as such and said out loud. Nobody arrives there by default.

This rule stands ahead of everything below because the commands that need it appear throughout,
and because a rebase that drops a merge cannot be spotted afterwards from the result.

### Editing a commit inside the branch takes `-i` as well

**`-r` keeps the merges; `-i` is what opens the todo list.** Given `-r` alone, a rebase runs the
sequencer but never presents a todo, so a sequence editor named in the environment is never
called — and the run reports every step successful while handing each commit back with the hash
it went in with.

```bash
git rebase -i -r --onto <base> <the commit the branch was cut from> <branch>
```

Measured on a branch carrying one merge: with `-r` alone the rebase printed eight steps and
`Successfully rebased`, and no hash changed. With `-i -r` the reword applied and the merge commit
survived, the graph keeping its shape.

- **Being unable to answer a prompt is not a reason to avoid `-i`.** Point the sequence editor
  and the message editor at scripts — one rewrites the todo, the other rewrites the message —
  and nothing prompts. An interactive rebase can be driven rather than attended.
- **Rebuilding the sub-branch by hand is the fallback for a conflict, not for a reword.**
  Cherry-picking its commits onto a fresh branch and amending the one that needed the new
  subject reaches the same tree at the cost of the branch and every merge above it.

## The trunk branch

A **trunk branch** is one that other branches are cut from and merged back into.

Four are trunks by name, in every repository.

| branch | what it carries |
| :-- | :-- |
| `main` | the mainline every other branch descends from |
| `release/x.x.x` | one version's work, until it merges into `main` |
| `dev` | long-lived integration |
| `env` | the initial environment setup |

**Every other branch is a general branch, and takes the role rather than holding it.** A
general branch behaves as a trunk for as long as work is split off it. The four above behave as
trunks whether anything is outstanding against them or not.

The shape of the name settles nothing. `release/x.x.x` is a trunk and
`retake/save-of-UserRepository` is not, and the two are the same shape.

- **Trunks nest.** A general branch cut from `main` that then has work split off it is both: a
  sub-branch of `main`, and the trunk of what it carries. The role is held against a particular
  branch, never held outright.
  - **Each half brings its own obligations, and neither cancels the other.** Being a trunk, the
    branch opens with the `Start …` marker described below; being a sub-branch, it merges back
    into `main` once it is done. That the merge is made locally, with no pull request to open
    early, takes nothing away from the marker.
- **A sub-branch merges back into its trunk, and a trunk never merges into what it carries.**
  The direction is the same at every level of the nesting.
- **A trunk is where work arrives, not where work is done.** Nothing is committed to a trunk
  directly: cut a branch for the change, commit it there, and merge it back. This holds for a
  general branch from the moment it takes the role, and it holds on a trunk whose only commit so
  far is its own opening marker — a trunk with nothing on it yet is still not a place to work.
  - The exceptions are the two commits a trunk makes about itself rather than about the work:
    the marker that opens it, and the `Merge …` commit that brings a sub-branch in. Both are
    described below, and neither carries a change of its own.
- **A trunk's published history is never rewritten.** `git push --force` and
  `--force-with-lease` are not operations these four branches take, and neither are the local
  rewrites that would make one necessary — `rebase`, `commit --amend`, `reset` onto an already
  pushed commit. There is no permission that unlocks this; it is what the four names mean.
  - **The reason is who else is holding the branch.** A trunk is what every other branch is cut
    from, so its commits are already in clones, in merge commits' parents, and in whatever CI
    recorded against them. Rewriting it does not correct a mistake — it makes everyone else's
    copy disagree with the remote, silently, until they try to push.
  - **A mistake already merged into a trunk is corrected by a new commit**, on a branch that
    merges in like any other. A subject worded badly, a value that turned out wrong, a file that
    should not have gone in: the trunk gains a commit that says so, and the record of the
    mistake stays. A history a reader can trust is worth more than one that is tidy.
  - **A sub-branch is the opposite**, until it is pushed and opened for review: rewriting it is
    how the structure described below gets cut at all. Nobody else is holding it, so nothing
    disagrees.

## When the structure is decided

**Commit the work in one line first, and shape the branches once it is finished.** How many
sub-branches the work wants, and whether it wants a trunk at all, are answers the work itself
gives — and it cannot give them before it exists. A structure chosen in advance is a guess, and
a guess that turns out wrong costs a rebuild of everything already committed under it.

This is the opposite timing from granularity, which the git commit convention has decided
*while working*: by the time the tree holds six unrelated edits, the cheap moment for that one
has passed. Both rules say the same thing — decide at the point where the answer is knowable —
and the answers become knowable at different points.

So the order is: commit in a meaningful sequence until the work is ready for review, fold
together whatever turns out to be one decision after all, and only then cut the markers and
split the line into sub-branches.

- **Every sub-branch is cut from the trunk, never from the sub-branch before it.** Splitting a
  finished line means returning to the trunk for each cut, not walking forward along the line as
  the branches come off it. A branch cut while standing on the previous sub-branch carries that
  branch's commits as well as its own, and the two arrive at the trunk stacked instead of side by
  side — the second merge then reopens a line the first one closed.
  - **What `git branch` shows afterwards looks the same either way.** Only the commit each branch
    was created from tells the two apart, and by the time the merges expose it the structure is
    already built.
  - **The exception is a branch that cannot stand up without another one's work**, where that
    other reaches the trunk by its own pull request rather than by the local merge described
    below. Sitting on it is how the prerequisite is had, and the reason the rule gives — a second
    merge reopening a line the first one closed — does not reach the case, because the two never
    merge into the trunk together. When the prerequisite lands, rebase the stack onto the trunk's
    new tip and carry on.
- **This holds only while the commits are unshared.** Folding and splitting both rewrite
  history. Once the work has been pushed, the line stands as it is, and a structure it did not
  get is a structure it does not get.
- **The marker is cut as part of that shaping**, which is why there is no separate rule for
  rewinding a trunk to insert one that was forgotten. A marker cannot be forgotten by a process
  that ends with cutting it.

What the finished line is looked at for:

- **One sub-branch's worth of work needs no trunk.** A single branch carrying the commits
  reaches the same pull request. A trunk earns its two extra commits — the marker and the merge
  — by grouping several pieces of work, and there is nothing to group.
- **Sub-branches that each carry one commit need no trunk either.** The merge commit's subject
  then restates the single commit beneath it, and the structure costs more commits than the work
  contains. A trunk is worth its cost when a merge commit names something its commits do not say
  individually.
- **One sub-branch with something to group is enough.** The others may carry a single commit
  each; what makes the trunk worth having is that at least one merge names a piece of work
  assembled from parts.
- **A sub-branch of its own is decided twice over: by what the piece carries, and by whether it
  stands beside the others as work in its own right.** The bullets above settle the first. The
  second asks whether the pieces are siblings — each a thing that was done, reviewable on its own
  merit — or steps toward one thing they share. **Steps of one piece go in one sub-branch,
  however much each carries.**
  - Two classes reshaped, each for reasons of its own, are siblings: either could be taken and
    the other left. A fix to the code and the removal of the relaxation that fix made unnecessary
    are not — the second exists only because the first happened.
  - **Looking like siblings is not being siblings.** Two methods rewritten, in two files, in two
    classes, read as a pair of independent jobs. If neither of them alone clears the thing the
    work set out to clear, they are halves of one job and belong on one branch.
  - Deciding on size alone splits a pair that should have been one branch, because size is the
    test that a pair of one-commit steps passes and a pair of large steps fails.

### The same work across sibling repositories takes the same order

Where one piece of work lands in two repositories at once, the sub-branches under each trunk
are cut in the same sequence, so the two histories read alike. Put whatever is peculiar to one
repository first and whatever both share last: the shared part then sits at the same depth in
both, and a reader comparing them has the differences gathered at one end rather than
interleaved.

Where a repository has nothing to put in the position its sibling fills, it skips that position
rather than filling it with something else. A gap in one of two matching sequences still reads
as a match; a substitution does not.

## Naming a general branch

A general branch is named `<verb | category>/xxxx`. Before the slash goes a verb for what the
branch does, or a category for what kind of work it is; after it, the name is free — an
identifier may appear verbatim.

```
declare/AlphaClass
define/sendMessage-of-AlphaClass
rename/FormElementClerk
fix/type-errors-reported-by-the-client-package
install/date-fns-4.1.0
tidyup/environment-files
```

**The name is written for whoever scans `git branch` while the work is still in flight**, so it
is deliberately descriptive. Nothing reads it after the branch is gone.

- **The verb is the word the branch's own commits would use, lowercased.** `Declare` names a
  class and `Define` names a member, a function or a constant, which is why
  `declare/AlphaClass` and `define/sendMessage-of-AlphaClass` say what they carry without any
  further explanation. The verbs are listed in the git commit convention.
  - **A verb of two words joins into one, with no hyphen** — `Tidy up` gives `tidyup/xxxx`,
    `Kick out` gives `kickout/xxxx`, `Turn off` gives `turnoff/xxxx`. The slash ends the token,
    so nothing inside it has to.
- **A member is written `<member>-of-<class>`.** The slash is already spent on the verb, so what
  is left spells the relation out instead of punctuating it.
- **Work of a scale that will make the branch a trunk takes a category at a higher level of
  abstraction** — `implement/xxx`, `setup/xxx`, `feature/xxx`, `retake/xxx`, `update/xxx`. A
  branch that is about to have six branches cut from it cannot be named for one narrow verb
  without lying about five of them.
- **A sub-branch cut from a general branch acting as a trunk is named the same way**, and the
  nesting adds no constraint of its own: `<verb | category>/xxxx`, free. The narrow verbs
  belong here, where each branch really does carry one thing.
- **Leave out the article.** `kickout/the-copied-rules` and `kickout/copied-rules` point at the
  same work, and the shorter one is what a reader scanning `git branch` gets through faster.
  What comes after the slash is a label, not a sentence.

## The branch-opening marker commit

A branch that will act as a trunk opens with an **empty commit** whose subject begins with
`Start` — or with `Release`, on a `release/x.x.x` trunk. This is a deliberate convention, not a
checkpoint or a placeholder.

```bash
# opening a long-lived dev branch
git switch -c dev
git commit --allow-empty -m 'Start dev'

# opening a general branch that will carry sub-branches
git switch -c feature/equip-tools-for-each-application
git commit --allow-empty -m 'Start adding the skills installer'

# opening a nested trunk: cut from the branch above, and carrying sub-branches of its own
git switch -c update/domains-a-repository-selects
git commit --allow-empty -m 'Start updating the domains a repository selects'
```

- It must be **empty** (`--allow-empty`). It exists to put a commit on a branch that has no
  work on it yet, so there is nothing for it to carry. A `Start …` subject on a commit that
  actually contains changes is not this convention — it is a mislabelled change.
- It is the **first commit on the branch**, made immediately after branching.
- **One per trunk, and none on a sub-branch.** The test is the branch's role: will other
  branches be cut from this one and merged back into it? Where the answer is yes, the branch is
  a trunk — that is what this document defines the word to mean, and that definition is the
  whole of the condition.
- **A nested trunk takes a marker of its own.** A general branch cut from `main` that then has
  work split off it is a sub-branch and a trunk at once, and it is the trunk half the marker
  answers to. Such a branch merges back locally, with no pull request anywhere in it, and it
  still opens with `Start …`.
- **Whether a pull request is ever opened decides nothing.** Where the merge does go through a
  host, the marker buys a branch that can be reviewed before any code exists — but that is
  something the commit makes possible, never the test for making one.
- **`Start` is not a verb for resuming work mid-branch.** A branch already carrying commits has
  nothing left to open.
- The subject names **what is being started**, which depends on the kind of trunk.
  - A **trunk that is one by name** is named directly: a `dev` branch opens with `Start dev`.
    Here `dev` is the branch, not a placeholder word.
  - A **general branch acting as a trunk** states the work it will carry — `Start adding the
    skills installer`, `Start renaming kit/skills/_core/ to core/`. A later reader scanning the
    log gets the branch's purpose for free.
  - A **`release/x.x.x` trunk** is opened by its version alone: `Release 0.2.0`. The word
    `Start` does not appear, because the version is the whole of what is being started.
  - **Where the work carries content in from elsewhere, the marker names the origin** — `Start
    migrating the mail templates from lunas-ec-cart-backend`. Stated once here, it covers every
    commit on the branch, and the merge commit keeps it in the history after the branch is gone.
- **The marker takes no type prefix, in either message format.** Repositories on Conventional
  Commits write `Start dev`, not `chore: start dev`. The marker sits outside the format.

## The merge commit

A branch merges back into its trunk with `--no-ff`, and the merge commit that results carries a
subject of its own.

```
Merge the classes of the skills installer
Merge the core/ rename in the repository documents
```

- **It names the work, never the branch.** `Merge rename/FormElementClerk` says only what
  `git log --graph` already shows, and the branch is deleted moments later. What it carried is
  the part that has to survive it.
- **It stands in for the message a host would have written.** A merge that goes through a pull
  request is described for free — `Merge pull request #53 from …`. A merge made locally has no
  such author, and this subject fills the gap.
- **It takes no type prefix, in either message format**, for the same reason the branch-opening
  marker takes none: it carries no change of its own. Repositories on Conventional Commits
  write `Merge …`, not `chore: merge …`.
- **A merge made through a pull request is left alone.** The host writes it, and no one here
  chooses its wording.

## Merging back into a trunk

- **Always `--no-ff`, never fast-forward.** A fast-forward leaves no commit a human can point
  at: the branch's commits are strung onto the trunk's line, and the fact that they arrived
  together, as one piece of work, stops being visible at all.
- **Before each merge, confirm the branch is fast-forwardable.** `--no-ff` is only doing its
  work when a fast-forward is what would otherwise have happened. On a branch that has fallen
  behind the tip, git makes a three-way merge regardless, and the flag changes nothing.

  ```bash
  git merge-base --is-ancestor <trunk> <branch>   # must succeed
  ```

  Failure means the branch has not been rebased onto the current tip. Rebase it, then merge.
  - **A merge commit's two parents are not evidence that `--no-ff` did anything.** A three-way
    merge has two parents as well, and nothing in the finished history separates the two. That
    is why this is checked before the merge, and cannot be checked after it.
- **Delete the branch once it is merged.** Its name was written for whoever watched the work in
  flight, and that reader is gone. This includes a trunk that merges into another trunk —
  `dev` and `env` are deleted once they land on `main`.
  - **A trunk kept alive after it merged sits at the past of the trunk it merged into**, and
    everything cut from it afterwards inherits that. Merging `origin/main` back in would
    settle it, but re-cutting the branch settles the same thing without leaving a merge commit
    that carries no work of its own:

    ```bash
    git branch -d dev
    git switch -C dev origin/main
    ```
- **Merging several sub-branches back is a cycle, not a batch: merge one, rebase the next onto
  the trunk's new tip, merge it, rebase the one after that.** Every merge moves the tip, so each
  branch is rebased against a commit that did not exist while the branch before it was still
  open. Each then merges into the trunk as it now stands, rather than reopening a line that was
  already closed.

  ```bash
  git switch <trunk>
  git merge --no-ff <first> -m 'Merge …'

  git rebase -r <trunk> <second>        # onto the tip the merge above just made
  git switch <trunk>
  git merge --no-ff <second> -m 'Merge …'

  git rebase -r <trunk> <third>         # onto the tip that merge made
  git switch <trunk>
  git merge --no-ff <third> -m 'Merge …'
  ```

  - **The rebases cannot be done in advance.** Aiming them all at one point — the commit the
    branches were cut from, or anywhere else — settles nothing past the first merge: the second
    branch is behind the tip again by the time its turn comes. The commit each rebase needs does
    not exist until the merge before it is made.
  - **Two branches are the smallest case of this, not a rule of their own.**
