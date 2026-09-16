---
name: hoc-git-branch
description: "Conventions for the branches a repository carries: which five are trunks and what that obliges, the one worked on directly, which may be cut from `main`, per flow, how a general branch is named, and how work is split off a trunk and merged back with `--no-ff`. Use before cutting a branch, before merging one back, and before deciding whether work needs a branch structure. What a commit holds, and how its subject is worded, belong to the git commit convention. Every `git rebase` here takes `-r`."
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

### A rewritten trunk takes `--onto` as well

**`git rebase -r <trunk> <branch>` is right while the trunk only moves forward, and wrong once
the trunk has been rebuilt.** The two-argument form replays everything in `<trunk>..<branch>`,
and that range holds the branch's own commits only for as long as the commit the branch was cut
from is still on the trunk. Rebuild the trunk — rewind it and merge the sub-branches again — and
that commit is gone, so the range reaches back to whatever the two still share and sweeps up the
old trunk's merges on the way.

```bash
git rebase -r --onto <the trunk's new tip> <the commit this branch was cut from> <branch>
```

**`-r` is what makes the damage silent.** The flag exists to preserve merge commits, and it
preserves these too: the old trunk's `Merge …` commits are replayed inside the sub-branch, so the
graph comes out with two paths carrying the same subjects, and the branch appears to hold work
that was folded away. The rebase reports success and the tree is correct, which is why nothing
downstream catches it.

- **The graph is where it shows.** `git log --graph` after each rebase, and one merge subject
  appearing twice is the whole of the symptom.
- **Measured twice on one branch structure.** Both times the trunk had been rewound and rebuilt,
  both times the two-argument form recreated its merges inside the sub-branch, and both times
  `--onto` naming the commit the branch was cut from replayed only what the branch held.

## The trunk branch

A **trunk branch** is one that other branches are cut from and merged back into.

**Five are trunks by name, in every repository: `main`, and the four branches that may merge
into it.**

| branch | what it carries | may merge into `main` |
| :-- | :-- | :-- |
| `main` | the mainline every other branch descends from | — |
| `release/x.x.x` | one version's work, until it merges into `main` | yes |
| `hotfix/xxxx` | one fix that cannot wait for a release | yes |
| `dev` | long-lived integration. **Legacy** | yes |
| `env` | the initial environment setup, and changes that leave the released artefact untouched | yes |

**Every other branch is a general branch, and takes the role rather than holding it.** A
general branch behaves as a trunk for as long as work is split off it. The five above behave as
trunks whether anything is outstanding against them or not.

- **`dev` is legacy.** It stays in the set for backward compatibility, and nothing new is opened
  on it. A repository still carrying one holds it to every rule here.
- **Which of the four a given repository actually uses is narrower than the set**, and it is
  decided by what that repository releases rather than by this convention. A repository that
  ships its own contents to whoever clones it merges nothing but `release/x.x.x` into `main`;
  one that publishes a tarball can also merge `env`, because a change the tarball does not carry
  needs no version. **The set here is the ceiling, not the instruction.**

The shape of the name settles nothing. `release/x.x.x` is a trunk and
`retake/save-of-UserRepository` is not, and the two are the same shape.

- **Trunks nest.** A general branch cut from a trunk that then has work split off it is both: a
  sub-branch of that trunk, and the trunk of what it carries. The role is held against a particular
  branch, never held outright.
  - **Each half brings its own obligations, and neither cancels the other.** Being a trunk, the
    branch opens with the `Start …` marker described below; being a sub-branch, it merges back
    into the trunk it was cut from once it is done. That the merge is made locally, with no pull
    request to open early, takes nothing away from the marker.
- **A sub-branch merges back into its trunk, and a trunk never merges into what it carries.**
  The direction is the same at every level of the nesting.
- **A trunk is where work arrives, not where work is done.** Nothing is committed to a trunk
  directly: cut a branch for the change, commit it there, and merge it back. This holds for a
  general branch from the moment it takes the role, and it holds on a trunk whose only commit so
  far is its own opening marker — a trunk with nothing on it yet is still not a place to work.
  - The exceptions are the two commits a trunk makes about itself rather than about the work:
    the marker that opens it, and the `Merge …` commit that brings a sub-branch in. Both are
    described below, and neither carries a change of its own.
- **`hotfix/xxxx` is the one trunk worked on directly.** The fix is committed onto it, with no
  sub-branch and no marker, and the branch merges into `main` as it stands.
  - **The reason is what the branch is for.** It exists to carry one fix past the release that
    would otherwise have carried it, and every step between the fix and `main` is time the
    failure it fixes is still running. A sub-branch and its merge buy structure that nothing is
    going to read on a branch holding one fix.
  - **It is a trunk in every other respect.** Its history is not rewritten once pushed, it
    reaches `main` through a pull request like the rest, and it is deleted on merge.
- **A trunk's published history is never rewritten.** `git push --force` and
  `--force-with-lease` are not operations these branches take, and neither are the local
  rewrites that would make one necessary — `rebase`, `commit --amend`, `reset` onto an already
  pushed commit. There is no permission that unlocks this; it is what the names mean.
  - **On a `main` that something releases from, it goes further.** The branch is the record of
    what was published, handed to whoever cloned it, or put in front of users, and rewriting it
    rewrites when each of those happened.
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

### What may be cut from `main`

**Under Git flow, the only branches cut from `main` are the trunks that may merge into it.**
Everything else is cut from one of those. A general branch taken straight off `main` has nowhere
to land: the four are what `main` accepts, a guard on `main` refuses the rest, and the branch
arrives at a base it is not allowed to merge into.

**Under GitHub Flow every branch is cut from `main` and returns to it**, because there is no
other trunk to cut from. So one and the same branch is correct in one repository and stranded in
the other, and which it is cannot be read off the branch itself.

**Ask which flow the repository is on before cutting a branch from `main`.** Nothing announces
it: a repository that has not yet opened `dev`, `env` or a release looks exactly like one that
never will, and a wrong guess is not found until the pull request is refused. The tells are the
four names and a guard on `main` — where none of them is there, the repository is on GitHub
Flow, and cutting from `main` is what it wants.

- **This is the only branching question that is asked rather than decided.** Everywhere else
  this convention settles the answer itself, because the answer does not change between
  repositories. Here it does, and the cost of assuming is a branch that has to be rebuilt.

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

### The order the sub-branches merge in

**Merge the cheap ones first and the substantial ones last.** Where one sub-branch changes a line
of configuration and another changes the code behind it, the line of configuration goes in first
and the code last.

**The reason is how a diff is read.** A reader starts at the newest commit and works down, so
whatever sits on top is what they see first and read hardest. Put a one-line change there and the
substantial one is buried beneath it; put the substantial one there and the small changes sit
where they cost nothing, at the bottom, passed over on the way.

- **This is not an argument about risk.** Ordering by cost so that the cheap gains survive if the
  expensive work is rejected reaches the same order by another route, and it is not the reason.
- **A change that only becomes possible once the others land goes last, whatever it costs.** Where
  several sub-branches each clear the way for one change to a file they share, that change is not
  spread across them: it gathers into a sub-branch of its own, placed after them. A relaxation
  removed only once every file it named has been fixed is the ordinary shape of this.
- **The order reverses where the sub-branches reach the trunk through pull requests.** Then the
  substantial one goes first and the cheap ones follow it. The reason above does not reach that
  case: a pull request is read on its own rather than as one diff worked down from the top, so
  there is no top for a small change to occupy. What the order decides instead is which piece of
  work the reviewer meets first, and that is the one the rest depends on.
  - **The two rules are told apart by the route, never by the work.** The same pair of branches
    takes one order merged locally and the other order opened as pull requests, so the question
    to ask is how they reach the trunk — not how large they are, which is what both rules
    measure once the route is known.
- **Do not reorder the merges to dodge a rename.** A rebase carries a commit onto a file its base
  renamed underneath it. Measured on a branch whose base had renamed a file the branch edits: the
  edit landed on the new name, every merge the branch held survived, and the tree came out
  differing only by what the new base added. A conflict predicted there is not a reason to put a
  branch anywhere but where its size says.

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
- **The label names the whole at one altitude, and never lists what the branch carries.** A
  branch holding two changes is still one branch, and `and` in the label hands the reader the
  division before the thing. Where several changes land on one thing, the label is that thing and
  the changes are what the commits are for.

  ```
  Bad   add/retry-and-timeout-options-to-AlphaClient
  Good  update/AlphaClient
  ```

  The altitude to find is the one the changes sit beneath, and it is usually already named —
  the class, the module, the document they all touch. A label that reaches for `and` is one
  written before that name was looked for.

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
  - **`hotfix/xxxx` is the exception, and it is the same test that exempts it.** Nothing is cut
    from it, so there is nothing for a marker to open. A branch whose first commit is the fix
    needs no commit placed ahead of the work, because the work is already there.
- **A nested trunk takes a marker of its own.** A general branch cut from a trunk that then has
  work split off it is a sub-branch and a trunk at once, and it is the trunk half the marker
  answers to. Such a branch merges back locally, with no pull request anywhere in it, and it
  still opens with `Start …`.
- **Whether a pull request is ever opened decides nothing.** Where the merge does go through a
  host, the marker buys a branch that can be reviewed before any code exists — but that is
  something the commit makes possible, never the test for making one.
- **`Start` is not a verb for resuming work mid-branch.** A branch already carrying commits has
  nothing left to open.
- **The subject is written at the altitude the branch name is written at.** Both name one piece
  of work, so a marker listing what the branch will carry fails the same way a label does, and
  the two then disagree besides.
- The subject names **what is being started**, which depends on the kind of trunk.
  - A **`dev` trunk** is named directly: `Start dev`. Here `dev` is the branch, not a
    placeholder word.
  - A **general branch acting as a trunk** states the work it will carry, and the shape is
    `Start <verb>ing xxxx` — the verb in its `-ing` form, then what it acts on.

    ```
    Start adding the skills installer
    Start renaming kit/skills/_core/ to core/
    Start updating the domains a repository selects
    ```

    A later reader scanning the log gets the branch's purpose for free.
  - An **`env` trunk** takes that same shape rather than its own name — `Start provisioning
    environment`, `Start setting up the environment`. It is a trunk by name like `dev`, and
    unlike `dev` the name says nothing about which environment work is being opened.
    - **What follows the verb is the thing being provisioned, never the branch.** `Start env`
      names the branch and stops there, and `Start provisioning env` puts the branch back into
      the slot the work belongs in. Either way a reader scanning the log learns only which
      branch they are on, which the ref beside the subject has already told them.
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
  `release/x.x.x`, `hotfix/xxxx`, `dev` and `env` are all deleted once they land on `main`.
  - **`git branch -d` judges against the branch's upstream, not against where you are standing.**
    A sub-branch that was given one is refused although the trunk in front of you holds every
    commit it carries:

    ```
    warning: not deleting branch 'add/xxxx' that is not yet merged to
             'refs/remotes/origin/release/x.x.x', even though it is merged to HEAD
    error: the branch 'add/xxxx' is not fully merged
    ```

    Unset the upstream and delete again:

    ```bash
    git branch --unset-upstream <branch>
    git branch -d <branch>
    ```

    **`-D` is not the answer to this.** It deletes whatever it is given, so it removes the check
    rather than the cause — and on the one branch where the check was right, nothing is left to
    say so.
  - **The upstream arrives from the start point.** `git switch -c <name> origin/<trunk>` sets
    one and `git switch -c <name> <trunk>` does not, so a branch cut from a remote-tracking ref
    carries a tie to that ref for the rest of its life. The refusal above is usually the first
    time anybody meets it.
  - **A trunk kept alive after it merged sits at the past of the trunk it merged into**, and
    everything cut from it afterwards inherits that. Merging `origin/main` back in would
    settle it, but re-cutting the branch settles the same thing without leaving a merge commit
    that carries no work of its own:

    ```bash
    git branch -d dev
    git switch -C dev origin/main
    ```
  - **Fetch before cutting from a remote-tracking ref.** `origin/main` is a local copy of what
    the remote held when it was last fetched, and nothing refreshes it on its own. Cut from a
    stale one and the trunk stands at a commit the remote has left behind — silently, because
    every branch cut from it afterwards inherits that base and the first report of it is the size
    of a pull request's diff.

    ```bash
    git fetch origin
    git switch -C dev origin/main
    ```

    **Cutting from a local branch does not need the fetch, and does not get the guarantee
    either**: a local trunk is as old as the last time somebody moved it.
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
  - **After each rebase and before its merge, confirm the branch still carries something.** A
    rebase drops a commit whose change the trunk already holds, and where that was the branch's
    only commit the branch arrives at the trunk's own tip. The `--no-ff` merge that follows then
    prints `Already up to date.`, exits zero, creates nothing, and **a rung of the structure is
    gone** — leaving a graph that reads as one which was never meant to have it.

    ```bash
    git rebase -r <trunk> <branch>
    git rev-list --count <trunk>..<branch>   # must be greater than zero
    ```

    **Measured before it and after it is not the same measurement.** Before the rebase the count
    was one and would have passed; the rebase took it to zero. The only warning git gave was a
    hint about skipped cherry-picks, which `-q` and a trailing `tail` both remove.

    - **This is not the ancestry check above, and neither catches what the other does.** A
      branch left behind the tip fails `--is-ancestor` and still makes a merge commit, because
      it holds commits of its own. A branch sitting exactly on the tip **passes**
      `--is-ancestor` — it is not behind anything — and makes none.
    - **A count of zero is not answered by skipping the merge.** The change is in the trunk
      already, so the structure was drawn with one rung too many, or the change belongs to a
      different sub-branch than the one that carried it. Both are decisions, not repairs.
    - The same emptiness arrives by accident wherever a step meant to fill the branch reported
      success without committing — a `cherry-pick` refused for a bad flag, a patch that did not
      apply, a copied file identical to the one already there. The check does not care which.
  - **Two branches are the smallest case of this, not a rule of their own.**
