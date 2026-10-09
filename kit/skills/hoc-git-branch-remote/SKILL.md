---
name: hoc-git-branch-remote
description: "The branches a repository keeps on its remote: `main` as the default branch, and which branches may merge into `main` — decided by what kind of repository it is, an npm package, a boilerplate or an application. Use before opening a trunk, before a pull request into `main`, or when deciding whether a repository may use `env`, `hotfix/xxxx` or `dev`. What a trunk is and how branches are cut and merged belong to the git branch convention."
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

## What else belongs elsewhere

- **What a trunk is** — that it is where work arrives, that its published history is never
  rewritten, how a branch is cut from it and merged back — belongs to `/hoc-git-branch`.
- **Pushing** — the permission it takes, and that a trunk is never force-pushed — belongs to
  `/hoc-git-push`.
