# Pre-publish audit

What to check before a package goes out, and why each check exists. Referenced from
`SKILL.md`.

## What lint and tests cannot see

Run on a tree where the linter was clean, every test passed and the audit reported nothing,
this audit still found four defects — every one of them something a consumer would meet in
their first minute with the package.

That is the shape of the risk. **Lint and tests examine the repository; a consumer receives
an artefact and a document.** Nothing in the repository's own checks reads either one as the
consumer reads it, so the audit has to.

## Read the artefact, not the repository

**Build the distributable and look inside it.** The package declares what ships as an
allowlist, and an allowlist is only as good as its contents — one wrong entry ships the
wrong thing, or leaves out something needed.

- Count the files and name them. **Tests, configs and fixtures must not be there**: they
  bloat the install, drag in development dependencies that are not declared, and can leak
  whatever the fixtures hold. The jest convention explains why the allowlist can be a
  one-liner at all; this check confirms it actually was.
- **Install the built artefact into a separate project and run the type checker there.** A
  type declaration that resolves inside the repository can fail to resolve for a consumer,
  because the path it is found by is different — the repository finds it by location, the
  consumer through the manifest's type field.
- **Import it and assert the shape.** Read the exports back — the default, each named one,
  and the sizes of whatever collections they hold — against what the package claims. A
  runtime import is the only check that sees what a consumer's code will see.

## Install it the way a consumer's CI will

- **Run the lockfile install under the project's own quarantine setting.** A version pinned
  recently can be newer than the quarantine allows, and the question is whether continuous
  integration still works. It does, for the reason the vulnerability convention gives, but
  confirm it rather than reason about it.
- **Confirm no install script produced output.** Run the install with scripts in the
  foreground and read the result: zero lines means the gate held. This is the observable
  form of the install-scripts convention's guarantee.

## Compare siblings released together

Where two or more packages are released as a set, **diff their file inventories against
each other.** They should differ only where they must — in the domain each one covers —
and be identical everywhere else.

- The comparison finds what a single-package review cannot: a file added to one and
  forgotten in the other, a config that drifted, an untracked file in one working tree.
- It also finds the reverse — a file that is byte-identical in both when it should have been
  specialised. Both directions are worth reading.

## Read the README as a user would

**Follow the document's own instructions and check each one against reality.** This is the
check that finds the most, and the only way to run it is to read.

Three failures found by doing exactly that, on documents nobody had touched in the release:

- **An install instruction pointing at the wrong place.** The document told the reader to
  configure one registry; the release actually went to another. The two had drifted two
  versions apart, so a reader following the document would have installed something old
  while believing they had the latest.
- **A paragraph copied from a sibling package.** True of the sibling, false here — and
  present in every translation of the document, because the copy was made before the
  translations were. **A document copied between sibling packages carries the sibling's
  claims**, and those claims are about counts, capabilities and behaviour that differ.
- **An example teaching the wrong default.** The document's own notation shows a default
  value in a comment beside the setting, and the comment said the opposite of the real
  default. For a package whose purpose is to teach defaults, this is the worst defect
  available, and no test can see it.

Also read for what the examples **omit**. An example leaving out half a setting's options
teaches that the other half does not exist.

## Scan for the conventions no linter enforces

A project holds rules its linter does not check — a forbidden character class, a notation
for types, a marker that must not survive a release, a file whose content is frozen by
policy. **Enumerate them and grep.**

- These are cheap to check and easy to forget precisely because nothing fails when they are
  violated. The release is the last point at which forgetting them is still recoverable.
- Include the rules about documents that must not be edited. A file whose whole purpose is
  to record a frozen state is one an automated tidy-up will happily update.
