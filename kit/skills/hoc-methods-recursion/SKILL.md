---
name: hoc-methods-recursion
description: "How a recursive method is laid out: the entry point callers reach, the member that recurses behind it, and where the state a recursion carries from call to call is started. Use when writing or reviewing a method that calls itself, or one that walks a tree or another nested structure. The prefix the recursing member wears belongs to the method super-prefix convention; method signatures in general, to the method-definition convention."
---

# Methods: Recursion

How a method that calls itself is laid out, and how the state it carries from one call to the
next is kept out of reach.

## A recursion sits behind an entry point of its own

**Callers reach a recursion through a plain-named member that does nothing but start it.** The
entry point wears no prefix, and the `deep~` member behind it stays off the interface —
private wherever it can be. Where the recursion carries an accumulator — a visited list, a
depth, a path — the entry point gives it its initial value, and the `deep~` member takes it as
a required parameter.

```javascript
// The entry point: a plain name on the interface
loadFiles ({
  directoryPath,
}) {
  return this.deepLoadFiles({
    directoryPath,
    visitedPaths: [],
  })
}

// deep~: the member that realizes the recursion
deepLoadFiles ({
  directoryPath,
  visitedPaths,
}) {
  // ...

  return this.deepLoadFiles({
    directoryPath: nextDirectoryPath,
    visitedPaths: [
      ...visitedPaths,
      directoryPath,
    ],
  })
}
```

- **Private is written as the absence of `@public`.** The entry point carries `@public` in its
  JSDoc, as every entry point reached from outside does (see `/hoc-jsdoc`); the `deep~` member
  does not, and that is what leaves it out of the published contract. A native `#` method is
  never the means (see `/hoc-prohibit-native-features`).
- **The entry point is what hides the initial value.** It is written inside the entry point,
  never as a default on any signature a caller can reach, so the recursion's internal state is
  stated by the member that starts it and by nothing else.
- **The reason is what the tests have to cover.** A walk over a tree often carries an empty
  array to collect what every node returns. With that array reachable from outside — the
  `deep~` member exposed, or the entry point taking it with a default — a caller can hand in an
  array that already holds values, and every such case is one more the tests must cover.
  Behind the entry point, the only array the recursion ever starts from is the one the entry
  point gives it.
- **A default on the `deep~` member says it can be entered directly**, which is the one thing
  it cannot do: called from outside with the accumulator empty, it starts a walk with no
  record of where it has been. Requiring the parameter is what states that it is a step,
  not a door.

## The `deep~` prefix

- The member that realizes the recursion wears the `deep~` super-prefix, and the entry point
  in front of it wears none. Which member takes the prefix belongs to
  `/hoc-super-prefix-method-name`.
- **It is read here as the check on where the initial value sits.** The prefix tells the step
  from the entry at a glance, so a default later added to the `deep~` member contradicts its own
  name, which is what makes the mistake visible.
