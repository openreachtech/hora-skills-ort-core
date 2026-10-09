---
name: hoc-comments
description: "Comment-writing conventions. Use when writing or reviewing a JSDoc block or an inline comment, or a code example that carries one. Which language a comment is written in — English by default, in source, in generated tests and in code examples — belongs to the writing-language convention."
---

# Shared: Comments

Conventions related to comment writing. Applies across JSDoc, block comments (`/* */`) and inline comments (`//`).

## Language

- Which language a comment is written in belongs to `/hoc-writing-language`. It is read here for
  the default it gives every comment — English, in source, in the tests generated under `tests/`
  and in the code examples of a document or a skill — which holds wherever no instruction, and no
  language code in the file's name, has settled it first.

## A long comment breaks where its sentence does

**Where a comment runs to more than one line, break it at a point the sentence itself offers** —
after a comma, at a conjunction, between two sentences. A column count such as 80 is not the
criterion: filling each line to a margin breaks wherever the word count happens to land.

```javascript
// NG: filled to the margin, so the line ends mid-clause
/**
 * A fragment already on the path contributes nothing, which is what stops a
 * cyclic document from being walked forever.
 */

// OK: the break falls after the comma, where the clause does
/**
 * A fragment already on the path contributes nothing,
 * which is what stops a cyclic document from being walked forever.
 */
```

- **The unit a reader takes in is the line.** Broken at a clause, each line is one statement and
  the comment can be read down the left edge; broken at a column, a line ends on `stops a` and
  carries no meaning of its own.
- **This holds for every comment that runs long** — a JSDoc block, a `/* */` block, and a run of
  `//` lines alike.
- This governs prose. A type literal inside a JSDoc tag is already one property per line.
