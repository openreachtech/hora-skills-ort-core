---
name: hoc-npm-install
description: "How a package is added to a project with `npm install`: what is settled before the command runs, and what to do when it stops. Use before running `npm install <package>`, and when it ends in an error such as `ENOVERSIONS` or leaves something it did not complete. Whether to adopt the package, which manifest field holds it, and what release age, install scripts or advisories require belong to the adopt-tool, categorize, vulnerability and install-scripts conventions."
---

# npm Install

How a package is added to a project with `npm install`. The command is where several
conventions meet, and this one says in which order they are taken, and what a stop means.

## Settle three things before the command runs

- **Whether the package may be adopted at all.** See `/hoc-npm-adopt-tool`.
- **Which manifest field will hold it.** See `/hoc-npm-categorize-deps`.
- **Which version is wanted.** Decide the range the manifest will carry, and never write an
  unbounded one.

## A stop is read, not worked around

When `npm install` ends in an error or reports something it did not complete, **the stop is
the information.** The command is applying a protection the project turned on, and the way
out is the convention that owns that protection, never a flag that lifts it.

- **`--min-release-age=0` is never used, for any reason.** It lifts the release-age
  quarantine from every package the command touches. No situation, no urgency and no
  argument that it is safe here makes it allowed.
- Read the message to the end before running anything else.
- Find the convention that owns what the message names, and follow it. This convention does
  not restate that convention's rules.
- **Do not retry with any other flag to get past the stop.** A flag that lifts a protection
  lifts it from every package the command touches.

## Taking one held-back version as an exception

A version the quarantine holds back can be taken through a route narrower than switching the
quarantine off, and **it is open only after a safety check made in the same session.**
`--min-release-age-exclude` exempts the one package named, and the quarantine stays on every
other package. Where the version is wanted for an advisory's fix, `/hoc-npm-vulnerability`
says when this route is taken; otherwise the person who asked for the install decides.

```sh
npm install @openreachtech/<package> --min-release-age-exclude='@openreachtech/<package>'
```

- **The safety check is the diff between the version installed in the repository at that
  moment and the version about to be installed.** Read it in full, the changes to
  dependencies and to install scripts included, and look for malware and dangerous code.
  A check made in an earlier session, or reported by somebody else, does not count.
- **A package under the organization's own scope: the agent reads the diff itself.** Where
  it finds nothing dangerous, it runs the command without being asked.
- **A package of another party needs the user's strong consent.** The agent reads the diff,
  shows the user what it found, and asks. It runs the command only after the user has
  consented in so many words; silence, and a remark that merely refines the rule, are not
  consent. Where the user does not consent, the package is not installed past the quarantine:
  for an advisory's fix the user chooses the way forward under `/hoc-npm-vulnerability`, and
  otherwise the install waits until the release is old enough.
- **Where no version is installed, there is nothing to diff, and this route is not open.**
  The quarantine stands until the release is old enough.
- **Name the exact package, never the scope.** `'@openreachtech/*'` also exempts a version
  published from a compromised account without anyone's knowledge.
- **Only the named package is exempt.** Its own dependencies still follow the quarantine.

## Where this convention stops

Run the command to its end. What it still reports, and what it left uncompleted, is the
neighbour's.

- When a held-back version is taken as an advisory's fix, and an advisory left behind:
  `/hoc-npm-vulnerability`.
- An install script not yet covered by `allowScripts`: `/hoc-npm-install-scripts`.
- The lockfile commit the install produces: `/hoc-git-commit`.
