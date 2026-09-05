---
name: hoc-license
description: "Write and update LICENSE files for projects. Use this skill whenever the user asks to create, update or delete a LICENSE file."
---

# License Skill

When creating or updating the LICENSE file and the README's license notation, follow the rules below.

**Behavior is determined by the `license` field in `package.json`** (if the user explicitly specifies a different license, prioritize that, and update `package.json` accordingly if needed).

## Workflow

1. Read `license` in `package.json`.
2. Adjust the `./LICENSE` file according to its value (see below).
3. Align the license sections of `README.md` (default language) and each `README.xx.md` per language.
   The wording and templates for the license section are governed by the "License" convention in the **README convention**.

## `UNLICENSED`

Denotes private/restricted distribution.

- If `./LICENSE` exists, **delete it** (if it does not exist, do nothing).
- The body of the license section in `README.md` / `README.xx.md` should be the single word `UNLICENSED`.

## `Apache-2.0`

Copy the full text of [templates/Apache-2.0](./templates/Apache-2.0) as-is into `./LICENSE` (do not modify the body).

- The **`[yyyy]`** in the trailing APPENDIX should match the year in the README's Copyright section (`© <year> ...`).
- The **`[name of copyright owner]`** in the trailing APPENDIX should be replaced with `author` from `package.json` (e.g., `Open Reach Tech Inc.`).

The license section of `README.md` / `README.xx.md` should contain wording that links to `./LICENSE`
(the template is governed by the "License" convention in the README convention).

## `MIT`

Copy the full text of [templates/MIT](./templates/MIT) into `./LICENSE`, replacing the placeholders.

- **`[full name]`** should be replaced with `author` from `package.json` (e.g., `Open Reach Tech Inc.`).
- **`[year]`** should match the README's Copyright section (`© <year> ...`).

The license section of `README.md` / `README.xx.md` should contain wording that links to `./LICENSE`
(the template is governed by the "License" convention in the README convention).

## Others

When the license is none of `UNLICENSED`, `Apache-2.0`, or `MIT`, prompt the user to confirm what to do with the `./LICENSE` file.
