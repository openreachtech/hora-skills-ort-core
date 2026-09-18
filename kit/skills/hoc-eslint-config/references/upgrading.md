# Upgrading the shared config

The interview a major bump requires, and the audits that follow any bump.
Referenced from `SKILL.md`.

## The interview

**A major bump is presented before it is acted on.** What the release demands of the local
config is not in the version number, and the five findings below are what make it knowable.
Gather all five, present them, and stop.

### 1. What newly fires

Capture the whole report machine-readable, and group it **by rule and by message** — one
rule can carry several distinct complaints, and they are different pieces of work. For each
group give the count and the number of files.

**The spread matters as much as the count.** Violations of one kind gathered in four files
are a single sweep; the same number spread across six files, one or two each, is six
separate reads. A table of counts alone hides that.

### 2. What reaches a published surface

Cross the groups against what the package exports. A violation inside a module nobody
imports is a free change; the same violation on an exported class is a decision about
consumers, and it belongs in the report as such.

### 3. What the release now carries itself

The shared config grows. A block the local file has been restating may now be in the
release, and keeping the local copy is then a duplicate that will drift.

**Two settings can hide each other, so probe coupled ones together.** A declaration made
without a file pattern applies to the whole tree, including files an earlier block had
already configured differently — and where a second local block exists to repair exactly
that, each of the two looks necessary when removed alone. Removing either one on its own
leaves the report non-empty; removing both leaves it clean. Probing one at a time concludes
that both are needed, which is the opposite of the truth.

### 4. What the release stopped requiring

**A consolidation upstream leaves packages behind.** Where the shared config once needed
several narrow plugins and now needs one that subsumes them, the narrow ones stay in the
manifest with nothing importing them.

They are found in the lockfile rather than by reading code: a package requested by the root
alone, and imported nowhere, is orphaned. Report it — removing it needs another install,
which the run may not be able to perform.

### 5. Which standing relaxations have gone quiet

See the audit below. In the interview it is a finding; after a minor bump it is work the run
does on its own.

## Auditing a standing relaxation

**Remove it, measure, and let the count decide.** A relaxation that reports nothing once
removed is no longer holding anything back, and what remains of it is the exemption itself.

Two things make this measurement lie.

### Remove the whole line, cleanly

**A line deleted from the middle of a list can leave the whitespace around it behind**, and
the formatting rules then report the artefact. The measurement comes back non-empty, the
relaxation looks necessary, and the reason has nothing to do with the rule under test.

Where a removal leaves a blank line doubled or a trailing separator stranded, close it up
before measuring. A count that includes a formatting complaint about the config file itself
is not a count of anything.

### Probe coupled settings together

The same trap as the interview's third finding. Where two entries exist because one
compensates for the other, neither can be measured alone.

## The count comment beside a relaxation

**A number written beside a turned-off rule records how many violations were outstanding
when it was turned off.** It is not a target and not a limit — it is a note saying how much
work the exemption is standing in for.

An upgrade can clear it. Where the rule now reports nothing, the comment is describing a
debt that has been paid, and it goes out with the exemption it annotated. Where the rule
still reports, the number is worth correcting to what it now is: a stale count reads as
though nothing has moved.

## What a bump costs the manifest

**The version and the lockfile are two commits, not one.** The manifest states the intent;
the lockfile records what the install resolved. Splitting them lets a reader see the
resolution separately from the decision, and the lockfile commit takes whatever subject the
repository's history already uses for it.

**Confirm the whole chain is available before setting the version.** The named package being
published where the install will look for it says nothing about its dependencies: a rule-set
package or a plugin one level down can be the one that is missing, or the one the release-age
cooldown declines. Check each package the target version requires, not only the target.
