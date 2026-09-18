---
name: hoc-readme
description: "The README a project carries — the file it keeps per language, the section order and the fixed text of each, the parts split out under `docs/` and linked back, the API reference, and the naming used in code examples. Use whenever a README is created or updated. Which `LICENSE` file the project carries belongs to the license convention, and what a document may state as fact to the documentation convention."
---

# README

When creating or updating a README, follow the rules in the following files.

- [structure.md](./references/structure.md) — Rules for the structure of language files (`README.md` / `README.ja.md`, and the language used in in-code comments).
- [sections.md](./references/sections.md) — Rules for README section structure (Features / Contribution / Developer / Copyright, etc.).
- [usage.md](./references/usage.md) — Rules for placement of split-out files (the `docs/<lang>/<category>/` language-to-category directory structure, keeping the `.xx.md` suffix, not bundling with the package plus linking with absolute GitHub URLs) and rules for the Usage / Features sections.
- [api-references.md](./references/api-references.md) — Rules for writing the API reference (member selection, Notation of Class Members, splitting multiple classes into `docs/<lang>/api/` and aggregating them via `docs/<lang>/api/index...`).
- [sample-code.md](./references/sample-code.md) — Rules for code examples (variable naming). Applied uniformly to the code examples in every chapter.
