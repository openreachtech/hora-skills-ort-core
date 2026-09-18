---
name: hoc-license
description: "Write and update LICENSE files for projects. Use this skill whenever the user asks to create, update or delete a LICENSE file."
---

# License

When creating, updating or deleting the LICENSE file, follow the rules below.

**Behavior is determined by the `license` field in `package.json`** (if the user explicitly specifies a different license, prioritize that, and update `package.json` accordingly if needed).

## Workflow

1. Read `license` in `package.json`.
2. Adjust the `./LICENSE` file according to its value (see below).
3. Align the license sections of `README.md` (default language) and each `README.xx.md` per language.
   What that section says, for every value of `license`, is governed by the "License" convention in
   the **README convention**.

## `UNLICENSED`

Denotes private/restricted distribution.

- If `./LICENSE` exists, **delete it** (if it does not exist, do nothing).

## `Apache-2.0`

Copy the full text of [Apache-2.0](./references/templates/Apache-2.0) into `./LICENSE`, replacing the placeholders in the trailing APPENDIX.

- **`[yyyy]`** should match the year in the README's Copyright section (`© <year> ...`).
- **`[name of copyright owner]`** should be replaced with `author` from `package.json` (e.g., `Open Reach Tech Inc.`).

## `MIT`

Copy the full text of [MIT](./references/templates/MIT) into `./LICENSE`, replacing the placeholders.

- **`[full name]`** should be replaced with `author` from `package.json` (e.g., `Open Reach Tech Inc.`).
- **`[year]`** should match the README's Copyright section (`© <year> ...`).

## Others

When the license is none of `UNLICENSED`, `Apache-2.0`, or `MIT`, prompt the user to confirm what to do with the `./LICENSE` file.
