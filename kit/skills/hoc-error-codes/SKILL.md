---
name: hoc-error-codes
description: "The string an error carries so a caller can tell it apart from every other: the parts of `Aaa.XBBB.CCC`, the categories a failure is sorted into, and the bands a client fills from the top. Which letters `X` takes, and what `BBB` counts, are settled per server kind in the detail files. Use when adding an error, or when choosing a category for a new failure. Declaring and throwing the code belong to each stack's own convention; whether a failure throws at all belongs to the errors convention."
---

# Error Codes

An error code identifies a failure across every boundary it crosses — whatever raised it,
the response that carried it, the client that matched it against a message, and the report someone
reads afterwards. This convention settles what that string is.

Whether a failure is reported by returning `null` or by throwing is settled before this convention
is reached; the errors convention settles it. What follows applies to a failure that arrives
carrying a code.

**`Aaa` reads the same wherever the code was raised; `XBBB.CCC` does not.** What kind of failure it
is holds across every server a project runs, while who raised it is spelled in the terms of the
server that did — an operation for GraphQL, a request method for a REST API. So this file settles
`Aaa` and the shape of the rest, and a detail file settles the rest itself.

## Detail files

- [graphql-error-codes.md](./references/graphql-error-codes.md) — what a GraphQL server fills
  `XBBB.CCC` with, and the bands its client raises
- [restapi-error-codes.md](./references/restapi-error-codes.md) — the same for a REST API, plus the
  status errors it alone carries

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
  what a server raises, and `00` exists for what none of the others describes. The bands below are
  the client's, and a server never reaches for one.

## The client fills the categories from the top

**A request can fail without the server ever running it.** What is sent can be rejected before the
request leaves, the network can drop it, and the response can arrive unparseable — failures the
client raises on its own behalf, which still need codes, and which must not be mistakable for
anything the server sent.

They take the same format, and the category is what tells them apart. **The client fills `aa` from
the top: the tens digit starts at `9` and drops to `8`, then `7`, as each band is used up**, while
the units inside a band count up from `0`. Counting down is what keeps the client's categories from
ever meeting the server's, which count up from `00`.

- **`A` is still `1`**, because these are the client framework's own errors. A client raising a code
  of its own — one the framework does not define — takes `2` and the same bands.
- Which bands exist is per server kind, and the detail files carry them. `190` is unknown in both,
  and the rest diverge with what each client can see.

## `XBBB` — who raised it

**The four characters together name the raiser, and how they divide is per server kind.** A stack
may spend the letter on a kind of raiser and the digits on which one of that kind it was, or spend
the whole group on a single identifier and leave the digits at `000`. Which letters exist, and what
the digits are assigned to where they are assigned at all, is settled in the detail file.

- **`X000` is reserved, and means the same everywhere: no single raiser.** A code the engine
  raises alike for every operation, every method and every endpoint has nothing to name, so it
  takes the letter `X` and the number `000` together — `102.X000.001` is unauthenticated wherever
  it happens. `X` appears with no other number.
- **`000` behind any other letter is not that.** There the letter is doing the naming and the
  digits are simply unspent, which a reader tells apart by the letter rather than by the zeros.
- **An identifier belongs to what it names for as long as that thing exists.** It is not
  reassigned when a neighbour is deleted, and the gap that leaves is left as a gap.
- 999 per letter is the ceiling wherever the digits are counted at all.

## `CCC` — the running number

**The number counts within one raiser and one category, from `001`.** So one raiser's validation
errors run `001`, `002`, `003` under `203`, and its first database error is `001` again under `204`.

Counting per category, rather than once across the raiser, is what keeps a category's codes
contiguous. Adding a validation error to a raiser that already has database errors appends to the
validation run; counted across the raiser it would land after them, and a reader could no longer
tell from the number whether a category was complete.

- **A detail file may spend `CCC` on something the stack already numbers**, in a band of its own.
  Where it does, it says so and gives the reason — a number the stack has already published is a
  name a reader knows, and a running number beside it would be a second name for one thing.

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

## Where this convention stops

Run this convention to its end and what you have is a string. **Everything that happens to that
string afterwards belongs somewhere else**: which hash declares it, how it is thrown, how a
validator reaches it, and how a client turns it into something a person reads. Those are the
backend and frontend conventions for the stack in hand, and each of them assumes the string this
one produces.
