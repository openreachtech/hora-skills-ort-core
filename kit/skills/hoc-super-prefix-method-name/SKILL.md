---
name: hoc-super-prefix-method-name
description: "The super-prefixes a method name may carry before its verb, and what each one promises about how the method behaves. Use when naming or reviewing a method that handles many items at once, realizes a recursion, or calls a delegate. The rest of a method name belongs to the naming convention; how a recursion is laid out, to the method recursion convention; how the call to a delegate is isolated, to the delegation convention."
---

# Super-Prefix: Method Name

**A method name may carry a prefix before its verb, marking what kind of behavior the method
is.** The verb says what the method does; the super-prefix says how it goes about it, and it is
what tells one member from its siblings at the call site.

**It is called *super* because it goes in front of the word that already leads the name.** A
method name opens with its verb, so a prefix placed ahead of it stands one level above what the
name begins with: an adjective before the verb, as with `deep~` and `bulk~`, or a verb before
the verb, as with `invoke~`.

| Super-prefix | What it marks |
| :-- | :-- |
| `bulk~` | The method carries out a single-item behavior for many items at once |
| `deep~` | The method realizes a recursion: it calls itself as it descends into a nested structure |
| `invoke~` | The method calls a method of a delegate, and does nothing else |

- **The prefix comes before the verb and leaves the rest of the name alone.** Everything behind
  it follows the naming convention, the transitive-verb-plus-object rule included, so it is
  `deepLoadFiles()` rather than `deepLoad()`.

## `bulk~`

| behavior | `bulk~` |
| :-- | :-- |
| `createEntity()` | `bulkCreateEntities()` |
| `fetchResult()` | `bulkFetchResults()` |

```javascript
// bulk~: the many-at-once member, standing beside the one that takes a single item
bulkSaveCustomers ({ customers }) { /* ... */ }

saveCustomer ({ customer }) { /* ... */ }
```

- **`bulk~` says a single-item member exists beside it.** The prefix is what tells the two
  apart at the call site, so it is worn by the many-at-once member and never by the one that
  takes a single item — a lone `bulk~` with nothing to contrast with is naming a distinction
  the class does not make.

## `deep~`

| behavior | `deep~` |
| :-- | :-- |
| `buildClause()` | `deepBuildClause()` |
| `kickOutNull()` | `deepKickOutNull()` |

- **`deep~` is worn by the member that realizes the recursion.** It makes explicit that the
  method calls itself. Where in its body the call to itself sits does not decide it.
- **The plain-named entry point in front of it wears no prefix.** How a recursion is laid out
  behind its entry point belongs to `/hoc-methods-recursion`.

## `invoke~`

| behavior | `invoke~` |
| :-- | :-- |
| `launchRequest()` | `invokeLaunchRequest()` |
| `validate()` | `invokeValidate()` |

- **What it calls is a method of a delegate**, and calling it is the member's whole body. What a
  delegate is, why its call is isolated in a member of its own, where the `try`/`catch` goes and
  why the member stays private belong to `/hoc-classes-delegation`.
- **What it is read for here is the name.** `invoke~` is worn by the member the delegation
  convention isolates, and by nothing else: a member that also builds what it passes on or
  handles what comes back is not an `invoke~`, whatever it calls.
