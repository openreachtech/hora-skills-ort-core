---
name: hoc-classes-member-notation
description: "How a class member is written when it is referred to — `#` for an instance member, `.` for a static one, `get:` / `set:` for an accessor, and the class name in front where it is needed. Use whenever a member is named outside its own declaration: in a document, a commit message, a test's `describe()`, an error message or a comment."
---

# Classes: Member Notation

How a class member is written when something refers to it. The notation is the one place this
library spells a member out, and every other convention that names a member uses it.

## The notation

- Wherever a class member is referred to — in prose, in a commit message, in a test's `describe()`, in an error message or a comment — use the following notation.
- Instance members are prefixed with `#`, static members with `.`.

| notation | members |
| :-- | :-- |
| `#instanceProperty` | instance property |
| `#instanceMethod()` | instance method |
| `#get:instanceGetter` | instance getter |
| `#set:instanceSetter` | instance setter |
| `.staticProperty` | static property |
| `.staticMethod()` | static method |
| `.get:staticGetter` | static getter |
| `.set:staticSetter` | static setter |

- **Where the kind of member does not matter, write `#instanceMember` / `.staticMember`.** A rule that holds for every member alike turns on the prefix alone, and naming a kind it does not depend on would narrow it.
- When attaching the class name, write it as in `SampleClass#extractValue()`.

| notation | member |
| :-- | :-- |
| `SampleClass#extractValue()` | instance method of `SampleClass` |
| `SampleClass.createValue()` | static method of `SampleClass` |

## It applies beyond prose

- This notation applies not only to Markdown prose, but to **any text within implementation code that refers to a class member**. Specifically, this includes the following.
  - **Error messages** (message strings in `throw new Error(...)`, etc.)
  - **JSDoc / comments** (places within a member's description that refer to a member)
- When dynamically embedding a class name, use the same notation. Prefix instance members with `#` and static members with `.`.

```javascript
// OK: instance method (the class name is resolved via this.constructor.name)
throw new Error(`${this.constructor.name}#normalize() must be inherited`)

// OK: static getter / static method (the class name is resolved via this.name)
throw new Error(`${this.name}.get:rawSchema must be inherited`)
throw new Error(`${this.name}.generateCredential() must be inherited`)
```
