---
name: hoc-dependency-defect
description: "How to deal with a bug in code this project uses but does not own — a framework, an in-house package, anything inside `node_modules/`. Covers where to put the workaround, what to name it, and how to mark it so it can be deleted later. Use when a package behaves wrongly and the correction has to live in this project. A bug in code this project owns is fixed directly instead."
---

# Dependency Defect

**Never edit code this project uses but does not own. Work around it instead.**

Code this project does not own means a framework, an in-house package, or anything inside
`node_modules/`. This skill explains how to write that workaround and where to put it. **It does not
decide who approves the work, which branch it goes on, or what gets written down outside the code.**
The team's process decides those.

**A bug in code this project owns is a different thing.** Fix that code directly.

## The steps

0. **Check whether the package already fixed it.** Read the package's own history: the version in
   use, and every version after it. If the fix is already there, upgrade the package and stop.
1. **Add a class in this project that extends the broken class.**
2. **Override only the member that is broken.**
3. **Use the new class at every call site.**
4. **Write a comment saying why the class exists**, so someone can delete it later.

**Step 0 is the one people skip.** Then they write a subclass that repeats a fix the package already
shipped. Reading the history takes a few minutes.

## Why a subclass, and not something else

| What people do instead | What goes wrong |
| :-- | :-- |
| Edit the package inside `node_modules/` | The next install wipes the edit. Nothing shows that it was ever there, so the bug comes back and nobody knows why. |
| Fork the package | Every future upgrade turns into a merge. Forever, for one bug. |
| Rewrite what the package does | As much work as a fork, and now this project owns all of it. Fixes to the parts that were never broken stop arriving. |
| Replace the class's methods at runtime | The call site looks normal, so a reader cannot tell that the behavior changed, and has no file to open. If two of these run in one process, whichever one loads last wins. |
| Copy the class into this project and fix the copy | The same problems as a fork, in one file. |

**A small bug is not an excuse to pick one of these.** The rule holds because the code belongs to
someone else, and they will change it again.

## Step 1 — add a subclass of the broken class

The subclass is normal code in this project. It follows the same rules as every other class here.

- **Put it where this project keeps its other classes of that kind.** Never create a folder named
  after the package, and never put the file next to the package's own files.
- **Name it after what it does**, as a singular UpperCamelCase noun, like any other class here. Do
  not use names like `PatchedXxxx`, `FixedXxxx` or `XxxxWorkaround`. Those names describe how the
  file was born, not what it does, and they become wrong if the class is kept for another reason.
  The reason belongs in the comment of step 4.
- **Import the broken class the way the package exports it** (default or named). Put that import with
  the other package imports at the top of the file, as the import convention says.
- **This subclass does not need a property of its own.** A class here normally has to hold state, but
  that rule does not apply once a class has `extends`. The parent holds the state, and the parent
  already decides the constructor arguments.

## Step 2 — override one member, not the whole class

**Override one member, and the subclass keeps getting every later improvement from the package.**
Rewrite the whole class, and it stops getting them the moment you write it. Nothing warns you.

- Override the one method that behaves wrongly, and call `super` for everything else.
- Override a getter when the parent computes a value wrongly.
- Do not copy the parent's code into the child and change two lines.
- Do not override a member that already works, just to keep the class tidy.
- Put `/** @override */` above every override.

**When the correct answer needs part of the parent's work, call the parent and correct its result.**
Do not rewrite what it does.

```javascript
// NG: the parent's code is copied here, so the package's later fixes never reach this class
export default class SingleDayDateRangeFormatter extends DateRangeFormatter {
  /** @override */
  formatRange ({
    startedOn,
    endedOn,
  }) {
    if (startedOn === endedOn) {
      return startedOn
    }

    return `${startedOn} - ${endedOn}` // copied out of the package, minus the bug
  }
}

// OK: only the broken case is handled here, and the rest stays with the parent
export default class SingleDayDateRangeFormatter extends DateRangeFormatter {
  /** @override */
  formatRange ({
    startedOn,
    endedOn,
  }) {
    if (startedOn === endedOn) {
      return startedOn
    }

    return super.formatRange({
      startedOn,
      endedOn,
    })
  }
}
```

**A class whose name starts with `Base` is meant to be extended.** Extending it is normal use of the
library, not a workaround. Nothing in this skill applies to it: no comment, and no deletion
condition.

## Step 3 — call the new class by its own name

**Every call site names the new class.** A reader who follows the import then lands on the class that
really runs.

- Import this project's class by its own name at each call site.
- Do not re-export the subclass under the original's name.
- Do not rename the import so that the original's name points at the subclass.
- Do not overwrite the original's prototype or its export.

```javascript
// NG: exported under the package's name, so the import lies about what it loads
export {
  default as DateRangeFormatter,
} from './SingleDayDateRangeFormatter.js'

// NG: renamed at the call site, with the same result
import {
  default as DateRangeFormatter,
} from '../modules/SingleDayDateRangeFormatter.js'

// OK: the call site imports this project's class by its own name
import SingleDayDateRangeFormatter from '../modules/SingleDayDateRangeFormatter.js'
```

Hide the swap, and the next person opens the package's source looking for behavior that is not there
any more.

**Many call sites is not a reason to hide the swap.** It is a reason to report how big the change is,
because files outside the current task are not this task's to change.

## Step 4 — write the reason above the class

**Without a comment, the workaround stays forever.** Nobody deletes a class when nobody knows why it
is there.

Write the reason right above the class, in English, like any other comment. Name members in the
`#instanceMember` / `.staticMember` style of the documentation convention. Say what has to happen
before someone can delete the file:

```javascript
/*
 * Works around <package>@<version>: <class>#<member> <what it does wrong>.
 * Remove this class and go back to <original> once <the condition> holds.
 * Reported upstream: <link>, or "not reported yet".
 */
```

**"Remove this once the package fixes it" is not a condition**, because nobody can check it. A
version number or an issue link can be checked. If nobody has told the package authors yet, write
"not reported yet". That is honest, and it tells the next reader what is still missing.

## When the broken thing is a function, not a class

**Wrap it.** Add a class in this project that calls the function and corrects its result, and have
the call sites use that class. The same rules apply: correct the result, do not rewrite the function.
It is a class rather than another function, because this project writes one class per job (see
`hoc-modules-exports`).

## When a subclass cannot reach the broken part

Some parts cannot be overridden: private members, values kept inside the module, or behavior decided
before the class is created. **Stop and report it.** Do not fall back to a row from the table above
just because the subclass did not work.

Report the package and its version, the class or function, what it does now, what it should do, and
what exactly blocks the subclass.

## Rules

- Never edit, fork, copy, rewrite, or patch the package while it runs
- Read the package's history first: the version in use, and every version after it
- Correct the bug with a class in this project that extends the broken class, overrides only the
  broken member, and calls the parent for the rest
- Correct a broken function with a class in this project that wraps it
- Call this project's class by its own name, and never let the original's name point at it
- Report a broken part that a subclass cannot reach, instead of working around it some other way
- Give every workaround a comment stating the package, the version, the bug, and what has to happen
  before the file can be deleted
