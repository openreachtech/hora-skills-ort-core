---
name: hoc-async
description: "Conventions for writing asynchronous code. When writing Promises, use async/await whenever possible."
---

# Shared: Async

Conventions related to writing asynchronous code.

## Promise

- When writing Promises, use `async`/`await` whenever possible.

## A value that crosses the boundary a promise makes

**Making one member async is not a local change.** Every caller becomes async with it, and the
value it hands back now travels through a promise instead of being returned directly — so
properties of that value which nothing used to inspect begin to matter.

**What inspects it is the promise itself, and what it looks for is `then`.** A value answering
that lookup with a function is taken for a thenable: the promise calls it as
`then(resolve, reject)` and settles to whatever that call passes on, so the value is replaced by
something it was never meant to produce.

**Nothing reports it.** The call lands inside the value's own code carrying arguments it has no
reason to expect, and the failure surfaces wherever that code first reads one of them — a long
way from the `await` that caused it, and wearing the shape of an ordinary bug.

- **A value that answers every property lookup must refuse `then`.** Where a lookup is answered
  by producing something — a handler built per key, a member resolved on demand — `then` is
  produced too, and the value becomes a thenable by accident. Answer it as absent, and answer the
  symbol keys the same way.
- **The hazard is the probe, not whatever answered it.** Any value carrying a `then` member meets
  it, so a class that happens to define one is caught in the same way. The rule is about what
  crosses the boundary, never about how the value was built.
- **It appears only once the boundary is there.** The same value returned directly is never
  probed, which is why making a member async is where this is met rather than where the value was
  written.
