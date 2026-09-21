---
name: hoc-error-codes
description: "The string an error carries so a caller can tell it apart from every other: the parts of `Aaa.XBBB.CCC`, the categories a failure is sorted into, the per-resolver identifier below them, and the codes a client raises on its own behalf. Use when adding an error to a resolver, or when choosing a category for a new failure. Declaring and throwing the code belong to each stack's own convention; whether a failure throws at all belongs to the errors convention."
---

# Error Codes

An error code identifies a failure across every boundary it crosses — the resolver that raised it,
the response that carried it, the client that matched it against a message, and the report someone
reads afterwards. This convention settles what that string is.

Whether a failure is reported by returning `null` or by throwing is settled before this convention
is reached; the errors convention settles it. What follows applies to a failure that arrives
carrying a code.

## The code is a string

**An error code is a string, never a number.** It is matched for equality and never ordered or
added to, it carries a letter in the middle, and its leading zeros are part of it — `001` and `1`
are not the same code.

```js
Unauthenticated: '102.X000.001'   // OK
Unauthenticated: 102000001        // NG — the operation letter is gone, and so are the leading zeros
```

## The three parts of `Aaa.XBBB.CCC`

| part | what it says | width |
| :-- | :-- | :-- |
| `Aaa` | what kind of failure this is | 3 digits |
| `XBBB` | who raised it | 1 letter + 3 digits |
| `CCC` | which one of theirs it is | 3 digits |

**The dots are what make the widths cheap.** An unseparated code has to reserve enough digits for
the largest count anyone will ever need, and every count it holds is then bounded by a decision
taken before any of them were known. Separated, each part is read on its own, and a part that
turns out too narrow widens without disturbing its neighbours.

## `A` — standard or application

| value | what it covers |
| :-- | :-- |
| `1` | the framework's own errors, raised by the engine and the packages beneath a project |
| `2` | the application's own errors, raised by what the project itself writes |

**A project declares `2` codes and never declares a `1` code.** The standard ones already exist
below it: an application that redeclares one has two names for a failure the framework raises, and
the client matching on the code cannot tell which of them it received.

## `aa` — the category

The two digits below `A` sort the failure. **They carry the same meaning under `1` and under `2`**,
so a category is learned once and read in both: validation is `03` whether the framework raised it
(`103`) or the application did (`203`).

| `aa` | category | raised when |
| :-- | :-- | :-- |
| `00` | unknown | nothing more specific applies |
| `01` | implementational | the code itself is wrong — an abstract member reached without an override |
| `02` | access right | unauthenticated, unauthorized, denied schema permission, banned |
| `03` | validational | the input did not pass its check |
| `04` | database record | the record needed was absent, or the state forbade the operation |
| `05` | external API | a service outside this system failed or refused |

**Sort by what failed, not by what the caller should do about it.** A credential that does not
match is an access-right failure (`02`) even though the check ran inside a mutation, and a service
outside the system is `05` even when the reason it refused was a permission of its own. The two
readings part company exactly where it matters: the client decides what to show from the category,
and a category assigned by the handling it expected makes that circular.

- **A category that does not fit is not a licence to invent one.** The list above is the whole of
  it, and `00` exists for what none of the others describes.

## `X` — which operation raised it

| letter | operation |
| :-- | :-- |
| `Q` | query resolver |
| `M` | mutation resolver |
| `S` | subscription resolver |
| `X` | none of them — the code is raised across resolvers, or below them |

**`X` is what a built-in takes.** A code the engine raises for every operation alike has no one
resolver to name, so it takes `X` and the resolver number `000` with it — `102.X000.001` is
unauthenticated wherever it happens.

## `BBB` — which resolver raised it

**A resolver holds one identifier, and every code it declares carries that identifier.** The number
is assigned per operation letter, so `Q001` and `M001` are two different resolvers and neither
collides with the other.

- **`000` means no single resolver.** It goes with `X`, and only with it.
- **The identifier is the resolver's for as long as the resolver exists.** It is not reassigned
  when a neighbouring resolver is deleted, and the gap that leaves is left as a gap.
- 999 resolvers per operation is the ceiling. Mutations are where a project accumulates them, and
  that is the count the width was chosen for.

## `CCC` — the running number

**The number counts within one resolver and one category, from `001`.** So a resolver's validation
errors run `001`, `002`, `003` under `203`, and its first database error is `001` again under `204`.

```
203.M024.001   InvalidEmail       validation, first
203.M024.002   InvalidPassword    validation, second
204.M024.001   UserNotFound       database, first — the count starts over
```

Counting per category, rather than once across the resolver, is what keeps a category's codes
contiguous. Adding a validation error to a resolver that already has database errors appends to the
validation run; counted across the resolver it would land after them, and a reader could no longer
tell from the number whether a category was complete.

## A code that has shipped is never renumbered

**Once a code has left the system it is permanent.** A client matches on the string, a log holds
it, and a report cites it — none of which are reachable from here, and all of which break silently
when the string moves.

- **Deleting a code leaves its number spent.** The next error takes the next number, and the gap
  stays. A gap costs a reader one question; a reused number costs a reader a wrong answer.
- **A code discovered to be in the wrong category stays where it is.** Renumbering it to the right
  category is the same break as any other. Where the miscategorization matters enough to act on,
  what moves is the error — a new code in the right category, the old one retired and its number
  left spent.

## Codes the client raises

**A request can fail without a resolver ever running.** The variables can be rejected before the
request leaves, the network can drop it, and the response can arrive unparseable — failures the
client raises on its own behalf, which still need codes, and which must not be mistakable for
anything a resolver sent.

They take the same format, and they are told apart by the category. **Client categories fill `aa`
from the top down**, starting at `9` in the tens digit and descending as they are used up, so they
can never meet the server's categories counting up from `00`.

| code | raised when |
| :-- | :-- |
| `190.X000.001` | unknown — nothing more specific applies |
| `191.X000.001` | the variables did not pass their check before the request left |
| `191.X000.002` | the headers did not pass theirs |
| `192.X000.001` | the request never completed — a network failure |
| `192.X000.002` | the response arrived and did not parse |

- `X000` throughout: no resolver raised these, so there is none to name.
- **`A` is still `1`**, because these are the client framework's own errors. A client raising a
  code of its own — one the framework does not define — takes `2` and the same categories.

## Where this convention stops

Run this convention to its end and what you have is a string. **Everything that happens to that
string afterwards belongs somewhere else**: which hash declares it, how it is thrown, how a
validator reaches it, and how a client turns it into something a person reads. Those are the
backend and frontend conventions for the stack in hand, and each of them assumes the string this
one produces.

- **The operation letters are GraphQL's.** Anything else raising a code in this format — a job, a
  consumer, the engine below them — has no operation to name, and takes `X` with `000` behind it.
- An HTTP error carries a status code and a message instead, and is not written in this format.
