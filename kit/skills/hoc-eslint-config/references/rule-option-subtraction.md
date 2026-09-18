# Narrowing a rule to a set of files

How to leave one rule's options in force everywhere except where they must not be.
Referenced from `SKILL.md`.

## Flat config replaces options; it does not merge them

**A later configuration object's entry for a rule replaces the earlier one outright.** The
options are not combined, so there is no way to say "the same list, minus one entry" by
declaring only the difference. The whole list is restated.

That is what makes the shared config's exported option material necessary rather than
convenient. Without it, restating the list means copying it, and a copy goes stale the next
time the shared config moves — silently, because a stale copy still lints.

## The shared config exports the material it builds from

Beside its default array, the shared config exposes the hash it builds its list-shaped core
rules from. Import it, and restate the rule with one entry filtered out.

```javascript
import {
  default as sharedConfig,
  coreRuleOptionHash,
} from '<the shared config package>'

export default [
  ...sharedConfig,

  {
    files: [
      'tests/**/*.js',
    ],
    rules: {
      'id-denylist': [
        'error',
        // There are 0 or more rest parameters in the array
        // string
        ...coreRuleOptionHash['id-denylist'].spreadOptions
          .filter(it => it !== 'val'), // Kick out `val`
      ],
    },
  },
]
```

`files` is what limits the reach. Everything the pattern does not match keeps the full list,
and a block written without `files` narrows the rule for the whole tree — which is the one
mistake this technique makes easy.

## One entry per filter call

**Chain a separate call for each entry subtracted, rather than one predicate rejecting
several.** Each call is then a line that can be taken back on its own, and taking one back is
the ordinary end of the story: the code gets fixed, and that exemption stops being needed.

```javascript
        ...coreRuleOptionHash['id-denylist'].spreadOptions
          .filter(it => it !== 'val')      // Kick out `val`
          .filter(it => it !== 'callback') // Kick out `callback`
```

A single predicate rejecting both reads shorter and costs a rewrite the moment one of them
is no longer wanted.

## The shape differs by key

**Read the exported types before writing the predicate.** The hash does not hold one shape.

| Shape | Predicate compares |
| :-- | :-- |
| A list of strings | The element itself |
| A list of objects carrying a selector and a message | The selector |
| A list of objects carrying an object, a property and a message | The property |
| A single options object, not a list | Nothing — restate the object |

A predicate written for the wrong shape does not fail loudly. It matches nothing, the list
comes back whole, and the rule keeps firing while the config claims otherwise.

## Match on the selector, never on the message

**Where an entry carries both, the selector is what identifies it.** The message is display
text: it can be reworded in a release without changing what the entry matches, and a
predicate keyed to it then silently stops subtracting anything.

The selector is long and unreadable, and that is not a reason to prefer the message. Take it
verbatim from the exported material rather than retyping it, and put the intent in a comment
beside the call.

## A rule the hash does not carry

**Not every rule has list-shaped options.** A plain on-or-off rule is simply turned off
inside a `files` block:

```javascript
  {
    files: [
      '<the one module that needs it>',
    ],
    rules: {
      'no-undefined': 'off',
    },
  },
```

Nothing is imported and nothing is filtered, because there is no list to subtract from.
Check whether the rule appears in the exported hash before reaching for the subtraction
machinery.
