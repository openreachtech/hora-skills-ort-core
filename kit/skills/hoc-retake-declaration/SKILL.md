---
name: hoc-retake-declaration
description: "Conventions for redoing existing code without moving what its callers see. A retake claims no gain, so the declaration it comes out with is the one it went in with — the logic stays where callers already reach it, an added argument carries a default, and a defect found on the way is left alone. Use before reshaping a class or a member that already has callers, and to tell a retake from an update. The release that would admit a moved interface belongs to the npm publish convention."
---

# Retake a Declaration

A **retake** redoes what is already there — replacing something poor, hurried or a stopgap with
what should have been written — and **claims no gain beyond that**. That is what separates it from
an update, which carries a sound implementation forward and leaves it giving something it did not
give before.

The distinction is not a nuance of wording. It decides what the work is allowed to do.

## The declaration comes out as it went in

**A retake does not move the interface.** Whatever a caller could see before — which members
exist, what they take, what they give back, and what those values are — it sees afterwards,
unchanged.

**Where a change would move the interface, the work is not a retake.** It is a different piece of
work, wanting a version that admits it and a decision that it is worth making. Until that decision
exists, the shape callers see is fixed and the work is done inside it.

- **"Nobody will notice" is not the test.** The test is whether the declaration is the same one. A
  change nothing happens to depend on today is still a change to what may be depended on tomorrow,
  and the version is what tells consumers which they are holding.
- The release that admits a moved interface, and where its version bump sits among the commits,
  belong to the npm publish convention.

## Keep the logic where the callers already reach it

**When the shape has to grow — a constructor where there was none, instance members beside static
ones — the member that already has callers keeps its body, and the new shape calls it.** Never the
reverse.

Moving the body into the new member and leaving the old one delegating looks equivalent and is
not. The old member's behaviour is then produced by code written today, and nothing says that code
produces what the old body produced: it is a rewrite wearing the old name. Left alone, the old
member cannot have changed, because nothing in it changed.

```javascript
// The member that already has callers keeps the logic; the new one calls it
static supply (value, at = new Date()) {
  // the body that was already here, untouched
}

supply (value) {
  return this.Ctor.supply(value, this.at)
}
```

## An added argument carries a default

**A parameter added to a member that already has callers takes a default, and that default is the
existing call written down.** Not a convenience and not a taste: it is the only thing that keeps
the calls already out there meaning what they meant.

Dropping the default to preserve some finer property of the behaviour trades the contract away for
a detail. Where the two cannot both be had, the contract wins and the finer property waits for the
version that may change it.

## How often a default is evaluated is part of what callers see

**A default is evaluated where it is written, as often as that member is called** — so moving one
from an inner member to an outer one changes the result while the signature stays identical.

Measured on a member that maps over its input and hands each element to a second member, with the
same default written on each: on the inner member it was evaluated once per element, on the outer
one once for the whole call. Two thousand elements came back carrying two distinct values in the
first arrangement and one in the second.

Nothing in either signature says which. **A default that is added, moved or hoisted is a change to
behaviour until a measurement says otherwise**, and reading the two signatures side by side will
not tell you.

## A defect found on the way is left alone

**Establishing that the existing behaviour is wrong does not license changing it.** A retake claims
no gain; a fix is a gain. The moment a change is worth making on its own merits it has stopped
being part of this work and become work of its own — with its own decision, its own commit, and
where it moves the interface, its own version.

The pull is strongest exactly where the defect is plainest. A member that declines to supply the
very thing it is named for, whenever its input already carries something under that name, is
visibly not doing what it says; that reading was confirmed, and the behaviour stayed, because
correcting it is a change to the specification and the specification was not what this work had
been handed.

- **Report it rather than absorbing it.** What the work owes is a statement that the defect exists
  and what fixing it would cost, so that somebody can decide. Quietly leaving it is as wrong as
  quietly fixing it.
