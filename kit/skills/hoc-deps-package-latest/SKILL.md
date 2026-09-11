---
name: hoc-deps-package-latest
description: "How a project's declared dependency versions are brought up to what the registry publishes — which side of the project the comparison reads, why a version move is written into the manifest rather than updated into place, and where the pass's single install and one lockfile commit sit. Use when raising dependency versions or comparing them against the registry. Resolving an advisory belongs to the vulnerability convention, and an install script to the install-scripts convention."
---

# Deps Package Latest

Bringing a project's dependencies up to date is one pass with three parts: **read what the
registry publishes, write each move down, and resolve the tree once at the end.** The parts
are in that order for a reason, and most of what goes wrong here is a part taken out of it.

The pass moves **declared versions**. Resolving an advisory is a different errand that uses
some of the same mechanisms — see `hoc-npm-vulnerability`. Deciding an install script a
raise happens to bring in belongs to `hoc-npm-install-scripts`.

## The comparison reads the declared range, never the lockfile

**Compare what the manifest declares against what the registry publishes.** A comparison
that reads the lockfile instead answers a different question and usually answers it
`up to date`.

The reason is that **an install which is already satisfied does not re-resolve.** A caret
range permits everything up to the next major, so a newer release inside it changes nothing
about whether the lockfile is valid — and the lockfile therefore keeps the version it
pinned, indefinitely, while the range it was resolved from has long since grown past it.

Measured across thirty-three declared entries: four sat below a version their own caret
already permitted, and in every one of the four the lockfile held the older release.

- So the interesting state is not "the declaration forbids the new version" but **"the
  declaration permits it and nothing has gone and fetched it."** That state is invisible
  from the lockfile and invisible from an install that reports success.
- Split the result three ways, because the three take different work: entries already at
  the latest release, entries behind a release their range permits, and entries behind a
  major their range excludes. **Only the middle group is this pass.** A major is a decision
  of its own, one per package, and grouping several into one pass hides which of them broke
  something.

## Raising a floor is a record, not an upgrade

**Nothing resolves differently because a floor moved.** The caret already permitted the
release, so raising `^1.2.3` to `^1.3.0` changes no tree. What it changes is what the
project says about itself.

- A floor states **the version the project was last known to work against.** Left behind,
  it stops saying that: a reader cannot tell a version that was tested from one that merely
  happens to resolve today.
- This is why the pass is worth running even when nothing is broken, and why "it already
  works" is not an argument against it.

## A version move is written down, never updated into place

**A command that moves a version without touching the manifest is not the route.** Package
managers offer one — it walks the declared ranges and refreshes the lockfile to the newest
release each one permits — and it does the job, which is the trap.

**What it leaves behind is a lockfile and no reason.** The manifest is unchanged, so nothing
in it says why that version is where it is, and the next reader has a pin they cannot
account for and cannot safely move.

So each move is written as text somebody chose:

| The move | Where it is written |
| :-- | :-- |
| A direct dependency | Its declared range in the manifest, raised |
| A dependency the project never asked for | An override naming that package |

- **This holds even where the parent's declared range already permits the newer release.**
  That the range permits it is the reason an implicit update *would* work; it is not a
  reason to use one. The override then does two things: it moves the lockfile off the
  release it had pinned, and it keeps it off.
- The override's mechanics — scoping a key to one major line, and checking that the version
  named clears whatever release-age quarantine the project keeps — belong to
  `hoc-npm-vulnerability`.

## An override reaches every copy, and the call site is the test

**An override applies at every depth, to every copy, whatever each consumer declared.** A
consumer whose range excludes the version named still gets it.

That is worth checking and it is not automatically a break. **What decides is the API the
consumer actually calls, not the range it declares.**

Measured: a top-level override was written at `^4` for a package whose one reported copy
arrived through a consumer declaring `^4.3.0`. A second consumer further down declared `^3`
and had been resolving a 3.x release; the override lifted that copy across the major as
well, and the 3.x subtree left the lockfile — four packages removed, the package deduped to
one copy. It did not break. The consumer made a single call, to a member the new major
still provides, and the member the major had removed was never reached.

- So the declared range is **the flag, not the verdict.** It says a consumer is now outside
  what it asked for, which is the signal to go and look; the look is at the consumer's
  source, for what it calls and whether the new major still answers.
- Where the call site does not survive, the override is scoped to the major line that
  reported the problem, and the out-of-range consumer keeps the release it asked for.
- **Do not settle this from the package's reputation or from the size of the version jump.**
  A major that removed nothing the consumer uses is safe, and a minor that changed a default
  it depends on is not.

## One install, one lockfile commit, at the end

**The lockfile is not committed until the pass is finished.** However many declared versions
move, and however many commits they take, there is **one** install and **one** lockfile
commit, and it is last.

The alternative — a lockfile commit beside each declared version — fails twice:

- It needs an install per package, since a lockfile committed beside a change has to be the
  lockfile that change produces.
- It needs the lockfile's diff split per package, **and that cannot be done.** A lockfile is
  one resolution, not a sum of parts: raising three versions produces a single tree in which
  the three are already entangled with whatever else moved underneath them.

So the shape of the pass is N commits that move declared versions, then one that records the
resolution:

```
Update alpha-package version to 1.3.0 in <manifest>
Update beta-package version to 2.7.1 in <manifest>
Update gamma-package version to 4.0.2 in <manifest>
<the single lockfile commit>
```

- **The intermediate commits carry no matching lockfile, and that is the shape rather than
  an oversight.** A reader checking one of them out finds a manifest ahead of its lockfile,
  which is the same state the project sits in whenever a range is widened.
- The lockfile commit's own subject names the command that produced it, not the versions:
  `hoc-git-commit` settles its wording, and the same convention keeps generated artefacts
  out of the commits that carry hand-written source.
- Concentrating the install into one run at the end is the same shape a release takes when
  its version bump is left until last, and the same shape the install-scripts gate takes
  when its settings go in before the install they govern.

## A raise can add an install script

**A version raise is a way new packages enter the tree**, including packages that carry an
install script nobody has decided about. A minor bump is enough: one was measured pulling in
a native package as a regular — not optional — dependency of something two levels down.

- **Settling it is part of this pass.** Leaving it means every later install reports a
  decision outstanding, and a project whose gate is strict stops instead.
- **How to settle it is not this convention's** — `hoc-npm-install-scripts` covers reviewing
  the script, denying or approving it, and what the record looks like.

## Read the deprecation flag on the version the project declares

**A package's latest release says nothing about whether the release this project declares is
deprecated.** They are different versions, and the flag is per version.

Measured: a sweep reading the flag off each package's latest release reported none
deprecated at all, while an install printed a deprecation notice for one of the versions the
project actually declared. A second sweep, reading the flag off each resolved version
instead, found that one — and established it was the only one.

- A deprecation notice **prints and installs anyway**, so it does not announce itself except
  in install output nobody keeps.
- What it is evidence for is a major that has been left undecided: the line the project sits
  on has stopped being supported. It does not make the major this pass's work, but it ranks
  that one ahead of the others still waiting.
- **A package at its own latest release can still be a dead end.** A release years old and a
  release from this week both report "current" against a comparison that only asks whether
  anything newer exists. Where the answer matters, read the publish date alongside the
  version.
