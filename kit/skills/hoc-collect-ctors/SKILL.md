---
name: hoc-collect-ctors
description: "How a set of classes, one per member, is collected into the one place callers ask: a barrel file, or a folder loaded once at startup, chosen by how an addition reaches its users. Use when variants are modelled as classes, when about to list them in an array or a keyed object, or when adding a barrel or a directory loader. Why a member is a class belongs to the class design principles convention."
---

# Collect Ctors

A set whose members differ — the days of a week, the fingers of a hand, the currency pairs an
application trades, the expressions a query builder accepts — is held as one class per member,
and the classes are collected into one place that callers ask. How they are collected is
decided by one question: **how does an addition to the set reach the people who use it?**

## One class per member

- **Each member of the set is a class of its own**, never an entry in an array of values or a
  key in an object literal. What sets one member apart from the others lives in its class, and a
  member that has to behave differently overrides only what differs.
- Why a member is a class rather than a shape of data belongs to `/hoc-classes-principles`.

## Choose by how an addition reaches its users

| The set | How an addition reaches its users | Collected through |
| :-- | :-- | :-- |
| Closed: the number of members is fixed by the domain | It never does; no member is added | A barrel file |
| Grows inside a published package | Through a release, once a user raises the version | A barrel file, updated as part of the publish |
| Grows inside an application | A file placed in the folder takes effect at the next start | A folder, loaded once at startup |

**Whether the set will grow is not the question.** A package's expressions grow too, and still
take a barrel: nothing added to a package reaches anyone until it is published, so the barrel is
updated as part of the release work that has to happen anyway. What asks for a folder is an
addition that takes effect without passing through any release.

### A barrel file

**A barrel imports every class of the set and exports them together.** Every member stands in
it as one import line, it is resolved statically, and nothing is loaded at run time — so a reader
sees the whole set without running anything.

```javascript
// fingers/index.js — a closed set: five members, and no sixth will come
import ThumbFinger from './ThumbFinger.js'
import IndexFinger from './IndexFinger.js'
// ... the remaining three

export default [
  ThumbFinger,
  IndexFinger,
  // ...
]
```

- **In a package, the barrel is the published surface.** It states in import lines what the
  package offers, and adding a member to it is one step of the release that publishes the member.
- **A barrel is not used for a set that grows inside an application.** Every addition would edit
  the barrel, an existing file, which is the cost the folder exists to remove.

### A folder loaded once at startup

**Where an addition takes effect without a release, it is made by adding a class file to a fixed
folder, and by nothing else.** No list, no barrel and no existing file is edited to admit it.

- **The classes are collected once, when the application starts.** A collection class with an
  asynchronous factory method loads every class under the folder through `DeepCtorsLoader`, and
  holds them as its one property. The constructor only receives what the factory loaded, as
  `/hoc-classes-constructor` requires.
- **`DeepCtorsLoader` is taken as a dependency, never written in the project.** It comes from
  the package `@openreachtech/mentsu-deep-loader`, installed from the public npm registry
  (`registry.npmjs.org`); no `.npmrc` line points its scope at another registry. It walks a folder
  recursively and keeps only the classes that are, or extend, the base class it is bound to. How it is called is that
  package's README; a loader of the same job written by hand is a second copy of it, and one
  carrying its name is mistaken for it.
- **The collection is then used for the life of the process.** It is built once and never
  changes, so nothing has to guard the set against callers: the set is the folder.
- **A loader is not used for a closed set.** It tells the reader the set can grow when it cannot,
  and it turns a static import into a load at run time for nothing.

## Asking the collection

- **Callers ask the collection, never the folder or the barrel.** It answers whether a member
  exists, which class it is, and what the members are.
- **A member is found by asking the classes**, with `find()` over the collected classes and a
  static member each class answers through. The classes are held in an Array, and no dictionary
  keyed by a string is built beside it: classes defined in the source code are never so many that
  `find()` over them is slow — see `/hoc-prohibit-native-features`.
- **A list handed to a caller is a new array each time**, so what a caller does with it never
  reaches the collection.
- **The collection reaches the loader through a static getter**, so a subclass or a test can
  replace it — see `/hoc-wire-dependencies`.

## Where this stops

- Registering a class in a barrel is a commit of its own, and how that commit is split from the
  class belongs to `/hoc-git-commit`.
