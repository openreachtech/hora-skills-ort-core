---
name: hoc-npm-adopt-tool
description: "Whether a third-party tool may be brought into a project at all — what counts as a reason to reach for one, the audit its tarball takes before anybody installs it, and the wrapper the project meets it through so that only one module ever names it. Use when a package is proposed for a problem the project could close by other means. What happens once it is in belongs to the dependency conventions beside this one."
---

# Adopting a third-party tool

**A dependency the product reads and a tool the workflow runs are two different decisions.** The
conventions beside this one settle what happens to a package already in the manifest: raising its
version, approving its install scripts, answering an advisory against it. This one settles the
question before any of that — whether to take it on.

## The remedy is no larger than the fault

**A tool is machinery, and machinery is answerable for its size.** Where the fault is closed by
writing a rule, or by writing a file, that is the remedy — a compiler introduced to prevent a
kind of mistake is a house shelled to deal with an insect.

- **The volume of work a tool would save is not a reason to adopt it.** Work that scales linearly
  with the number of things is work somebody does, and it gets done. What a tool buys is not the
  typing.
- **Verification is the reason that survives**, and only where the project does not already live
  without it. Where nothing checks that a declaration matches its implementation, that is a real
  gap — but the same project accepts unchecked documentation, unchecked comments and unchecked
  prose, and singling out one of them earns no machinery.
- **The proposal that widens is the proposal that has outgrown its fault.** Where the second
  reason offered is that the tool would also settle something in the next repository, and the next,
  the case being made is no longer about the defect that started it.

**A proposal that lifts an ongoing obligation from whoever is making it deserves the closest
reading.** Machinery that removes a duty to keep two things in step removes it from the proposer
first, and a case built on that will be built out of the costs somebody else bears.

## Before it is installed

**Popularity is not evidence.** A star count and a download count are numbers anybody can raise,
and raising them is within reach of an adversary with means. Neither settles whether the code is
safe to run.

**The tarball is read.** Not the repository it claims to be built from — the artefact the registry
will hand over:

| What to look for | Why |
| :-- | :-- |
| Code that does not correspond to what the package says it does | The published artefact and the source are two different things |
| Any outbound network access | A tool with no reason to reach the network reaching it is the finding |
| `install` scripts, and what they run | They execute before anybody has read a line |
| Files the package has no reason to ship | Credentials harvesters arrive as ordinary-looking extras |

**Anything doubtful stops the adoption and goes to a person.** The finding is reported with what
was read and where; it is not weighed and waved through.

**Where the tool is small enough that writing it would be comparable, measure before installing.**
Read what it actually does, and set that against what the project would have to write. A
dependency taken on for fifty lines is a dependency the project now carries for as long as it
lives.

## The wrapper

**Where a tool is adopted, exactly one module names it.** Its `import` lives there and every call
into it is made from there, so the tool's surface never reaches the rest of the project.

**The reason is the lint configuration.** A third-party interface breaks naming conventions
routinely, and a project that answers each breach by adjusting a rule ends up with a rule set
shaped by somebody else's naming. Confining the tool to one module confines the exception with it:
the rule is turned off for that file by name, and nothing wider is relaxed.

- **The exception leaves with the module.** Once that module is extracted into a package of its
  own, the lint exception goes with it and the application is left with neither.

### The interface is renamed, not relayed

**The wrapper's surface is written in the project's own vocabulary**, not in the tool's. An
abbreviation the tool chose is expanded, and a name that says nothing is replaced by one that
does — **parameter names included.**

```
Tool          (from: Date, to: Date)
Wrapper       ({ startedAt: Date, endedAt: Date })

Tool          ({ msg: string })
Wrapper       ({ message: string })
```

This is what makes a replacement a change to the wrapper. A surface relaying the tool's own names
has published them, and every call site then has to move when the tool does.

### Two layers by default, three where the tool will be replaced

**The ordinary wrapper is two layers**: the module the project calls, and the tool behind it.

**Three layers are for a tool that will be swapped** — one with several equivalents in the field,
or one whose replacement is evident from the start:

| Layer | What it holds |
| :-- | :-- |
| The face | What the project calls. Holds a client and delegates to it |
| The abstract client | Declares the members the face uses, and implements none of them |
| The concrete client | The only class that imports the tool |

The selection is a single expression in the face's factory naming the concrete client, so a
replacement adds one class and edits one line.

**The structure answers to the rule above it.** Three layers where a tool has no equivalent is the
house shelled for the insect a second time — this time by the wrapper rather than by the tool.

## Out of scope

- **Moving the version of a package already taken on.** That belongs to the dependency-raising
  convention
- **Approving or denying an install script.** That belongs to the install-scripts convention, and
  reading one here is part of deciding whether to adopt at all, not part of approving it
- **Answering an advisory.** That belongs to the vulnerability convention
