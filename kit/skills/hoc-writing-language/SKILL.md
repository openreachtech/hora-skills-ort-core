---
name: hoc-writing-language
description: "Which natural language a piece of writing is in — code, comments, a code example, a skill, a README, a document written for a reader — and the order that resolves it: an explicit instruction, then the language the artifact names for itself, then the language an existing document is already in, then the default for its kind. Use when about to write any of them, or when the language of one is in question. Which language files a README keeps belongs to the README convention."
---

# Writing Language

Which natural language a piece of writing is in. **Every artifact resolves it the same way, in the
same order**, and the conventions for each kind of artifact refer here rather than settling it
themselves.

## The order that resolves it

1. **An explicit instruction.** Where the requester names a language for the artifact, that is the
   language, whatever the conversation is in and whatever the kind's default would be.
2. **The language the artifact names for itself.** A file or a directory that carries a language
   code is written in that language: `README.ja.md` in Japanese, `docs/ja/` in Japanese. That
   includes the comments in its code examples — the examples of `README.ja.md` carry Japanese
   comments, and the code lines match those of `README.md` with only the comments differing.
3. **The language an existing document is already written in.** A document is edited in its own
   language: an older repository's `README.md` written in Japanese stays in Japanese. This step
   is for prose; a comment in source takes the default for its kind, whatever the comments
   around it are written in.
4. **The default for its kind**, below.

**The first step that answers settles it.** A later step is never consulted to overturn an
earlier one.

## The default for each kind

| Kind | Default | Why |
| :-- | :-- | :-- |
| Code and identifiers | English, ASCII only (see `/hoc-naming`) | Code is read by whoever maintains it, in any language |
| Comments, in source, in generated tests and in code examples | English | A code example illustrates the very code the rule governs |
| `SKILL.md` and its `references/` | English | Every skill in a package reads the same way |
| A new file without a language code, such as `README.md` or `docs/en/` | English | It is the default language file the others are translated from |
| A document written for a reader — a requirement definition, a progress document, a review or audit report, an acceptance report, a deployment runbook | The language the reader is using | A document nobody can read has not been delivered |
| `LICENSE` | The original text, unchanged | The text is the licence; a translation is not |

- **The reader's language is the language of the request.** Someone who asks for a requirement
  definition in Japanese gets it in Japanese; asked in English, it is in English.
- **The reason the reader's language decides it.** Writing a requirement definition in English for
  a team that works in Japanese means the one person who has to approve it reads it slowest — and
  approval is the step the document exists for.

## Where this stops

- **Which language files a README keeps**, and when another one is added, belong to `/hoc-readme`.
  Once a file exists, the language it is written in is resolved here.
- **How a comment is written**, beyond its language, belongs to `/hoc-comments`.
