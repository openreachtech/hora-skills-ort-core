---
name: hoc-eslint-config
description: "Conventions and procedure for `eslint.config.js` and the shared config package it spreads. Covers bumping that package — what may be fixed and committed without a decision, and what has to stop for one — narrowing a rule's options to a set of files, and when a rule may be relaxed rather than the code changed. Scope is settled by the package name carrying `eslint`. Dependencies in general belong to the npm conventions."
---

# ESLint Config

Conventions for the local `eslint.config.js`, for the shared config package it spreads, and
for the run that follows bumping that package.

**Scope is settled by the package name.** A package whose name carries `eslint` is this
skill's — the shared config, the rule-set packages under it, the plugins. Everything else is
the npm conventions'. Which of the shared config's rules apply inside a particular subject
is that subject's own convention.

- Narrowing a rule's options to a set of files:
  [rule-option-subtraction.md](./references/rule-option-subtraction.md)
- The upgrade's interview and its audits: [upgrading.md](./references/upgrading.md)

## Choosing the version

| Asked for | Take |
| :-- | :-- |
| A version | That version |
| Nothing | The newest version the release-age cooldown admits |
| A version whose major differs from what is installed | **Nothing yet. Interview first** |

**A cooldown makes the newest published version and the newest installable version two
different things.** Where a repository holds a minimum release age, `latest` means the
newest the cooldown admits, and reading the dist-tag alone will name a version the install
then declines. Compare the candidate's publish date against the cooldown before committing
to it, and do the same for every package in the chain — a dependency of the config can be
the one that is too young.

Wanting a version the cooldown excludes is a real case, and its handling — an exclusion at
install time rather than an edit to the setting — belongs to `hoc-npm-vulnerability`.

**A major bump is not run autonomously.** What it demands of the local config is not
knowable from the version number, and the interview in
[upgrading.md](./references/upgrading.md) is what establishes it. Present the findings and
stop.

## What may be committed without a decision

**Four kinds, and they share one property: a measurement settles whether the change is
right.** Nothing here needs a name chosen, a behaviour weighed, or a shape judged.

| Kind | What settles it |
| :-- | :-- |
| A violation the linter's own fixer resolves | The fixer, checked — see below |
| A local block the upgraded shared config now carries itself | Removing it and finding the report still empty |
| A standing relaxation that now reports nothing | Removing it and finding the report still empty |
| A violation-count comment beside a relaxation that has gone stale | It follows from the row above |

### The fixer's output is read before it is committed

**Applying the fixer is the machine's; trusting it blindly is not.** Fixers do occasionally
misbehave — reflowing lines they were not asked to touch, or two of them undoing each
other's work across passes.

So capture the report **before** applying the fixer, then check that every hunk it produced
sits at a location the report named. A hunk anywhere else is backed out and reported rather
than explained. Apply only what the fixer applies; never take a rule's *suggestions*, which
are offered precisely because they are not safe to apply unattended.

That check is itself a comparison of two lists, so it adds no judgment.

## What stops for a decision

| Kind | What the decision is |
| :-- | :-- |
| A denied identifier | The replacement name, or which words to exempt — both are choices |
| A value the ruleset denies | Whether the module treats it as a value or was merely being loose |
| A rewrite that changes behaviour | Whether every caller survives the new semantics |
| A rule about a class's or a member's shape | Whether the shape is right and the rule mis-reads it |
| Anything reaching a published surface | What consumers rely on, which the tests do not fully hold |

**Report these grouped by rule, with the count and the number of files for each**, and say
for each group whether a code change or a relaxation is indicated. A group of one file with
eight violations and a group of six files with seven are different pieces of work, and the
spread is what tells them apart.

## The run

1. Set the version in `package.json`. Commit.
2. Install. **This step may not be the machine's to run** — where it is not, hand over and
   resume once the lockfile has moved.
3. Commit the lockfile, with the subject the repository's history already uses for it.
4. Capture the whole report, machine-readable, and keep it. Everything below compares
   against it.
5. Apply the fixer, check its hunks against the report, and commit what survives.
6. Audit the local config against the upgraded shared config, and audit every standing
   relaxation. Commit each removal a measurement supports.
7. Report what is left.

**Cheap commits go underneath.** The fixer's output and the config removals cost a reader
nothing to skim, so they land before whatever judgment produces, and the diff reads from
the newest side without wading through them.

**Commits that touch the config go last among the code.** A relaxation is answerable for
being the thing that remained, and the order is what shows it was.

## Relaxing the config is the last resort

**A rule that fires is answered by changing the code.** Editing `eslint.config.js` comes
last, and only once no change to the code will do. A relaxation hides, for as long as it
stands, whatever else that rule would have caught in that scope — a cost paid by every
later reader rather than by whoever wrote it.

### Which way to go

**Relax only where the code's shape is already right and the rule is wrong about it.** Where
the rule has a point, the code changes, even when that is the larger change.

The two cases read almost alike and part on one question: *would the code be worse after the
change the rule is asking for?*

| The code | Answer |
| :-- | :-- |
| Is shaped as it should be, and the rule mis-reads it | Narrow the rule to that file |
| Has a shape the rule is right to refuse | Change the code |

A class that is static because it holds a stateless transformation is the first case: a rule
against static classes describes a smell that is not present, and giving the class instance
state to satisfy it would leave it worse.

A constructor that calls one of its own methods is the second. The rule is pointing at
composition done inline, and naming that step — extracting it into a member of its own,
leaving the constructor with its assignment — improves the code whichever way the rule had
gone.

**A published surface is not reshaped to satisfy a rule.** Where the only code change that
clears a rule would alter what callers rely on — a constructor's parameters, a member other
repositories override, a factory's contract — narrow the rule instead.

**That narrowing is waiting on a release, not on nothing.** A surface this repository publishes
is still its own, and the next major version is where its shape may change. What puts a
relaxation beyond reach is a shape decided somewhere else entirely — a property name a library
gives the objects it returns, an argument a library hands over to be rewritten in place. Reading
"cannot break the callers now" as "cannot ever change" files a relaxation under the wrong kind,
and it then stops being looked at.

What a change may do to a declaration that already has callers is settled by
`hoc-retake-declaration`, whatever the reason for making it.

**Measure the surface before calling a restructure internal.** A change that looks confined
can drop a subclass's override silently, and a suite exercising only the base class passes
anyway. Construct the object every documented way, call the members a consumer would, and
subclass it overriding what a consumer would override.

**Check that the ruleset does not also deny the alternative.** A set can close both a value
and its usual workaround, leaving no third spelling: one rule against writing a bare
`undefined` and another against writing `void 0` between them rule out every way of saying
the same thing. Establish that an alternative exists before concluding the code can absorb
the fix.

**Measure the floor the rule leaves that code before promising the code can clear it.** A
threshold rule counts what the signature carries as well as what the body does — cyclomatic
complexity counts every defaulted parameter — so a method declaring eight of them stands at nine
before its body is read. Where the floor already reaches the threshold, changing the code is not
one of the two options and the choice was never between them.

The same measurement says what a rewrite can win. Measured on a method whose conditional chain
was replaced by a table of the same conditions: the table cost nothing at all, every point above
the floor had come from the branching, and all of it came off.

## Verifying a change to the config

**A scoped relaxation is checked on both sides of its boundary, and against the rest of the
same rule.** Three questions, and all three have to be answered:

1. Inside the scope, has the relaxed rule stopped firing?
2. Outside it, does the same rule still fire?
3. Inside the scope, do the rule's *other* entries still fire?

**The third is the one that gets skipped**, and it separates subtracting one entry from
turning the whole rule off by accident. A predicate matching more than intended passes the
first two questions and fails only this one.

Linting a snippet under a chosen filename answers all three without committing anything:
feed the source on standard input and name the path it is to be judged as. Probe a path
inside the scope, a path outside it, and a violation of a neighbouring entry of the same
rule.

## How the file arranges what it keeps

**Two kinds of relaxation live in the file, and it says which is which by where they sit.** The
ones nothing in this repository can clear go above; the ones waiting on code here go below. Each
group opens with a `/* */` comment of its own — the first states the reason its blocks cannot
go, the second states what the blocks beneath it are waiting for.

Side by side and unlabelled the two read alike, and then neither gets revisited: a block that has
outlived its cause stays exactly as long as one that never had a way out.

- **The permanent group is closed by a marker**, rather than being left to end wherever the next
  block happens to begin. The marker says what belongs below the line, and **says so even when
  nothing does** — a group that announces its own emptiness is a state somebody can keep, where a
  file that simply stops says nothing about whether the emptiness was reached or never attempted.
- **A block that is kept carries, in its comment, why the shape has to be what it is** — not
  which rule is off, which the line beneath already says. Where alternatives were tried, the
  comment names them and what each one broke. That is what stops a later reader from
  "simplifying" the shape back into the one the rule fires on.
- **Run that investigation even where the answer is believed known.** Asked to confirm that a
  value had no substitute, the confirmation turned up a second reason nobody had stated: the
  obvious replacement failed not because the ruleset denied it, but because the value is compared
  against what arrives from outside, and a token private to the module cannot catch that. The
  first reason had been settled for a long time; the second is the one the comment needed.

## A block that covers several files

**Where one block gathers more than one file, its comment and its commit subject take wording
that fits all of them.** A comment naming the first file stops describing the block the
moment a second is added, and a reader then trusts a label that is no longer true.
