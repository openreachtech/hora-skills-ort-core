# REST API Error Codes

What a REST API fills `XBBB.CCC` with, the bands its client raises, and the status errors a REST
API alone carries. The shape of the code, the `A` digit, the categories and everything that holds
across every stack are in [SKILL.md](../SKILL.md).

## `XBBB` — which request method raised it

**A REST API spends the whole group on one identifier: the request method.** There is no second
thing below it for the digits to count, so they stay `000` and the letter does the naming.

| identifier | method |
| :-- | :-- |
| `G000` | `GET` |
| `P000` | `POST` |
| `D000` | `DELETE` |
| `U000` | `PUT` |
| `A000` | `PATCH` |
| `H000` | `HEAD` |
| `O000` | `OPTIONS` |
| `T000` | `TRACE` |
| `C000` | `CONNECT` |
| `X000` | none of them — the code is raised across methods, or below them |

- **The letters are a table, not a derivation.** `POST`, `PUT` and `PATCH` all begin with `P`, so
  `POST` keeps it and the other two take `U` and `A`. Read the letter off the table; do not work
  it out from the method name.
- **`X000` means here what it means everywhere**: no one method raised this. The other rows name a
  method, and their zeros are unspent rather than meaningful — what a reader reads is the letter.

## `CCC` — counted per method and per category

**The number counts within one method and one category, from `001`.** The method is the whole of
the identifier, so the count runs across every endpoint reached by that method: the validational
errors a `POST` can return are `203.P000.001`, `203.P000.002`, and on.

## The bands the client raises

| code | raised when |
| :-- | :-- |
| `190.X000.001` | unknown — nothing more specific applies |
| `191.X000.001` | the arguments did not pass their check before the request left |
| `192.X000.001` | the request never completed — a network failure |
| `192.X000.002` | the response arrived and did not parse |

- The bands divide by when the client noticed, and they are the same three a GraphQL client
  fills: `190` unknown, `191` before the request left, `192` after the response came back.
- `X000` throughout: the client raised these on its own behalf, so there is no method to name.

## `193` — the status errors

**A REST API answers with an HTTP status, and its client has to act on one.** That is a failure no
GraphQL client meets, so it takes a band of its own below the three above.

**In `193` the number is spent on the status itself.** `CCC` reproduces the three digits of the
HTTP status rather than counting from `001` — the one place this convention takes the licence
[SKILL.md](../SKILL.md) gives for a number the stack has already published.

| code | status |
| :-- | :-- |
| `193.X000.400` | `400` Bad Request |
| `193.X000.401` | `401` Unauthorized |
| `193.X000.403` | `403` Forbidden |
| `193.X000.404` | `404` Not Found |

- **A status the client has to act on takes its row, and takes it without being assigned one.**
  The number is the status, so adding a row decides nothing — which is the whole reason the band
  is spent this way.
- `X000`: the status arrived on the response rather than from a method the client raised the code
  through.
