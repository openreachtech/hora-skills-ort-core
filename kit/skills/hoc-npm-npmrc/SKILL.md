---
name: hoc-npm-npmrc
description: "How the npm configuration file `.npmrc` is written: each setting as `key = value`, with spaces on both sides of the `=`, edited directly and never through `npm config set`. Use when adding, changing or reviewing a line of `.npmrc`, or when about to run `npm config set`. What each setting does belongs to the convention that turns it on — the install-scripts gate, the release-age quarantine."
---

# npm: `.npmrc`

How the npm configuration file is written. **This skill holds the form of the file; what each
setting in it does belongs to the convention that turns it on.**

## Each setting is written `key = value`

Spaces on both sides of the `=`.

```
strict-allow-scripts = true
min-release-age = 7
```

## Edit the file directly, never with `npm config set`

- **`npm config set` writes the pair without the spaces, and it rewrites the lines already in
  the file** — so applying one setting through it reformats everything else, and the diff
  carries lines nobody asked to change.
- Edit the file directly. Where the command has already run, the file has to be reformatted by
  hand afterwards.

## What each setting does belongs elsewhere

| Setting | Belongs to |
| :-- | :-- |
| `strict-allow-scripts` | `/hoc-npm-install-scripts` |
| `min-release-age` | `/hoc-npm-vulnerability` |

- **A line is added to `.npmrc` by the convention that needs it**, and written here in the form
  above. The setting's meaning, its value and the order it is put in relative to an install are
  that convention's to decide.
