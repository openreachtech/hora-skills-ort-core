---
name: hoc-comments
description: "Comment-writing conventions — what a comment says (the why, never a restatement of the code), where a long comment breaks (at a clause, never at a column), a multi-line block of three lines or more, and lines commented out with `//`. Use when writing or reviewing a JSDoc block, a block comment or an inline comment, a code example that carries one, or when commenting lines out. Which language a comment is written in belongs to the writing-language convention."
---

# Shared: Comments

Conventions related to comment writing. Applies across JSDoc, block comments (`/* */`) and inline comments (`//`).

## Language

- Which language a comment is written in belongs to `/hoc-writing-language`. It is read here for
  the default it gives every comment — English, in source, in the tests generated under `tests/`
  and in the code examples of a document or a skill — which holds wherever no instruction, and no
  language code in the file's name, has settled it first.

## What a comment says

**A comment carries what the code cannot say for itself — above all, why.** Where the code makes
a choice a reader would question, the comment stating the reason is the one that has to be
there, and it is the one most often missing.

- **A comment that restates the code is not written.** It says nothing the line beside it does
  not, and it is one more thing to keep in step with that line.
- **A comment moves with the code it describes.** Where a change makes a comment untrue, the
  change corrects it; a comment describing code that is no longer there misleads more than no
  comment would.
- **Code is not commented out and left.** The history keeps what was removed; a commented-out
  block keeps only the question of whether it still matters.

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

## A block comment that runs to several lines takes at least three

**A block comment written over more than one line takes three lines at the least**: the opening
marker on a line of its own, the text, and the closing marker on a line of its own. Every line
between the two markers keeps its leading `*`.

```javascript
// NG: two lines, with the text sharing a line with each marker
/* Retries are spaced exponentially,
   so a struggling server is given room to recover. */

// NG: the leading * dropped from the lines between the markers
/**
   Retries are spaced exponentially,
   so a struggling server is given room to recover.
 */

// OK: three lines or more, every line between the markers marked
/**
 * Retries are spaced exponentially,
 * so a struggling server is given room to recover.
 */
```

- **Two block comments in a row are separated by a blank line.** Placed against each other, the
  closing marker of one runs straight into the opening marker of the next.

```javascript
// NG: two blocks with nothing between them
/**
 * @typedef {*} BooleanLike
 */
/**
 * @typedef {*} NumberLike
 */

// OK: a blank line between the two
/**
 * @typedef {*} BooleanLike
 */

/**
 * @typedef {*} NumberLike
 */
```

## Lines are commented out with `//`, never with a block comment

**Where several lines are commented out, each takes its own `//`.** A block comment is not used
for it: `/* */` does not nest, so a block wrapped around code that already holds a block comment
ends at that comment's `*/`, and the rest of the code runs again.

```javascript
// NG: the block ends early, at the */ of the JSDoc inside it
/*
/**
 * @returns {number}
 */
computeDelay () { ... }
*/

// OK: one // per line
// /**
//  * @returns {number}
//  */
// computeDelay () { ... }
```

- **This is how lines are commented out while the work is in progress.** What reaches a commit
  carries no commented-out code at all, as "What a comment says" above holds.
