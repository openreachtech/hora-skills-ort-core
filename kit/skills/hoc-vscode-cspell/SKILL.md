---
name: hoc-vscode-cspell
description: "Conventions for the spell checker's vocabulary in `.vscode/cspell.json`, and for settling the words it reports. Use when cspell flags a word, when adding to or pruning `words:` or `ignoreWords:`, or when tempted to reword code to silence it. How names are spelled in the first place belongs to the naming convention; a lint rule that fires belongs to the ESLint conventions."
---

# VS Code cspell

Conventions for `.vscode/cspell.json` — the vocabulary the spell checker reads in the editor —
and for the order in which a reported word is settled.

**The vocabulary is a record of the words the repository uses.** It is not a list of
exceptions to be kept short, and it is not a place to hide mistakes. Every entry is either a
word the repository writes correctly or a string it writes on purpose, and nothing else.

## Settling a reported word

**First ask whether the report is right.** A word that is simply misspelled is a typo, and the
typo is corrected where it was written. Everything below is about a word the checker does not
know, which is a different thing: the repository wrote it correctly, or wrote it on purpose.

**For those, the checker's own vocabulary comes first and changing the code comes last.** What
the tool can settle by itself is settled there, and rewording code to satisfy a spell checker is
the last resort.

| The word is | It goes to | Why there |
| :-- | :-- | :-- |
| A real word the dictionaries lack — a term of a tool, an API, a shell, a word of prose | `words:` | It is spelled correctly, and the checker should learn it |
| A string written on purpose that is not a word at all | `ignoreWords:` | It should be silenced, never taught |
| Neither, and the code can say the same thing in a real word | The code, last | Only once the two above do not fit |

**`words:` and `ignoreWords:` are not interchangeable.** A word in `words:` is one the checker
treats as correct, and it is offered as a suggestion when something near it is mistyped. A
string that is not a word, placed there, is a spelling the checker will then recommend.
`ignoreWords:` only silences it.

### Strings that are not words

**A fragment produced by a pattern is not a word, and it goes to `ignoreWords:`.** The checker
splits text into words by its own rules, and a regular expression or an escape sequence gives
it material those rules were never meant for. The commonest shape is a token glued to the text
beside it: the escape `\u001b[2J` followed by `hora` is read as the single word `Jhora`.

- **The fragment is not a typo, so the code is not where it is fixed.** Splitting the string
  apart to avoid it changes the value the code is about.
- The same holds for a test value that is deliberately not a word, where no real word would do
  the same work.

### Changing the code

**Rewording code to satisfy the checker is allowed only when it keeps what the code is for.**
That is mostly a test value.

- **A value whose job does not depend on its spelling may become a real word.** A test passing a
  near-miss command to see it refused needs a string that is not the command, nothing more.
  `installer` refuses as well as `installl` did, so the test keeps its meaning and the
  vocabulary gains nothing.
- **A value whose spelling is what the test checks stays as it is.** A path carrying a control
  sequence, written to see that an error message survives it, is checking exactly that string.
  Inserting a separator to break up `Jhora` would test a different path, so the fragment goes to
  `ignoreWords:` instead.

## Pruning the vocabulary

**An entry no tracked file uses is kicked out.** The vocabulary records the words the repository
uses, so a word that has left the repository leaves the vocabulary too. Check against the tracked
files (`git ls-files`), not against whatever happens to be in the working tree.

**Do not add `ignorePaths:` for `node_modules/`.** cspell skips `node_modules/` by default, and
dot-prefixed paths as well, so an exclusion written for them excludes nothing and only suggests
that something needed excluding.

## Recording the work

**The order above is also the order of the record.** In an issue's `# Checklist`, and in the
commits that carry the work, the sections stand in this order:

```markdown
## Kicking out words

## Adding words

## Ignored words

## Replacing words
```

- `## Kicking out words` and `## Adding words` are entries to `words:`; `## Ignored words` is
  entries to `ignoreWords:`; `## Replacing words` is the code changed as the last resort.
- **Configuration first, code last.** What the tool settles comes before what had to be
  rewritten, so a reader sees the last resort taken only after the rest was tried.
- Omit a section that has nothing in it.

## Not a lint rule

**ESLint goes the other way, and the two are not to be reasoned from each other.** A lint rule
that fires is a judgement about code, so the code changes and relaxing the rule is the last
resort — see `/hoc-eslint-config`. A spelling vocabulary records words, so the vocabulary changes
and rewording the code is the last resort. Carrying either order across to the other gets it
backwards.
