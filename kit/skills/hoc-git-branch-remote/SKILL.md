---
name: hoc-git-branch-remote
description: "The branches a repository keeps on its remote: `main` as the default branch, which branches may merge into `main` by kind of repository — npm package, boilerplate or application — and why a trunk's published history is never rewritten. Use before opening a trunk, a pull request into `main`, or a rewrite or force-push of a trunk, or when deciding whether `env`, `hotfix/xxxx` or `dev` may be used. How branches are cut and merged belongs to the git branch convention."
---

# Git: Remote Branches

The branches a repository keeps on its remote, and which of them may reach `main`.

## The default branch is `main`

- **Every repository's default branch is `main`.** It is the branch a clone checks out, the base
  a pull request is opened against unless another trunk is named, and the branch every trunk
  descends from.

## What may merge into `main` is decided by the kind of repository

**Which branches may merge into `main` is not a choice made per change.** It follows from what
the repository is, because what reaches `main` is what the repository releases.

| Kind of repository | May merge into `main` |
| :-- | :-- |
| npm package | `release/x.x.x`, `env` |
| boilerplate | `release/x.x.x` |
| application | `release/x.x.x`, `env`, `hotfix/xxxx` |

- **An npm package follows Git Flow.** A version reaches `main` through `release/x.x.x`, and
  `env` carries what the published tarball does not hold, which needs no version.
- **A boilerplate takes `release/x.x.x` alone.** Why the others are refused there belongs to
  `/hoc-boilerplate`.
- **An application developed with Hora Kit takes `release/x.x.x`, `env` and `hotfix/xxxx`.**
  `hotfix/xxxx` is the application's alone; neither an npm package nor a boilerplate merges one.
- **`hotfix/xxxx` exists for the case with no time to run a version through.** An npm package is
  published to the registry and taken from there, and a boilerplate is there to be cloned into
  other repositories; both reach their users through a version. An application runs in front of
  its users, and a failure there is measured in a different sense of time altogether — the fix
  goes out before a release could be cut for it.

### `dev` in an older application

- **An older application that still carries `dev` may merge it into `main`.** That is accepted
  for the repository as it stands, not offered to a new one.
- **Once `dev` has merged into `main`, move to `release/x.x.x` wherever it can be done.**

## A trunk's published history is never rewritten

**A trunk's published history is never rewritten.** `git push --force`, `-f` and
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
- **A trunk may be re-cut while nothing on the remote descends from it, and both halves of
  that are a person's.** Where no branch has been pushed from it and no pull request is open
  against it, the commits the rule protects are held by nobody, and the reason above does not
  reach the case.
  - **What may be run here is `git fetch`, and nothing past it** — why belongs to
    `/hoc-git-push`. Refresh the remote-tracking refs, show what they now hold, and leave both
    the judgement and the push where they belong.
  - **Local branches cut from it are recovered afterwards**, by `--onto` naming the commit
    each was cut from. That half is ordinary work and asks no permission of its own.
  - **A single pushed branch, or one open pull request, ends it** and the rule is back whole.

## What else belongs elsewhere

- **What a trunk is** — that it is where work arrives, how a branch is cut from it and merged
  back — belongs to `/hoc-git-branch`.
- **Pushing** — the permission it takes, and that a trunk is never force-pushed — belongs to
  `/hoc-git-push`.
