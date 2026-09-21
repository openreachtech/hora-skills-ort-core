---
name: hoc-manifest-pattern
description: "The manifest pattern, also called the super strategy pattern: a structure keeps one shared object declaring its wiring, and everything the structure is made of may take that object. Covers why an interface taking it stays the same as the structure grows, the line between it and the per-call objects a member has to justify taking, and taking it without holding it. Use when deciding what a class receives, or when a collector starts naming each member's own parameters."
---

# Manifest Pattern

Also called the **super strategy pattern**. The two names are one thing.

A structure — a server, an application, a subsystem — keeps **one object declaring its shared
wiring**. It is built once at boot and lives as long as the structure does. Everything the
structure is made of may take that object, and does.

**The role travels under more than one name.** A class holding it takes `~Manifest` or `~Engine`
as its main suffix, and whatever it is called, the parameter is named after that suffix. What
names the pattern is what the object does: it **declares** the structure's wiring and holds no
logic of its own.

## Grand principle: anything in the structure may take the manifest

**A class belonging to a structure may declare the manifest in its interface, and needs no reason
for it.** Receiving it is not a dependency being smuggled in. It is access to the object the
structure already shares with everything inside it.

```javascript
// OK: every member of the family takes the same thing
static isAcceptable ({
  manifest,
}) {
  return manifest.isProduction()
}

static async createAsync ({
  manifest,
}) {
  const threshold = this.resolveThreshold({
    manifest,
  })

  return this.create({
    threshold,
  })
}
```

- **The manifest is the parameter, not the values inside it.** A member reads what it alone needs
  — one reads its config, another its error declarations, a third its environment — and nothing
  above it has to know which.
- **What a class may hold is a separate question**, governed by the class design principles
  convention. This skill is about what a class may be handed.

## Why the interface is uniform

**Where every member takes the manifest, whoever collects them treats them all alike, and adding
a member changes nothing at the call site.**

```javascript
// OK: the collector lists candidates; it knows nothing of what each one needs
const acceptableCtors = MemberCtors.filter(it =>
  it.isAcceptable({
    manifest: this,
  })
)

const memberPromises = acceptableCtors.map(it =>
  it.createAsync({
    manifest: this,
  })
)

return Promise.all(memberPromises)
```

```javascript
// NG: narrowed to "just the values each one needs", so the collector knows all of them
const alphaMembers = this.config.alphaLimit
  ? [
    AlphaMember.create({
      threshold: this.config.alphaLimit,
      reporter: this.reporterHash[AlphaMember.reporterName],
    }),
  ]
  : []

const betaMembers = this.env.isProduction()
  ? [
    BetaMember.create({
      reporter: this.reporterHash[BetaMember.reporterName],
    }),
  ]
  : []
```

**The narrowed form looks like the smaller dependency and is the larger one.** Every member's
parameter list has moved into the collector, so a new member edits shared code — the very thing
the strategy pattern exists to stop. This is where the *super* in the other name earns itself: an
ordinary strategy hands each variant its own arguments, and this one hands every variant the same
structure.

- **Whether a member applies is the member's own question**, for the same reason. Where the
  collector decides, it grows a branch per member; where the member declares it, the collector
  grows a line.

## The line: a manifest is not a per-call object

**What may be passed freely is the structure's own object. What arrives with a single call is
not.** A validation context, a request, a response, a transaction, a session — each belongs to
one call rather than to the structure, and taking one is a real dependency that needs a reason.

The test is one question: **would this be the same object on the next call?**

| Object | Same next call | May be taken freely |
| :-- | :-- | :-- |
| The manifest | yes — built once at boot | yes |
| A call's context, a request, a job payload | no — one per call | only where the member's own work needs it |

```javascript
// NG: the base takes a per-call object so that it can report through it
reportError ({
  context,
  value = null,
}) {
  context.reportError(
    this.ErrorCtor.create({
      value,
    })
  )
}

// OK: the base builds the value; handing it to a per-call object belongs to whoever holds one
createError ({
  value = null,
}) {
  return this.ErrorCtor.create({
    value,
  })
}
```

- **The symptom is a base that knows a channel it has no use for.** A member whose own work reads
  the per-call object does take it — that is its work, not a channel it was handed.

## Taking the manifest is not holding it

**A static member taking the manifest does not put it on the instance.** The factory reads what
it needs and hands the constructor plain values; the instance holds those and never sees the
manifest again.

- **An instance holding the manifest is a different decision, and usually the wrong one.** It
  widens what the instance depends on for its whole life, where the factory needed it for one
  statement.
- **Where it is genuinely instance state** — a builder that must reach the manifest on every call
  — holding it is correct, and the class design principles convention governs it.

## Symptoms of getting it wrong

Each of these reads as a different defect and has one cause: the manifest was treated as an
ordinary dependency.

| Symptom | What actually happened |
| :-- | :-- |
| A collector's method grows a branch per member | The members were never asked whether they apply |
| A collector names each member's constructor parameters | The members were handed values instead of the manifest |
| The shared object grows one policy member per member of a family | A question the member should answer was answered for it, because the manifest held the data |
| A caller reaches through a field of the manifest to a method behind it | The manifest was treated as a bag of collaborators instead of an interface |

## Reading the manifest is one hop, and one hop only

**Taking the manifest freely is not licence to reach through it.** `manifest.env` is the
interface the structure declares; `manifest.env.isProduction()` is the caller reaching past that
interface to a collaborator behind it, and the general rule against reaching through objects
applies to it in full. The manifest is not an exception to that rule — it is the object the rule
is measured from.

```javascript
// NG: the caller now depends on env, and on what env offers
return manifest.env.isProduction()

// OK: the manifest answers for itself
return manifest.isProduction()
```

- **The fix is a method on the manifest, not a shorter chain at the call site.** `Manifest`
  grows `#isProduction()`, which delegates to whatever it holds. Adding a member is a cheap
  direction and it compounds: once the method exists, the next caller reaches it without ever
  meeting the field behind it.
- **A method call on a value is not this.** `it.name.toUpperCase()` calls a built-in on the value
  a property gave back; nothing behind the property is being reached for. What this rule turns
  away is a hop to a **collaborator** whose behaviour is a decision of its own.
- **The chop-down rule counts the same hops**, under "one property per line in a property chain"
  in the coding-styles convention. A line that
  has to be chopped to satisfy it is usually a line that should not exist.

### Where the manifest is a plain object

**A manifest that cannot grow a method is a design mistake**, and the correction is to make it an
instance of a class that can. Where that is cheap, do it.

- **Where it is not, measure before reaching for it.** A manifest built as a plain object at boot
  may be read from everywhere, and giving it a class touches every one of those sites.
- **Survey the reach first, and where the correction turns out to be a large-scale change, stop
  and report it** as a refactoring target rather than starting one. The review convention has no
  `WARNING`; the level this takes there is `MEDIUM` — a departure that will mislead the next
  reader, with no defect behind it.
- The report names the manifest, the sites that reach through it, and the members it would have
  to grow. Deciding to spend that is not this convention's call.
