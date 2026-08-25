# Test Cases (data convention for cases)

Convention for how to build the data for `cases` in Jest. Referenced from
`SKILL.md`. See [naming.md](./naming.md) for variable/property naming, and
[structure.md](./structure.md) for loop structure (single/double).

Each behavior defines `const cases = [ ... ]` and runs it with `test.each(cases)`.

- Every element of `cases` must be **an object** (never place a primitive directly).
- See [naming.md](./naming.md) for the top-level property names of element
  objects (`override` / `input` / `tally` / `expected`, and the `~Cases`
  exception for the outer loop of a double loop), and for the variable naming
  of `cases` / `~Cases`.
- In principle, `cases` (and the inner `~Cases`) should contain **2 or more**
  elements. A single element is acceptable only when there is a specific
  reason, such as when only one case can be truthy.
- Unnecessary properties may be omitted (not all 4 are mandatory).
- Unless there is a specific reason not to, primitive values that appear in
  `cases` should be **unique across elements** (don't reuse the same value
  across multiple cases).

  **Why**: This is to confirm that "the value given as `input` is correctly
  reflected in `expected`." If values are duplicated, the test can pass by
  coincidence **even if the implementation returns a different input's value**
  (a false positive). Making values unique guarantees that the input-to-output
  correspondence is actually being exercised.
- For values like `id`, make sure the **element index** within the array can
  be read off from Jest's console log. Specifically, use values like
  `100001`, `100002`, `100003`, … where **the last digit (the index) stands
  out as a large value** (`index` is counted per array). This lets you
  immediately identify which case a value in the log corresponds to.
  - Numeric `id`s should be **at least 6 digits** (`100001` or higher). Fixing
    the digit count aligns the position of the last-digit index across all
    cases, keeping visibility consistent. Do not use small values like `id: 1`.
- For strings, similarly attach a **base string + index suffix**, like
  `'alpha-0001'`, `'alpha-0002'`, …, so the index can be read off.
  - Keep the base string **consistent across elements** (`'source-0001'`,
    `'source-0002'`, `'source-0003'`). Do not change the base per element
    (`'alpha:0001'` / `'beta:0002'`), and do not use a separator other than `-`.
  - Even if you're tempted to use a domain-plausible-looking value (e.g. an
    input that resembles Basic auth like `'username:password'`), do not break
    this format. Prioritize keeping the index readable.
- When `expected` is an **opaque value derived from the input** (Base64, hash,
  signature, etc. — a value that cannot embed a readable index), have the
  **`input` side** carry the index instead. `expected` is written as the raw
  derived result (it need not carry an index). Since `input`'s value appears
  in the log, the case can be identified from there.

```js
// expected is Base64 (opaque), so input.source carries the index
const cases = [
  {
    input: { source: 'source-0001' },
    expected: 'c291cmNlLTAwMDE=',
  },
  {
    input: { source: 'source-0002' },
    expected: 'c291cmNlLTAwMDI=',
  },
]
```

### Write derived values and magic constants as literals, with the derivation expression added as a comment

When writing boundary values or derived constants — **literals whose meaning
cannot be read off from the value itself** (a large integer beyond
`Number.MAX_SAFE_INTEGER`, a boundary value at a specific digit count, etc.)
— into `cases`, write the value **directly as a literal**, and attach its
**derivation expression** as a `// <expression>` comment. This keeps the case
data static and diff-friendly, while letting the reader (or a future editor)
understand "why this value — i.e., what it was derived from" without having
to recompute it.

- Write the value in **the actual type of that field** (a string literal for
  a string field). Do **not compute inline** with an expression (keep the
  case data as a literal, and show the derivation via an expression comment).
- Write the derivation expression in the comment as **readable plain
  arithmetic** (e.g. `Number.MAX_SAFE_INTEGER + 2`). Since the literal itself
  already fixes the exact value and type, there's no need to wrap the comment
  in `BigInt(...)` or append `2n` to make it "match exactly if executed" (the
  comment is an annotation showing the meaning of the value, not executable
  code). In fact, it's precisely because `Number` arithmetic would round and
  fail to match exactly (as with the string representation of a `bigint`)
  that you should avoid inline computation and instead use a literal plus a
  derivation-expression comment.

```js
const cases = [
  {
    input: {
      value: '9007199254740993', // Number.MAX_SAFE_INTEGER + 2
    },
    expected: 9007199254740993n, // Number.MAX_SAFE_INTEGER + 2
  },
]
```

### For opaque-value titles too, first use a readable identifying property (`$#` is a last resort)

Even when the test target handles **opaque objects** (types like `WeakMap` /
`Map` / `Set` / instances, whose console output collapses to something like
`WeakMap {}`), do not **casually reach for `$#`** (Jest's row index, i.e. a
serial number). The number only conveys "which position," not "what is
different about this case."

- First, look for **a meaningfully different value** across cases. Even for
  objects/instances, if there is an attribute that distinguishes the cases
  (a config value, a kind, a key, etc.), display it via a key path like
  `$input.<field>.<prop>`. For example, if the difference is in a config
  value of an instance, write
  `test.each(cases)('<prop>: $input.<field>.<prop>', ...)`.
- Using `$#` is acceptable **only** when the value has **no readable
  identifier to begin with** (the contents collapse and there's no way to
  distinguish them, e.g. two empty `WeakMap`s). Even then, do **not fabricate**
  a readable index (don't invent `{ id: 1 }`, `{ id: 2 }` and distort the type).
- Even when the argument type is opaque, **pass a real instance matching the
  declared type** ("prepare a real value matching the declared type" in
  [types.md](./types.md)). Don't substitute `{ id: 100001 }` and paper over
  the type with `/** @type {*} */`; pass the real thing, like `new WeakMap()`.

```js
// No distinguishable readable property exists (two empty WeakMaps) -> use $# as a last resort
describe('.ensureCtorPool()', () => {
  describe('should be a WeakMap', () => {
    const cases = [
      {
        input: {
          weakMapKey: new WeakMap(), // real instance; { id: ... } would not satisfy the type
        },
      },
      {
        input: {
          weakMapKey: new WeakMap(),
        },
      },
    ]

    test.each(cases)('weakMapKey: $#', ({ input }) => { // no distinguishable value, so use the index
      const received = BoundCtorRegistry.ensureCtorPool(input)

      expect(received)
        .toBeInstanceOf(WeakMap)
    })
  })
})
```

```js
// Even for opaque instances, if there is an attribute that distinguishes the cases, display it (don't use $#)
test.each(cases)('<prop>: $input.<field>.<prop>', ({ input, expected }) => {
  // ...
})
```

## When a sample string needs to be a "run of words," mechanically line up alpha, beta, … omega

When a sample string needs to be **a run of words (tokens)** (words joined by
a delimiter, a sequence of multiple tokens, i.e. cases where the index-suffix
scheme above doesn't fit as a shape), assign **`alpha`, `beta`, `gamma`, …
`omega`** to the word tokens **mechanically, in this order**. Picking real
words at random (one element being `one_two_three`, the next
`four_five_six`, the next `hotel_india_juliet`, …) has no pattern, and the
reader can't track "which word, in which position, of which case." Using the
Greek letters in order lets the reader grasp the sequence just by reading top
to bottom.

- For cases that need multiple tokens, consume `alpha` onward **in order
  across the whole `cases`** (2 words → `alpha`/`beta`; if the next case
  needs 3 words → `gamma`/`delta`/`epsilon` …).
- When treating **just a single token as representative** (a placeholder
  where the content doesn't matter and "some one word" is enough), use
  **`omega`**. Do not use conventional names like `foo` / `bar`.
- For values you want to identify by reading the index (a sequential `id`,
  etc.), use the conventional `100001…` / `'source-0001'…` index-suffix
  scheme (top of this file) as before, not Greek letters. The Greek-letter
  scheme is only for strings that need to "look like a run of words."

## Show intent for strings that carry meaning, either via the string itself or an English inline comment

When a string value is used as **a value carrying a specific intent (role,
kind)** (not just an indexed placeholder, but a case where you're **giving
meaning** to the string, such as "a value that is not a constructor" or "an
unparseable date string"), supplement that intent **in English**, via one
of the following:

- **Make the string itself descriptive**: make the value such that reading it
  reveals the role (e.g. `'not-a-constructor'` / `'unparseable-date'`). Don't
  leave it as a meaningless indexed value like `'text-0001'`.
- **Add an English inline comment**: when the value itself cannot be made
  descriptive (or you don't want to change it), supplement the role with
  `// ...`.

- Indexed placeholders (`'text-0001'` / `'source-0002'`) and Greek-letter
  tokens (`alpha` / `omega`) are used for "a representative value whose
  content doesn't matter," which is a separate concern from this convention.
  Only for strings you want to **carry specific meaning**, show the intent
  through a descriptive value or a comment.
- Comments should be written in English, per "Write comments in the code in
  English" in [SKILL.md](../SKILL.md).

```js
// Good: the string itself conveys the role
const cases = [
  {
    input: {
      handler: 'not-a-function', // verifies the fallback for a non-function value
    },
  },
]

// Good: when the value itself can't show it, supplement the intent with an English inline comment
const cases = [
  {
    input: {
      value: 'text_0001', // an unparseable date string (a hyphen would be parsed as a year)
    },
  },
]
```

## If the set of possible values is finite and small, write out all cases

If the possible inputs (or branches) of a member are **finite** and
**small in number** (a range a human can hold in memory), don't sample
representative values — **enumerate all cases**. The criterion for "small" is
**"whether a human can memorize it."** As a rule of thumb, up to roughly
**100 cases** counts as "all cases."

- Examples of finite and small: the set of special characters requiring
  regex-escaping, enums, boolean combinations, days of the week, HTTP
  methods, and other domains that **can be exhaustively enumerated**. If you
  sample only representative values from these, a broken branch for an
  omitted value would go unnoticed.
- Infinite/large inputs (arbitrary strings, the full range of numbers, etc.)
  cannot be exhaustively enumerated, so **sample** representative values plus
  edge values as before ([list normal values first](#list-normal-values-first)).
- Decision procedure: consider whether the set of values accepted by the
  target member can be enumerated. If it can be enumerated and is small
  enough for a human to memorize, write all cases; otherwise, sample.
- Sampling only part of a finite set means a broken branch for an omitted
  value would still pass the test. If there is a separate, paired infinite
  set (e.g. "the enumerable target" vs. "everything else"), sample that one
  with representative values instead.

## Choose values that exercise the internal transformation (don't fill cases with no-op values only)

When a member **doesn't use a property or argument as-is, but transforms it
once internally** (escape / normalize / trim / encode / substitution, etc.),
always include in the cases **a value for which the output would change if
the transformation were not applied**. Testing only with values for which the
transformation is a pass-through (no-op) means the test would still pass even
if the transformation logic were broken or removed, failing to verify that
the transformation exists at all (a concretization of the QA principle in
[SKILL.md](../SKILL.md)).

- Decision procedure: check whether, in the target member's code, the
  property/argument is used "as-is" or processed once before use (passed
  through another method, embedded into a regex, used as the target/
  replacement of a substitution, etc.). If there is processing, include at
  least one case with **a value for which the result changes depending on
  whether that processing happened**.
- Don't fill `cases` with **no-op values** only (values where the result is
  the same whether or not the transformation is applied). It's fine to
  include a no-op value as a typical case, but you must also add **at least
  one value that actually exercises the transformation**.
- Examples of "values that exercise the transformation," by transformation
  kind:
  - Escaping before embedding into a regex → **regex special characters**
    (`.` `$` `*` `+` `?` `(` `)` `[` `]` `{` `}` `|` `\` `^`). If not escaped,
    they misbehave as metacharacters.
  - Trim / whitespace normalization → **values containing whitespace**
    (leading/trailing or consecutive).
  - Uppercasing / lowercasing → **values that originally contain the
    opposite case**.
  - Encoding (URL / Base64, etc.) → **values containing characters subject to
    encoding**.

### For a terminal method that performs the transformation, write all cases; for the caller, sample (at least one per path)

Even for the same transformation, the granularity of verification varies by
**where the member sits**.

- **The terminal/helper method that implements the transformation** (the one
  that holds the transformation logic itself): if the set of transformation
  targets is finite and small, write **all cases**
  ([If the set of possible values is finite and small, write out all cases](#if-the-set-of-possible-values-is-finite-and-small-write-out-all-cases)).
  Since this is the final place that guarantees the correctness of the
  transformation, leave no gaps here.
- **The caller of the terminal method** (the side that internally calls the
  terminal method and uses its result): exhaustive enumeration is unnecessary
  (correctness is already guaranteed at the terminal). However, sample **at
  least one value where the transformation takes effect, and at least one
  no-op value**, respectively. This is to confirm that both paths are wired
  correctly (i.e., that the value is actually routed through the terminal
  before use) — with only one, a break in the other path would go unnoticed.

## For features that handle integers, always test near the `Number.MAX_SAFE_INTEGER` threshold

For tests of features that involve integers (integer scalars, integer
validation, anything with a safe-integer check like `Number.isSafeInteger`),
always include a case **near the `Number.MAX_SAFE_INTEGER` threshold**.
JavaScript's `Number` can only safely represent integers up to `2 ** 53 - 1`
(`Number.MAX_SAFE_INTEGER` = 9007199254740991), and this boundary is the most
fragile and meaningful edge for integer features.

- Place the boundary itself (`Number.MAX_SAFE_INTEGER`) on the **valid
  (normal)** side, and one past the boundary
  (`Number.MAX_SAFE_INTEGER + 1`) on the **invalid (not a safe integer)** side.
- Don't hard-code a giant numeric literal for the value — use the
  **expression `Number.MAX_SAFE_INTEGER` / `Number.MAX_SAFE_INTEGER + 1`**
  directly as the value (readable, and no loss of precision. Since these are
  known named constants, you can write the expression itself rather than the
  "literal + derivation-expression comment" form from [Write derived values
  and magic constants as literals…](#write-derived-values-and-magic-constants-as-literals-with-the-derivation-expression-added-as-a-comment)).
- Add this boundary **in addition to** typical values (small integers like
  `100001`). With typical values alone, an implementation that mistakenly
  used `Number.isInteger` instead of `Number.isSafeInteger` would still pass
  (the QA stance in [SKILL.md](../SKILL.md)).
- For features that **receive integers as strings** (e.g. the string
  representation of a `bigint`), write values beyond the threshold as string
  or `BigInt` literals ([Write derived values and magic constants as
  literals…](#write-derived-values-and-magic-constants-as-literals-with-the-derivation-expression-added-as-a-comment)).

```js
describe('SomeIntegerScalar', () => {
  describe('#normalizeValue()', () => {
    describe('with valid values', () => {
      const cases = [
        {
          input: {
            value: 100001, // typical value
          },
          expected: 100001,
        },
        {
          input: {
            value: Number.MAX_SAFE_INTEGER, // boundary of a safe integer
          },
          expected: Number.MAX_SAFE_INTEGER,
        },
      ]
      // ...
    })

    describe('with invalid values', () => {
      const cases = [
        {
          input: {
            value: Number.MAX_SAFE_INTEGER + 1, // just outside the safe range → not a safe integer
          },
        },
        // ...
      ]
      // ...
    })
  })
})
```

## For variable-count elements, cover the count boundaries down to (…3, 2, 1, 0)

When a member processes **a variable number of elements** (words joined by a
delimiter, elements of an array/list, repetition, etc.), align not only the
typical multiple-element case but also the **reduced-count boundaries**
(3, 2, **1**, **0**). Element-count boundaries are fragile against branching
and regex boundary conditions, and multi-element cases alone cannot verify
the paths for "no delimiter at all (1 element)" or "empty (0 elements)."

- **0 elements**: empty string `''`, empty array `[]`, input with no elements.
- **1 element**: a single element with no delimiter. There's no processing
  between elements (delimiting, joining, etc.), so this often goes through a
  different path than the multi-element case (no transformation, pass-through, etc.).
- **2 or more**: the typical case.

### Treat count boundaries not as an "independent describe" but as symmetric combinations with existing axes

Variations in count must **not be carved out into their own independent
behavior describe or separate axis**. Extracting only the boundaries (1
element, 0 elements) into a dedicated describe actually creates an asymmetry —
"why is count treated specially on its own?" Count is merely **a variation of
the input value**, so lay it out **symmetrically** within a **grid combined
with the existing axis** (the other property/argument), for each axis value.

- If another axis already exists, give **the same count range for each of
  its values**. Avoid adding the boundary only to one specific axis value
  (fixing one side, as described under [When multiple properties are
  involved in the output](./structure.md#when-multiple-properties-are-involved-in-the-output-property--property)),
  since that produces asymmetry.
- Don't reason "the boundary produces the same output regardless of the
  other axis, so it should be separated out." Even with some duplication,
  aligning the **axis × count** grid keeps the structure symmetric and
  readable, and gaps are less likely.
- Ordering the counts from largest to smallest within each inner `cases`
  groups the boundaries at the end, making them easier to read (the same
  idea as [list normal values first](#list-normal-values-first)).

## Split cases where a structural marker sits at a special position into a separate describe

When a member consumes **structural markers** embedded in the input
(delimiter/separator/quote/escape characters — characters that indicate
token structure) as it processes them, verify cases where the marker sits at
a **special position**. These go through **a different branch** from the
usual "a marker sits between tokens" case (matching at the start / no
following token at the end / a token being empty because it's only a marker),
and often produce non-obvious results.

- **Start**: a marker at the start of the input.
- **End**: a marker at the end of the input (no following token).
- **Both ends**: markers at both start and end.
- **Marker only**: no tokens at all, just markers (one or several).

### Treat this separately from count boundaries (reason for a separate describe)

[Count boundaries](#for-variable-count-elements-cover-the-count-boundaries-down-to-3-2-1-0) are a
continuous variation of the input value, so they get folded into the grid.
A marker's special position, on the other hand, is **a qualitatively
different edge** (a different path, a non-obvious result), so it is split
into an independent behavior describe to make the intent explicit. Mixing it
into the describe for normal processing would blur what is being verified.

### The marker's value can change the behavior

Even at the same special position, the result can branch depending on the
marker's **value** (whether it belongs to a character class, whether it is a
regex special character, etc.). Use representative values that cause the
behavior to branch as an axis (`describe.each()`), and verify symmetrically.

## Don't fill cases with edge cases only (always include typical values)

Don't fill `cases` with **edge cases only**. If you line up only boundary,
special, and abnormal values and drop the **typical (most ordinary) input**,
you cannot verify that "normal usage works correctly," and the test bypasses
the main use case. Put in typical values first, then add edges.

- A common mistake: in an eagerness to **highlight** some behavior (the
  effect of a flag, a special branch, etc.), you end up choosing only the
  special inputs that show that difference. Inputs that show the difference
  are necessary, but they must not be the **only** ones. Also include a
  **typical input** for which there is no difference, and pin down the
  result for the ordinary case too.
  - For example, in a describe split by a mode flag (structure.md's "separate
    mutually-influencing arguments with `describe()`"), there's a tendency to
    skew toward special inputs where the flag has an effect. Also add a
    **typical input unaffected by the flag** for each mode, pinning down that
    "for typical inputs, the result is the same regardless of mode."
- Decision procedure: look over `cases` and check whether it includes "the
  input this member most commonly receives in real-world use." If not, add a
  typical value (placed at the front — see the next section, "List normal
  values first").

## List normal values first

Within a single `cases` array, place **representative normal values first**,
and edge values such as `null` / `undefined` / empty arrays **later**. The
reader first grasps the intent from the typical cases, and can check the
edges together at the end.

```js
// Good example: normal values (arrays, functions) first, null at the end
const cases = [
  {
    input: {
      replacer: alphaReplacer,
    },
  },
  {
    input: {
      replacer: betaReplacer,
    },
  },
  {
    input: {
      replacer: null,
    },
  },
]
```

```js
// Avoid: placing null first buries the typical cases
const cases = [
  {
    input: {
      replacer: null,
    },
  },
  // ... alphaReplacer / betaReplacer follow
]
```

If type-violating irregular values are also included, use the Separate
Valid / Invalid Values split in [structure.md](./structure.md) to separate
`describe()`s instead of this ordering. The ordering here is only about the
order **among valid values that live within the same `describe()`**.

## At most one property key per line

Format `cases` objects so that **no more than one property key (`foo:`)
appears on a single line**.

- Structural containers (where the value of `input` / `tally` / `override`
  is an object) must **always be broken onto new lines**, with child
  properties on separate lines. Don't put parent and child on the same line
  like `input: { replacer: X }`.
- Leaf data values (payload objects held by things like `value` /
  `valueHash`) **may be inline if there is a single key**
  (`valueHash: { id: 100001 }`). **Expand if there are multiple keys**.
- When placing multiple objects inside an array too, **expand each element
  onto its own line**. If an element has just a single key, it may be inline.

```js
// Avoid: placing input and replacer on the same line (inlining a structural container)
const cases = [
  { input: { replacer: alphaReplacer } },
]

// Avoid: placing value and id / name on the same line (inlining a multi-key payload)
const cases = [
  {
    input: {
      value: { id: 100001, name: 'Alpha' },
    },
  },
]

// Avoid: placing multiple objects on one line inside an array
const cases = [
  {
    input: {
      value: [{ id: 100001 }, { id: 100002 }],
    },
  },
]
```

```js
// Good example: break structural containers onto new lines, expand multi-key payloads and multiple elements
const cases = [
  {
    input: {
      value: {
        id: 100001,
        name: 'Alpha',
      },
    },
  },
  {
    input: {
      value: [
        { id: 100001 }, // single-key elements may be inline
        { id: 100002 },
      ],
    },
  },
]
```

## Don't put branches into sample functions

For **sample callback functions** passed into `cases` (`replacer` /
comparator, etc.), avoid **branches (`if` / ternary)** as much as possible,
and keep them a simple transformation. What the test wants to confirm is
"whether that function is passed and applied correctly," not the logic
inside the function itself, so a branch becomes noise and makes it look like
you're verifying the sample function's own behavior.

```js
// Good example: a simple transformation with no branching
const alphaReplacer = (key, value) => `*${value}`
```

```js
// Avoid: includes branching logic (unclear what is being verified)
const alphaReplacer = (key, value) =>
  key === 'secret'
    ? undefined
    : value
```
