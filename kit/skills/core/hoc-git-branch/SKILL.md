---
name: hoc-git-branch
description: "Conventions for the branches a repository carries: which branch is a trunk and what that role obliges, how a general branch is named, the empty marker that opens a trunk, and the `--no-ff` merge that closes a sub-branch along with the subject that merge commit carries. What goes inside a commit, and how a subject is worded generally, belong to the git commit convention. Use before cutting a branch, before merging one back, and before deciding whether work needs a branch structure at all."
---

# Git Branch

Conventions for the branches a repository carries — which one is a trunk, what a general
branch is called, the commit that opens a trunk and the commit that closes a sub-branch.

What belongs in a single commit, and how a subject is worded, are settled by the git commit
convention. The two commits described here are the exception it points at: their subjects are
specified in full below, because what they say is inseparable from what they are for.

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
tidyup/the-environment-files
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
git switch -c update/the-domains-a-repository-selects
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
- **When two branches were cut from the same commit on a trunk, whichever merges second rebases
  onto the trunk's new tip first.** The second branch then merges into the trunk as it now
  stands, rather than reopening a line that was already closed.
- **Every rebase in this scheme uses `-r` (`--rebase-merges`).**

  ```bash
  git rebase -r --onto <trunk's new tip> <the commit this branch was cut from> <branch>
  ```

  Without `-r`, `git rebase` drops every merge commit it replays. A branch that carried its own
  sub-branches then arrives flattened, and the `--no-ff` merges inside it are gone — the exact
  thing `--no-ff` was used to keep.
