---
name: hoc-charters-coding
description: "ORT coding charter. Write readable, unified code; avoid modification; let the code explain everything."
---

# ORT Coding Charter ⛩️

The coding charter for ORT development. It aims to minimize the reader's burden while maximizing maintainability and AI learning efficiency.

## (1) Neither Quirk Nor Quiz

- Don't write quiz-like code that forces readers to rack their brains, nor unique, unconventional, show-off code.
- Write simple, unified code that anyone can easily understand.

## (2) Writing Once, Reading Unlimited

- Code is written to be read by others. Hiding information out of laziness only increases the reader's effort.
- Our job is to write code, not to omit it. Syntactic sugar is the furthest thing from our values.
- However, widely adopted and established constructs — such as `class` (sugar over prototype inheritance) or `async`/`await` (broadly, sugar over Promises) — are not regarded as syntactic sugar. For constructs whose "sugar-ness" differs between the broad and narrow senses, use only those approved on a whitelist.

## (3) Write Code Like Carving on a Slab

- Once written, code should be modified as little as possible. Overlooking an affected area leads to hard-to-find bugs.
- If a feature can be added without touching existing code, bugs are confined to the "added code." Learn the Open-Closed Principle and design as if carving into a slab.

## (4) Let the Code Explain All

- Don't write code that relies on implicit understanding or undocumented assumptions. Give the reader all the information needed to understand its purpose and functionality.
- `t()` tells us nothing; `i18n.t()` gives at least a basic sense of what it does.

## (5) Write Code You Can Justify

- The PR author is responsible for justifying their code, able to explain every line when asked "why this logic?".
- Unacceptable: "I wrote it as told", "I pasted AI output as-is", "it seemed to work", "I reused existing code without understanding it".

## (6) Count Spaces as Code

- Here, "spaces" means half-width spaces and blank lines. Treat them as spaces that contain code, not gaps between code.
- Spaces clarify structure and reduce the reader's burden. Don't omit blank lines to save keystrokes.

## (7) Vertical is Better than Horizontal

- Prefer vertical code over horizontal. Long lines force horizontal scrolling and disrupt reading flow; the human eye is not accustomed to reading horizontally separated information.
- Examples: don't pack multiple lines into one; break method chains one by one; put each ternary term on its own line; prefer more records over more DB columns; put each HTML attribute on its own line.

## (8) Keep Everything One by One

- Making the reader extract multiple pieces of information from one piece of code is a trap for oversight. Narrow information down to one to reduce the reader's burden.
- Examples: one expression per line; don't nest if statements; one responsibility per method; one concept per class; break `))` between the two characters; RDB normalization; one purpose per PR.

## Version

- v1: 2025/04/03
