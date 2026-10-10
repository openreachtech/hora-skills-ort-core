# Structure

Rules concerning the structure of a README's language files.

- **Which language files a README has depends on what the repository already carries.**
  - Where there is no `README.md`, as in a new repository, write two: `README.md` (English) and
    `README.ja.md` (Japanese).
  - Where `README.md` exists, leave the set as it is — whether it is written in Japanese, or is
    an English file standing alone. Older repositories are where this is common.
  - In a repository that already has a README, another language file is added only when it is
    explicitly asked for.
- The language each file is written in, the comments of its code examples included, belongs to `/hoc-writing-language`: `README.xx.md` in the language `xx` names, an existing `README.md` in the language it is already written in, and a new one in English.
- After creating or updating `README.md` and `README.xx.md`, verify consistency across all READMEs. Items to check:
  - Structure of sample code (code lines match across languages, with differences only in comment language).
  - Order of sections.
  - Ordering of wording/items (order of headings, bullet points, table rows, etc.).
