# Criteria for using abbreviations

This lists the criteria and whitelist for which abbreviations may be used in naming. Referenced from the naming convention itself (`SKILL.md`). This abbreviation whitelist is shared between JavaScript and CSS and is not managed in two places (abbreviations in CSS custom property names also follow this criteria/whitelist).

- For variable names, class names, method names, etc., any abbreviation is prohibited except those "generally recognized widely, not just in programming."
- Any abbreviation not listed in the whitelist below is prohibited. The whitelist may be expanded as needed.

## Whitelist

| abbreviation | original word | note |
| :-- | :-- | :-- |
| admin | administrator | general knowledge |
| app | application | general knowledge |
| config | configuration | general knowledge |
| enum | enumerate | - |
| env | environment | general knowledge |
| id | identity | general knowledge |
| init | initialize | - |
| int | integer | general knowledge |
| max | maximum | general knowledge |
| min | minimum | general knowledge |
| nav | navigation | general knowledge |
| sin | sine | - |
| cos | cosine | - |
| tan | tangent | - |
| Ctor | constructor | `constructor` is JavaScript reserved word |
| noop | no operation | for description |
| faq | acronym of "frequently asked questions" | - |
| func | function | Reluctantly allowed since `function` is a reserved word. Prefer a name that explicitly states the functionality wherever possible. `fn` / `f` are not allowed |

Supplementary notes (abbreviations):

- Only two kinds are permitted: "abbreviations universally used in society" or "abbreviations conventionally used in programming for many years" (e.g. `char`, `exec`, `eval`, etc. are also conventionally allowed).
- The following abbreviations are prohibited: `acc`, `arr`, `avg`, `auth`, `btn`, `cate`, `cfg`, `cnt`, `cond`, `ctx`, `e`, `err`, `ev`, `ex`, `fmt`, `msg`, `no`, `num`, `prod`, `tx`, `tz`, and so on.
- Even if an abbreviation is used within an external module, that is not a reason to use the abbreviation yourself (e.g. do not imitate Sequelize's `msg` or Express's `req` / `res`).
