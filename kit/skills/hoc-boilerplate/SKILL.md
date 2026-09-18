---
name: hoc-boilerplate
description: "Conventions for `about-boilerplate.md` and the repositories it appears in: the `## Version` field that moves in a boilerplate and is frozen everywhere else, and how a boilerplate's own release is closed and guarded. Read before touching `about-boilerplate.md` at all, and before opening or releasing a boilerplate. Which branch merges where in general belongs to the git branch convention; publishing a package belongs to the npm publish convention."
---

# Boilerplate

A boilerplate repository is cloned to start new ones. What it hands over is the repository
itself — its files, its workflows, its configuration — so the thing it releases and the thing
it is are the same object. That single fact decides everything below.

`about-boilerplate.md` travels with every clone, which is why this convention reaches far
past the few repositories that are boilerplates. **Most readers of this file are in a clone,
and for them the whole of the rule is: leave it alone.**

## Which repository you are in decides what you may do

**The test is the repository's name.** A name containing `boilerplate` is a boilerplate;
everything else is a clone of one.

| | `about-boilerplate.md` | `## Version` |
| :-- | :-- | :-- |
| Name contains `boilerplate` | may be updated | moves, once per release |
| Every other repository | **never touched** | frozen at the version cloned |

- **In an application, or any other clone, the file is not updated at all.** Not the version,
  not the prose, not the formatting — **and not deleted.** It is the record of where the
  repository came from, and a record that can be edited by whoever finds it inconvenient is
  not a record.
- **The name is the whole of the test**, and it is the same test `boilerplate-version.yml`
  applies (`*-boilerplate*`) before it checks anything. Nothing about the file's contents
  tells the two cases apart: the same file, with the same field, means the live version in
  one repository and a frozen record in the other.
- **Check the name before the file.** The two cases are the same edit with opposite verdicts,
  so the question cannot be answered by looking at what is being changed.

## In a clone, a stale `## Version` is the correct state

The field says which version of the boilerplate this repository was taken from. **That is a
fact about the past, and the past does not move.**

- **A gap against the upstream is not drift, and not a missed sync.** The boilerplate
  releasing 1.7.0 does not make a repository cloned at 1.4.0 wrong. Both statements are true
  and they are about different things.
- **Bringing a clone in line with a newer boilerplate does not change it either.** The
  repository now holds newer files; it was still created from the version it says.
- **So it is never a synchronisation candidate.** When working out what a repository has
  fallen behind on, leave this file out of the comparison from the start — not listed, not
  proposed, not mentioned as a possibility.
- Where a record of having synced is wanted, it goes in a section of its own — a `## History`
  line naming what was taken in — and **only when it has been asked for.** The version field
  is not where it goes, and adding a section nobody asked for is its own mistake.

## In a boilerplate, the version lives in `## Version`

There is no `package.json` version to release and no tarball to publish. **The version of a
boilerplate is the line under `## Version` in `about-boilerplate.md`, and nothing else
states it.**

```markdown
## Version

1.1.0
```

- **The bump is one commit**, the last one on `release/x.x.x`, on its own branch
  `update/boilerplate-version-to-x.x.x` and its own pull request.

  ```
  Update boilerplate version to 1.1.0
  ```

  Nothing is generated from it, so there is no second commit — the pair a package's own bump
  makes exists because a lockfile has to be regenerated, and here there is no lockfile to
  regenerate.
- **Until it is bumped, the release cannot reach `main`**, because the guard below refuses
  the pull request. That refusal is what the position of the bump buys: a release branch that
  is not finished has nothing to merge with.

## The guard holds three statements of one version together

`boilerplate-version.yml` runs on pull requests into `main`, and only where the repository
name contains `boilerplate` and the head branch is `release/*`. Everything else it passes.

It requires all three of these to name the same version:

- the version in backticks in the pull request title
- the branch name `release/x.x.x`
- the line under `## Version` in `about-boilerplate.md`

**Three hand-typed statements of one value are worth nothing unless they must agree.** The
third is the reason the check exists at all: it sits in prose that no test reads, so a
release that left it behind would go out with nothing noticing.

## A boilerplate merges only `release/x.x.x` into `main`

Whoever starts a new repository takes the tip of `main`, so **every commit that reaches
`main` is something released.** There is no such thing as a change that arrives there without
being part of a version.

- **`env` is not used for it.** A repository that publishes a tarball can merge `env`,
  because a change the tarball does not carry needs no version. A boilerplate has no such
  change: the workflows, the configuration and the documents are exactly what it hands over.
- **`hotfix/xxxx` is not used either.** A fix that cannot wait still goes out as a release,
  on a `release/x.x.x` with the patch raised. Sending it any other way skips the guard above,
  which only inspects `release/*`, and puts a change into `main` that no version names.
- The rest of the branching — what a trunk is, how a branch is named, what opens one — is the
  git branch convention's, and this narrowing is one of the examples it already describes.
