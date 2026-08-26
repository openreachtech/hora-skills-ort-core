---
name: hoc-naming
description: "Naming conventions. Covers naming of classes, methods, properties, and accessors, datetime suffixes (`At` / `On`, plus `From` / `To` for ranges), criteria for abbreviations, American spelling, forbidden words, and the prohibition of non-ASCII characters."
---

# Shared: Naming

Summarizes naming conventions.

## Class names

- Class names are written in UpperCamelCase.
- Use the singular form of a noun.
- Reason: if a plural noun is used, you cannot define a variable name for "an array holding instances of that class."
  This is because there is no way to express "the plural of instances of a plural-named class."
- When no specific class name is dictated, name the class to describe the logic it encapsulates.

```javascript
// NG
class Users {}
const users = [new Users(), new Users()] // Cannot express "plural of instances of a plural class"

// OK
class User {}
const users = [new User(), new User()]
```

## Name variables and properties holding arrays as plural nouns

- A variable or property that holds an array (a value with multiple elements) is always named with "the plural form of the noun denoting its element."
- This is the counterpart of the "class names are singular" rule (see "Class names"). If the element's class is the singular `User`, its array is the plural `users`.
- Do not express plurality with a `List` suffix (see `list` in [references/prohibited-words.md](./references/prohibited-words.md)).

```javascript
// NG: named singular despite holding an array / expressing plurality with a List suffix
const user = users.filter(it => it.enabled) // holds an array yet named singular
const paymentList = payments.filter(it => it.completed)

// OK: plural form of the noun denoting the element
const enabledUsers = users.filter(it => it.enabled)
const completedPayments = payments.filter(it => it.completed)
```

## The `item` parameter of higher-order function callbacks

- For a function passed to a higher-order function, use `it` as the parameter name for receiving each item.
- If a higher-order function is called inside another higher-order function, using `it` for the inner item too would be confusing, so name the inner item's parameter according to the meaning of its value.
- The first-layer callback argument should basically use `(it, index, array) => ...`.
- For `reduce()` and `reduceRight()`, name the first argument (the accumulator) appropriately based on the meaning of the value being accumulated (e.g. `total` for a running sum).

```javascript
// OK: item parameter is it
const ids = samples
  .filter(it => it.enabled)
  .map(it => it.id)

// OK: name the inner nested item by meaning (outer it / inner user, etc.)
const names = teams
  .flatMap(it =>
    it.members.map(user => user.name)
  )

// OK: name the reduce accumulator by meaning (total for a running sum)
const total = prices
  .reduce((total, it) => total + it.amount, 0)
```

## Naming abstract classes and derived classes

- Prefix abstract classes with `Base~`.
- When extending `BaseSample` for a specific application, add `App` as in `BaseAppSample`.
- The `Base~` and `App~` in `BaseAppSample` are called "super prefixes."
- When naming a derived class, remove the super prefix (`Base~` / `App~`), keep the suffix, and replace it with a distinguishing prefix.

```javascript
// Abstract class
class BaseSample {}

// Abstract class extended for the application
class BaseAppSample extends BaseSample {}

// Derived classes (super prefix removed and replaced with a distinguishing prefix; suffix "Sample" is kept)
class AlphaSample extends BaseSample {}
class BetaSample extends BaseAppSample {}
```

## Method names are "predicates," not "complete sentences"

- A method call `receiver.method()` has a structure resembling a sentence: "subject (receiver) + predicate (method)."
- Therefore, a method name should be named as a **predicate** with the receiver as the subject, not as a "complete sentence with both subject and predicate."

```javascript
// NG: named as a complete sentence (object.isStatusValid() reads as "object is status valid?", which is unnatural)
isStatusValid () {
  return this.status !== null
}

// OK: named as a predicate (object.isValidStatus() reads as "object is valid status")
isValidStatus () {
  return this.status !== null
}
```

### Property syntax and "messages to the receiver"

- In the property syntax `object.alpha`, each token is called: `object` = receiver, `.` = dot operator, `alpha` = property.
- A method is a property in the broad sense. It is a property whose value is a function, invoked with `()`; `object.methodName` is property access, and `object.methodName()` is the notation for executing that value "as a function."
- Hence a method call is a message where "receiver = subject, method name = predicate," and follows a sentence-like structure such as `object.doSomething()` (object does something) / `object.isValid()` (object is valid).
- `#isStatusValid()` is wrong because it treats the method name as "a complete sentence with subject and predicate combined (Is status valid?)." Calling it via a receiver reads as "object is status valid?", which is unnatural. Name it with only the predicate (`#isValidStatus()` → "object is valid status").

### Method names start with a verb

- Since a method name is a predicate, it is natural, from an English grammar standpoint, for it to "start with a verb" (e.g. `#hasEntity()` / `#findUserProfile()`).
- It may also start with an auxiliary verb (e.g. `#canFindEntity()` / `#shouldHaveFulfilledInput()`).

### Third-person singular verb → convention that the return value is boolean

- Starting a verb in the third-person singular form (or with an auxiliary verb) carries the convention that "the return value is boolean" (`is...()` / `has...()` / `can...()` / `should...()`, etc.).
- This originates from Java (Sun Microsystems) coding conventions from the late 1990s, which spread to later languages.

```javascript
object.isValid() // boolean
object.containsInvalidTag() // boolean (third-person singular contains → boolean)
```

- This originates from Java conventions established by Sun Microsystems in the late 1990s (methods returning boolean start with a third-person singular verb or an auxiliary verb: `is...()` / `has...()` / `can...()` / `should...()`, etc.), which spread as a standard to later languages.
- Conversely, if a predicate is named correctly, the return type can be inferred even without JSDoc (e.g. `object.lovesWithoutConditions()` → readable as boolean because it's a third-person singular verb).

## Criteria for using abbreviations

- The criteria and whitelist (shared between JS and CSS) for which abbreviations may be used are collected in [references/abbreviations.md](./references/abbreviations.md).

## Do not use non-ASCII characters in names

- Do not use non-ASCII characters in variable names, class names, member names, etc.

```javascript
// Never use
class 😂 {
  取得 () {
    return null
  }
}
```

## Avoid single-word class names wherever possible

- Avoid single-word class names wherever possible; use a compound name containing two or more words.
- If this is pointed out in a review, always change it to a compound word to save discussion time.

## Use American spelling

- The correspondence between American and British spellings is collected in [references/spelling.md](./references/spelling.md).

## Prohibited words and redundant compound words

- Words that must not be used in naming (as prefixes/suffixes) and the redundant compound words that contain them are collected in [references/prohibited-words.md](./references/prohibited-words.md).

## Property names

- Property names are, in principle, nouns. They are acceptable if they are clear.
- Array properties should be pluralized (`#payments` correct / `#paymentList` incorrect). See "Name variables and properties holding arrays as plural nouns."

## Datetime suffixes: `At` for an instant, `On` for a date

- A value that carries a **time of day** ends with `At` (`modifiedAt`, `trashedAt`, `expiredAt`).
- A value whose meaning stops at the **calendar date** ends with `On` (`billedOn`, `dueOn`).
- A range keeps the suffix and appends `From` / `To` (`modifiedAtFrom` / `modifiedAtTo`, `billedOnFrom` / `billedOnTo`).
- Reason: the suffix is the only place the granularity is stated. Dropping it (`modifiedFrom`) forces every reader to open the column definition to learn whether a time component exists, and a date-only value silently compared against an instant is off by up to a day — a bug that only appears near midnight.
- `From` / `To` mark **the two ends of a range**. A single value meaning "in effect from this moment" is not a range end; name it for the instant it holds (`effectiveAt`), not `effectiveFrom`.

```javascript
// NG
const modifiedFrom = ...  // does the value carry a time of day? the name does not say
const billed = ...        // a date or an instant?
const effectiveFrom = ... // a single instant named as if it were a range end

// OK
const modifiedAt = ...
const billedOn = ...
const effectiveAt = ...
const modifiedAtFrom = ... // range lower bound, suffix kept
const modifiedAtTo = ...   // range upper bound
```

## Method names

- Method/function names should basically start with the base (present tense) form of a verb (e.g. `findUsers()` / `deleteUsers()`).
- Starting with a particle (such as a preposition) is allowed, but should be avoided when used alone wherever possible (`fromOpenedAt()` correct / `from()` incorrect).
  - Exception: inflator methods adopt preposition-like short names as their canonical vocabulary (`.from()` / `.of()` / `.by()`, etc.). See the inflator-methods convention for details.
- A single-word name for an instance method is basically prohibited (`#load()` doesn't reveal what is being loaded; use `#loadCardNumbers()`).
- For static methods, since the class name supplements the context, a single-word name is exceptionally allowed (`CardNumbersLoader.load()`).
- For a method name that serves as an entry point, interpret it as a transitive verb wherever possible and give it an object (e.g. for `ChunkBuilder`, use `#buildChunks()` instead of `#build()`).
  - Reason: if the instance is held in a short-scoped variable or property, the context supplementation from the class name is lost. If held as in `const builder = ChunkBuilder.create()`, then `builder.build()` doesn't reveal what is being built. Including the object, as in `builder.buildChunks()`, conveys it to the reader.

## Accessors (getter / setter)

- Getter names should in principle be nouns.
- Naming a getter with a "boolean method name" that starts with a third-person-singular verb or an auxiliary verb is also permitted.
- Since setters are prohibited, there is no naming rule for them (see the accessor-definition convention).

## Method name verbs (choosing among highly abstract verbs)

- How to choose among highly abstract verbs (creation `create` / `generate` / `build`, retrieval `find` / `fetch` / `extract`, update `save` / `send` / `set`) is collected in [references/verbs.md](./references/verbs.md).

## Contrasting terms

- The fixed pairs to use for variable/member names with opposite meanings are collected in [references/antonyms.md](./references/antonyms.md).
