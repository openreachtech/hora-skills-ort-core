# GraphQL Error Codes

What a GraphQL server fills `XBBB.CCC` with, and the bands its client raises. The shape of the
code, the `A` digit, the categories and everything that holds across every stack are in
[SKILL.md](../SKILL.md).

## `X` — which operation raised it

GraphQL spends the letter and the digits separately: **the letter names the kind of operation, and
the digits name which resolver of that kind it was.**

| letter | operation |
| :-- | :-- |
| `Q` | query resolver |
| `M` | mutation resolver |
| `S` | subscription resolver |
| `X` | none of them — the code is raised across resolvers, or below them |

**`X` is what a built-in takes.** A code the engine raises for every operation alike has no one
resolver to name, so it takes `X000` — `102.X000.001` is unauthenticated wherever it happens.

## `BBB` — which resolver raised it

**A resolver holds one identifier, and every code it declares carries that identifier.** The number
is assigned per operation letter, so `Q001` and `M001` are two different resolvers and neither
collides with the other.

- **`000` goes with `X`, and with nothing else.** A resolver's own identifier starts at `001`.
- 999 resolvers per operation is the ceiling. Mutations are where a project accumulates them, and
  that is the count the width was chosen for.

## `CCC` — counted per resolver and per category

```
203.M024.001   InvalidEmail       validational, first
203.M024.002   InvalidPassword    validational, second
204.M024.001   UserNotFound       database record, first — the count starts over
```

One resolver's codes spread across the categories its failures fall into rather than gathering
under one. For a resolver registered as `M001`:

```
203.M001.001   MaximumSizeExceeded    the upload was over the cap — validational
205.M001.001   FailToSendToStorage    the storage service refused — external API
```

## The bands the client raises

| code | raised when |
| :-- | :-- |
| `190.X000.001` | unknown — nothing more specific applies |
| `191.X000.001` | the variables did not pass their check before the request left |
| `191.X000.002` | the headers did not pass theirs |
| `192.X000.001` | the request never completed — a network failure |
| `192.X000.002` | the response arrived and did not parse |

- The bands divide by when the client noticed: `190` is unknown, `191` is before the request left,
  and `192` is after the response came back.
- `X000` throughout: no resolver raised these, so there is none to name.
