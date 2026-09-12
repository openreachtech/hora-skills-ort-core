---
name: hoc-npm-raise-deps
description: "How a project's declared dependency versions are raised to the newest release each declared range already permits — which side of the project the comparison reads, why a version move is written into the manifest rather than updated into place, and where the pass's single install and one lockfile commit sit. A major a range excludes is a decision of its own and not this pass. Use when raising dependency versions or comparing them against the registry. Resolving an advisory belongs to the vulnerability convention, and an install script to the install-scripts convention."
---

# npm Raise Deps

Raising a project's declared dependency versions is one pass with three parts: **read what
the registry publishes, write each move down, and resolve the tree once at the end.** The
parts are in that order for a reason, and most of what goes wrong here is a part taken out
of it.

**The registry is what the pass reads, not where it moves to.** A declared range is the
ceiling: the pass takes each entry to the newest release its own range already permits, and
a major that range excludes is left for a decision of its own. A pass that ends is therefore
not a project at the latest of everything.

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

So each move is written as text somebody chose: **the declared range in the manifest,
raised.**

- **A dependency the project never asked for is not written down here.** It moves when the
  pass's single install re-resolves the tree, under whatever range its own parent declares,
  and nothing in this manifest has an opinion about it.

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

## An advisory the pass did not clear is the vulnerability convention's

**Raising declared versions is itself the ordinary fix for an advisory**, so the pass leaves
the audit quieter than it found it far more often than it adds to it. The report worth reading
is the one taken after the pass's single install, and what matters in it is **what remains**.

What remains is, by construction, what raising a declaration could not reach: a package the
project never asked for, sitting at a version no range of the project's own controls. That is
the case the vulnerability convention opens with, and the override is its instrument.

- **Read the audit after the install, never before.** Beforehand it describes the tree the
  pass is about to replace, and every finding the raises are about to clear is still in it.
- **A finding that remains does not hold up the pass.** The declared versions moved and the
  lockfile records the resolution; what is left is a separate errand against the same tree.
- **How to settle it is not this convention's** — `hoc-npm-vulnerability` covers the override,
  scoping its key, and checking that the version it names clears the quarantine.

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
