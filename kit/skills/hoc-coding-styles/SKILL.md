---
name: hoc-coding-styles
description: "Coding style conventions. Covers where to chop down (wrap) expressions, method/property chains, function-call arguments, template literals, regular-expression flags, and more."
---

# Coding Styles

Conventions related to coding style.

## Chop down immediately before a binary operator

- When a line contains multiple expressions, chop down immediately before the binary operator (place the operator at the start of the next line).
- Literals are not considered expressions here. Therefore, when one of the operands is a literal (e.g., `length + 1`), do not chop down.

```javascript
// NG
const total = alpha + beta + gamma

// OK
const total = alpha
  + beta
  + gamma
```

### Chop down `??` as a branch

- Strictly speaking, `??` is syntactic sugar for an `if` statement / ternary operator (a branch). Therefore, the literal exception for binary operators does not apply, and it is **always chopped down as a branch** (placing `??` at the start of the next line).
- In other words, even when the right operand is a literal (such as `null`), `?? null` is not placed on the same line — it is wrapped onto a new line.

```javascript
// NG: applying the literal exception and keeping it on the same line
const author = this.entity.comment?.author ?? null

// OK: chop down immediately before `??` as a branch
const author = this.entity.comment
  ?.author
  ?? null
```

## When an if statement's condition spans multiple lines, apply consistency chopping down to `()`

- When an if statement's condition spans multiple lines, apply consistency chopping down to the condition's `()`.
  (Break the line immediately after `(`, place each condition on its own line, and place `)` on its own line.)

```javascript
// NG
if (!Number.isInteger(length)
  || length < 0) {
  // ...
}

// OK
if (
  !Number.isInteger(length)
  || length < 0
) {
  // ...
}
```

## Write one method per line in a method chain

- Write one method per line.
- When writing a method chain, allow at most one receiver per line, and at most one method per line.
- `this` does not count as a receiver.
- Unless instructed to chop down, place the first method of a method chain on the same line as the receiver.
- **Counting receivers and methods does not settle a chain on its own.** The count of complex
  expressions on the line governs it too, and a chain can pass this rule while failing that one.
  Read both before deciding where a chain breaks.

```javascript
// NG
const ids = this.extractSamples().filter(it => it).map(it => it.id)

// NG (not one method per line)
const ids = this.extractSamples()
  .filter(
    it => it
  ).map(
    it => it.id
  )

// OK
const ids = this.extractSamples()
  .filter(it => it)
  .map(it => it.id)
```

### Where the chain itself carries the complex expressions, chop it

**A receiver derived by a call, or reached through a computed key, is a complex expression of
its own** — and the method called on it is a second. One receiver and one method sit within
the count this section states, so this rule permits the line; the rule against two complex
expressions on one line is what forbids it.

**The fix there is to chop the chain, not to name a variable.** Chopping already secures the
readability the naming would have bought, and introducing a variable for what a line break
handles is what the rule against meaningless assignments turns away.

```javascript
// NG: two calls on one line — the key's, and the method's
return entity[resolveKey(this.ModelCtor.name)].toSorted(this.getSorter())

// NG: a variable that chopping made unnecessary
const key = resolveKey(this.ModelCtor.name)

return entity[key]
  .toSorted(this.getSorter())

// OK: the chain carries them, so the chain is where it breaks
return entity[resolveKey(this.ModelCtor.name)]
  .toSorted(this.getSorter())
```

### One-line and chopped chains stand together

**A file holding both is not inconsistent for holding both.** Every line is decided by what it
carries, so two chains of the same length break differently the moment one of them derives its
receiver and the other does not.

```javascript
// One line: the receiver is a plain identifier, and the method is the only call
const sorted = array.toSorted(sorter)

// Two lines: the receiver is a call, so the method is the second one on the line
const [latest] = this.extractStatuses()
  .toSorted(comparer)
```

Reading the shorter one as a breach of the longer one's shape is the mistake this note exists
to prevent. Where the two differ, look for what the receiver costs before calling it a
divergence.

## Write one property per line in a property chain

- Write one property per line.
- A receiver is not considered a property. `this.xxxx` is considered a receiver.
- This criterion also conforms to the "Law of Demeter".

```javascript
// NG
const id = this.entity.comment.author.id

// OK
const username = this.entity.username

const id = this.entity.comment
  .author
  .id
```

## Chop down when the same punctuation mark repeats consecutively

- Apply chop down when the same punctuation mark occurs consecutively.
- This is a habit to avoid making the reader count the nesting of `()` or `{}`.

```javascript
// NG
console.log(Math.round(average(SCRIPTS.filter(it => it.living)
  .map(it => it.year))))

// OK
console.log(
  Math.round(
    average(
      SCRIPTS.filter(it => it.living)
        .map(it => it.year)
    )
  )
)
```

## For function calls with two or more arguments, write one argument per line

- When calling a function with two or more arguments, limit arguments to one per line (chop down).

```javascript
// NG: two arguments on the same line
this.delimiter.replace(specialCharacterPattern, '\\$<special>')

// OK: one argument per line
this.delimiter.replace(
  specialCharacterPattern,
  '\\$<special>'
)
```

## Do not make meaningless variable assignments when chop down alone secures readability

- If chop down secures the readability of the expression, do not assign it to a meaningless variable.

```javascript
// NG: assigning to a meaningless variable when chop down alone would suffice
const specialCharacterPattern = /(?<special>[.*+?^${}()|[\]\\])/ug

return this.delimiter.replace(
  specialCharacterPattern,
  '\\$<special>'
)

// OK: secure readability with chop down, without a variable assignment
return this.delimiter.replace(
  /(?<special>[.*+?^${}()|[\]\\])/ug,
  '\\$<special>'
)
```

## Do not write two or more complex expressions on one line

- Do not write two or more complex expressions on a single line.
- When there are multiple complex expressions, split them by naming them with `const`, etc., so that there is at most one complex expression per line.

```javascript
// NG: two complex expressions on one line (two method calls)
return `${segment.charAt(0).toUpperCase()}${segment.slice(1).toLowerCase()}`

// OK: split each into its own const
const head = segment.charAt(0)
  .toUpperCase()
const tail = segment.slice(1)
  .toLowerCase()

return `${head}${tail}`
```

## Do not create unnecessary intermediate strings

- In "string → string" conversions, do not create unnecessary intermediate strings.
- By making full use of regular expressions, most conversions can be done in a single pass (a single `replace` / `replaceAll`).

```javascript
// NG: creating an intermediate string with toLowerCase() and then calling replaceAll (2 passes)
return text.toLowerCase()
  .replaceAll(pattern, replacer)

// OK: convert with a single replaceAll
return text.replaceAll(pattern, replacer)
```

## Use template literals for string concatenation

- For string concatenation, do not use `alpha + beta`; use the template literal `` `${alpha}${beta}` ``.

```javascript
// NG
return head
  + tail

// OK
return `${head}${tail}`
```

## Regular expression literal flags

- Always attach the `u` flag to regular expressions written as literals (mandatory). Place it immediately after the closing `/`.
- Other flags such as `i` or `g` are placed after `u` in alphabetical order.

```javascript
// NG: missing the u flag
const pattern = /[A-Z]/g

// OK: u first, the rest in alphabetical order (g → i)
const pattern = /[A-Z]/ug
const another = /[a-z]/ugi
```
