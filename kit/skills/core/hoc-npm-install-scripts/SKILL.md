---
name: hoc-npm-install-scripts
description: "How to handle the install scripts npm would run while installing dependencies — reviewing them, denying or approving each one, and the settings that enforce the gate. Use this skill when an install reports scripts not yet covered by `allowScripts`, when deciding whether a dependency's install script may be denied, when turning the gate on in a project, or when judging whether an upgrade introduces a new script."
---

# npm Install Scripts

A package's **install script** runs on the machine doing the install, with that machine's
permissions, before anybody has looked at it. npm can gate this: an install script runs
only for a package the project has approved, recorded in `allowScripts`. This skill covers
the gate — how it is switched on, how each package is decided, and the order those two
have to happen in.

## The gate is the defence, not the registry

**A version found to be malicious does not leave the registry.** A publish older than the
grace period is not removed, because removing it breaks every dependent. What gets applied
instead is a deprecation notice, and a deprecation notice **prints a warning and installs
anyway**.

Measured on a widely used package whose 1.x line carried a compromise, years after the
fact: every version inside the advisory's range still had its tarball, none had been
unpublished, and a dist-tag still pointed into the affected line.

- **"The bad version is gone" is never available as a premise.** The registry is not a place
  where dangerous versions expire.
- What can be stopped is **execution**. Fetching a tarball puts bytes on disk; running its
  install script is what hands over the machine. The gate governs the second, and only the
  second is the project's to govern.
- The gate also covers the interval **before** an advisory exists. An advisory is published
  after somebody notices; a package that was never approved was never running, notice or no
  notice.

## Deny is the default

**An install script is denied unless there is a demonstrated need for it.** Denial is the
resting state. Approval is the exception that has to earn itself.

- Reaching the decision means **denying the script and running the project's own checks** —
  its lint, its tests, whatever it runs to know it works. If those pass, the denial stands.
- **A package's reputation is not evidence.** Widely used, well maintained, and depended on
  by something the project needs are all true of packages whose install scripts do nothing
  this project uses. The question is not whether the package is trustworthy; it is whether
  the script is needed, and only running without it answers that.
- Approve only what actually broke, and say in the same commit what broke.

## Turn the list into a recommendation, package by package

npm reports the scripts it did not cover as **a list of names**:

```
npm warn install-scripts 2 packages have install scripts not yet covered by allowScripts:
npm warn install-scripts   alpha-package@2.3.3 (install: (install scripts present))
npm warn install-scripts   beta-package@1.12.2 (install: (install scripts present))
npm warn install-scripts
npm warn install-scripts Run `npm install-scripts ls` to review, or `npm install-scripts approve <pkg>` to allow.
```

Names alone are not a decision. **Turn the list into a per-package recommendation with its
reason attached** — denied because the checks pass without it, approved because this
specifically failed. Whoever reads the commit should not have to derive it again.

- **The record of the decision lives in `package.json`**, which is where npm reads it from.
  Do not keep a second copy of the list anywhere else, this skill included: a list that is
  not the one being enforced goes stale with nothing to reveal it.
- Where projects are generated from a boilerplate, the denials common to all of them belong
  in the boilerplate's own `package.json`, already recorded, so a new project starts gated.

## `--dry-run` answers "does this upgrade add a script?"

Before committing to an upgrade, the question is whether it introduces an install script
not already covered. **A dry-run install answers exactly that and changes nothing.**

```
npm install <pkg>@<version> --dry-run
```

Measured: no `node_modules` is created, `package.json` and `package-lock.json` are
untouched, and no script runs — while the warning listing uncovered scripts is still
printed.

- So the question is answerable **before** the upgrade lands, not after.
- **One caveat: a dry run resolves as though nothing were installed.** It reports the whole
  set for a fresh tree, not the delta against the tree that exists. Read it as "the scripts
  this dependency set has", and compare that against what is already covered.

## Configure the gate first; install afterwards

**The settings go in before the install they govern.** Not the other way round.

1. Turn on `strict-allow-scripts`, which upgrades the warning into a **stop**. Do not
   install.
2. Record `npm install-scripts deny <pkg>` for each package, against the tree already
   present.
3. Install once, with the gate in place.

- **It looks like a chicken-and-egg problem and is not one.** `deny` reads the installed
  tree, so with a tree present it succeeds even under the strict setting. The two appear to
  deadlock only in a directory that has never been installed — and a project in that state
  has nothing to gate yet.
- Measuring this in a fresh directory yields the opposite conclusion, and the opposite
  conclusion is wrong. **Where a measurement about ordering is taken decides its answer**;
  take it in a tree shaped like the one the rule will be applied to.
- **Recording a denial does not move `package-lock.json`.** The work is two commits — the
  setting, then the denials — with no lockfile commit between them.
- Installing once, after the settings are complete, is the same shape as concentrating the
  install into a single run at the end of a release.

## The npm configuration file is written `key = value`

Spaces on both sides of the `=`.

```
strict-allow-scripts = true
```

- **Do not use `npm config set` to write it.** It writes the pair without the spaces, and it
  rewrites the lines already in the file — so applying one setting through it reformats
  everything else.
- Edit the file directly. Where the command has already run, the file has to be reformatted
  by hand afterwards.
