---
name: hoc-npm-publish
description: "How a package release is ordered and checked before it goes out — where the version bump sits among the commits, what to do when it turns out not to be last, and the audit that reads the artefact a consumer will receive. Use this skill when preparing a release, bumping a package's own version, or deciding whether a package is ready to publish. Moving the dependency versions a release takes in belongs to the dependency-raising convention."
---

# npm Publish

Publishing is the one step in a package's life that cannot be undone. A version, once
published, stays fetchable: the registry does not withdraw a publish older than its grace
period, because withdrawing it would break every dependant. The install-scripts convention
draws on that permanence from the consuming side — here it applies to **your own release.**

So a release is guarded twice.

- **Nothing unfinished can get out.** The ordering rules below make a premature publish fail
  rather than succeed.
- **Nothing goes out unread.** The audit in
  [pre-publish-audit.md](./references/pre-publish-audit.md) reads the artefact a consumer
  will receive, not the repository it was built from.

## The version bump is the last commit of the release

The package's **own** version is bumped only after every change going into that release is
done, in the final commit. Two things depend on it.

- **A premature publish fails instead of succeeding.** While the version still matches what
  is already published, the registry rejects the publish as a collision — so a half-finished
  tree cannot get out by accident. Bump the version early and that protection is gone: any
  publish from any intermediate state goes through.
- **It concentrates the install into one run.** Bumping the version means the lockfile
  has to be updated too, which is the occasion to run the install — once, at the end. The
  dependency tree settles in that one run rather than being shaken through the work.

**The bump is therefore two commits.** The manifest first; then the lockfile, after the one
install.

```
Update the package version to x.x.x in package.json
Update package-lock.json after npm install
```

- Splitting them is not a special rule for releases — the git commit convention already
  separates a generated artefact from hand-written source. What is specific here is **the
  order and the single install between them.**
- **The second names the command, not the version.** That is the git commit convention's
  form for any regenerated lockfile: the versions are in the diff and the command that
  resolved them is not, so naming the version here would restate the commit above and lose
  the only thing a reader cannot recover.
- The bump commit is also the declaration that the release is ready. Reading it in a log
  means the release is imminent, so do not bump a version to keep a branch tidy.

## When the bump turns out not to be last

Work sometimes lands after the bump — a defect noticed late, a branch that was still open.
**Whether to reorder is decided by one thing: has the bump already merged into the release
trunk?**

| The bump | The fix found afterwards |
| :-- | :-- |
| Still on its own branch | **Reorder.** Merge the fix first, rebase the bump so it closes the line |
| Already merged into the release trunk | **Merge it on top. Do not rewrite** |

- The second is not an exception granted reluctantly. Rewriting a trunk that other work is
  already based on is irreversible, and the ordering is not worth that price.
- The bump then is not literally the last commit on the trunk, and that is accepted. **What
  the rule protects is that nothing unfinished gets published** — not the shape of the log.
  A fix merged on top before the publish satisfies that completely.

## What the release takes in is settled before this convention starts

**This one orders the commits a release makes about itself** — its own version, and the
lockfile that follows it. Everything the release carries arrived before that, and how it
arrived is not decided here.

- **Moving the dependency versions a release takes in belongs to `hoc-npm-raise-deps`**, which
  settles what a raise compares against, how each move is written down, and where its single
  install sits.
- What is left once that pass has run — an advisory still in the report, an install script
  nobody has decided about — belongs to `hoc-npm-vulnerability` and `hoc-npm-install-scripts`
  respectively.
