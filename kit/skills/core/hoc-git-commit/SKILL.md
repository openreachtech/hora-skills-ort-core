---
name: hoc-git-commit
description: >
  Conventions for git commits. Covers what belongs in a single commit and the order commits
  land in, the message format (imperative or Conventional Commits), and the verb vocabulary
  shared by both. The branches commits land on, and the two commits a branch makes about
  itself, belong to the git branch convention; the commands that gate a commit belong to the
  workflows convention. Use before writing a commit message, and before splitting a working
  tree into commits.
---

# Git Commit

Conventions for **what goes into one commit** and **how that commit is described**.

These are two concerns decided at two different moments:

- **Granularity** is decided *while working* — which changes get staged together.
- **Format** is decided *at commit time* — how the staged change is worded.

Granularity comes first. A badly scoped commit cannot be rescued by a well-written subject
line, because the subject is then forced to describe several unrelated things at once.

## Choosing the format

Two message formats are in use across projects. They are **not interchangeable within a
repository** — one repository uses one format throughout its history.

Resolve which one applies, in this order.

1. **The project's `CLAUDE.md`.** A project declares its format under a
   `## Commit message format` heading:

   ```markdown
   ## Commit message format

   - Format: imperative
   ```

   The accepted values are `imperative` and `conventional`.

2. **The existing history**, when `CLAUDE.md` declares nothing. Read the recent non-merge
   subjects and count how many carry a type prefix:

   ```bash
   git log --no-merges -n 30 --pretty=format:'%s'
   ```

   If **most** of them begin with `feat:` / `fix:` / `refactor:` / `test:` / `docs:` /
   `chore:` (optionally with a scope, as in `feat(resolver):`), the project uses
   conventional. Otherwise it uses imperative.

3. **Default to imperative** when the history is too short to judge, as in a new repository.

- Never mix the two formats within one repository. A history that is genuinely split down the
  middle is a defect to raise with the user — it is not a licence to choose per commit.
- When a project's resolved format contradicts what the user asks for in the moment, follow
  the user for that commit, but tell them which format the repository otherwise uses.
- Once resolved, follow the matching detail file:
  [format-imperative.md](./references/format-imperative.md) or
  [format-conventional.md](./references/format-conventional.md).

### The resolved format governs what to write, never how to read

A history may hold subjects in any other convention — written before the project settled on
one, or by another team, or by hand in a hurry — and those commits are still the record of what
happened. So when searching for where something changed, do not filter on the resolved format
alone.

```bash
git log --format='%h %s' <range>                                            # no filter
git log --format='%h %s' <range> | grep -P '^\S+ [a-z-]+(\([^)]*\))?!?: '   # type prefixes
```

Filter the subject line, as above, rather than reaching for `--grep`: that searches the whole
message, so it also matches trailers such as `Co-Authored-By:`. And treat any subject filter as
a shortcut rather than a guarantee — read the range's subjects, or its diff, without one before
concluding that a change is not in the history.

## Rules that apply to both formats

### Subject line

- The subject is a **single line**, and carries **no trailing period**.
- Keep the subject **below 72 characters as much as possible**. This is a target, not a hard
  limit — most subjects should clear it, and a minority legitimately will not.
  - When a subject runs long, the usual cause is that the **commit is doing more than one
    thing**. Reach for [granularity.md](./references/granularity.md) first: splitting the
    commit shortens both subjects and is the fix that actually improves the history.
  - The other legitimate cause is a **long but precise identifier**. Here the target yields:
    naming `BaseRestfulApiLauncher#createResponseBodyParser()` in full is worth more than
    hitting 72. Never truncate, abbreviate, or paraphrase an identifier to fit.
  - What the target rules out is **padding** — the redundant trailing clause that restates
    what the identifier already says. Cut the clause, not the identifier.
- Describe **the change**, not the activity that produced it. `wip`, `save progress`,
  `Update files`, and `Address feedback` describe a working session; they tell a later reader
  nothing about what the tree now does differently. (Two commits are deliberate exceptions: the
  branch-opening marker below, which carries no changes at all, and the throwaway save point in
  [granularity.md](./references/granularity.md), which is deleted before anyone reads it.)
- Do not record **how the change came to be asked for**: no "as requested", no "per review
  comment", no tool attribution. Where the content itself came from is a different matter — that
  belongs in the subject wherever it is part of what the work is.

### The commits a branch makes about itself

Two commits carry no change of their own: the empty marker that opens a trunk, and the
`Merge …` commit that closes a sub-branch. Both are specified in full by the git branch
convention — when to make one, and what its subject says — because what they say is
inseparable from what they are for. Here they surface only in the verb table below, where
`Start`, `Release` and `Merge` are reserved for them.

### Verbs

A subject opens with a verb naming what actually happened. The vocabulary is the same in both
message formats — capitalized on the imperative format, lowercase after the type on
Conventional Commits.

**A verb of two words joins into one where it sits in a single-token slot, and takes no hyphen
in its place.** `Kick out`, `Tidy up`, `Turn on` and `Turn off` stay two words in a subject and
become one before the slash of a branch name — `tidyup/environment-files` — and the same
holds wherever a project puts this vocabulary in the `type:` slot of Conventional Commits rather
than the types listed in [format-conventional.md](./references/format-conventional.md). The
slash and the colon already end the token, so a hyphen inside it marks nothing.

**The table below lists the verbs whose role is fixed, not the verbs a subject may use.** A row
is there because choosing the wrong verb would lose something a later reader needs — whether a
file survived, whether a class or one of its members was written, whether a constraint went one
way or the other. Where a listed verb names what happened, it is the one to use, and no synonym
substitutes for it. Where nothing listed names it, open the subject with the verb that does:
`Name`, `Point`, `Follow`, `Keep` and their like fix no such distinction and need no row.

| verb | use for |
| :-- | :-- |
| `Add` | a new file, member, case, or capability that did not exist |
| `Declare` | a class written for the first time |
| `Define` | a class member, function, or constant written for the first time |
| `Author` | an AI element written for the first time — a skill, an agent, a command |
| `Build` | markup written for the first time — a component, a page, an HTML skeleton |
| `Fulfill` | a gap filled where something was declared but left short — a TODO, an unset option |
| `Purge` | a whole file or folder deleted, with nothing replacing it |
| `Kick out` | a part deleted from what stays — a class member, a section, an entry, a field |
| `Tidy up` | a place brought into order, where nothing was removed and no behavior changed |
| `Update` | an existing thing changed in itself, leaving it better than before |
| `Retake` | an existing thing redone because what was there was poor, hurried, or a stopgap |
| `Fix` | incorrect behavior corrected |
| `Rename` | identifier changed, behavior untouched |
| `Move` | relocation between files or directories, content untouched |
| `Rearrange` | peers put into a different order — imports, declarations, cases, entries |
| `Extract` | logic pulled out into its own member or module |
| `Unify` | two members or modules folded into one |
| `Optimize` | a change made for speed, behavior untouched |
| `Use` | switching to a different existing mechanism |
| `Allow` / `Prevent` | a constraint loosened or tightened |
| `Turn on` / `Turn off` | a switch flipped — a boolean, a lint rule, a feature flag |
| `Adjust` | a dial moved — a threshold, a limit, the options of a rule that stays on |
| `Export` | public surface changed |
| `Install` / `Uninstall` | a dependency the project takes on, or gives up |
| `Start` | **empty** branch-opening marker only — see the git branch convention |
| `Release` | **empty** marker opening a `release/x.x.x` trunk only — see the git branch convention |
| `Merge` | **merge commits only** — see the git branch convention |

- **`Declare` is for the class itself; `Define` is for what is written inside or beside it** —
  a member, a function, a constant. Both are the specific forms of `Add`, and where they apply,
  `Add` is the vaguer choice. `Add` remains correct for everything else that did not exist
  before — a test file, a case, a reference document.

  ```
  Declare SkillsInstaller to replace the installed skills
  Define SkillsInstaller#replaceInstalledSkills() to swap the tree in one pass
  Define SKILL_DOMAIN naming the domains a repository can select
  Add tests for SkillsInstaller
  ```

- **`Author` is the `Declare` of an AI element.** A skill, an agent, a command — whatever is
  read by the agent rather than run by the application, and installs under `.claude/`:
  `Author scribe skill`. Like `Declare` and `Define` it is a specific form of `Add`, and where
  it applies `Add` is the vaguer choice. Editing one that already exists is not `Author` — that
  is `Update`, `Fulfill` or `Tidy up`, whichever names what was done.
- **`Build` is the `Declare` of markup.** A component, a page, the HTML skeleton of one:
  `Build TalkroomMessage component`, `Build HTML skeleton for pages/settings/security.vue`. It
  is the fourth of the family — `Declare` for a class, `Define` for a member, `Author` for an
  AI element, `Build` for markup — and each is a specific form of `Add`. **It never means
  running a build.** `npm run build` produces output, and output is not what a subject
  announces; a commit that changed the build's configuration changed a config, and takes the
  verb for that.
- **`Purge` and `Kick out` split on what survives.** `Purge` is for a file or a folder that is
  gone whole — `Purge tests/legacy/OldValidator.js`, `Purge tests/legacy/`. `Kick out` takes the
  shape `Kick out <what> from <where>`, because the point is that `<where>` is still there
  without `<what>` — a member of a class, a section of a document, an entry of a manifest:
  `Kick out main: from package.json`. The pair mirrors `Declare` and `Define`.
- **`Tidy up` is the cleanup that is not a removal.** It takes the shape `Tidy up <where>` —
  `Tidy up the environment files` — and covers stale naming, leftover duplication and disorder,
  where naming each micro-change on its own would be noise. When the cleanup *is* a removal, the
  verb is `Purge` or `Kick out`, however the branch that carries it happens to be named.
  - **The test is that no responsibility moves.** A typo in a comment or a type, a formatting
    correction, a blank line added or taken out, the parameter of every public function renamed
    to `it` — the tree does the same thing before and after, and a reviewer confirms exactly
    that. One identifier renamed on its own merits is `Rename`; a sweep that makes a whole
    surface uniform is a tidy-up.
  - **A type that was wrong over code that was right is a tidy-up, never a `Fix`.** The
    implementation is what the tree runs, and it was doing its job; only the annotation beside
    it was mistaken. `Fix` would say the behavior had been wrong, and a later reader could not
    then tell that subject apart from one that repaired a real defect.
  - **Reordering has its own verb.** Putting a set of peers into a different order is
    `Rearrange`, not a tidy-up: `Rearrange imports for vue/core alphabetically`.
- **`Rearrange` moves peers, and only peers.** Imports, declarations, cases, entries — a set
  whose members stand at the same level, where the order is a matter of reading and nothing
  else. Reordering control flow is not this: two `if` statements swapped is a behavior change
  wearing the clothes of an ordering, and it takes the verb its behavior deserves. Relocating
  something to another file is `Move`; `Rearrange` never crosses a file.
- **`Update` and `Retake` split on what was there before.** `Update` carries a sound
  implementation forward and leaves it giving something it did not give before — the gain is the
  point. `Retake` replaces what was poor, hurried or a stopgap with what should have been there,
  and claims no gain beyond that. Neither is `Fix`: that one is for behavior that was wrong,
  where `Retake` is for an implementation that worked and was not good enough.
- **A member's inputs and outputs are `Update`'s ground.** A parameter it now takes, a return
  value it now gives — the member itself is what changed. That a parameter is something which
  appeared does not make it `Add`: addition takes `Add` where what appeared stands as an item of
  its own, such as a test case or a manifest entry. Whether the change breaks a caller is marked
  by the format rather than the verb — Conventional Commits writes `!` with a `BREAKING CHANGE:`
  trailer.
- **`Fulfill` fills a gap that was already there; `Add` brings something that was not.** A test
  left as a TODO, a JSDoc block without its `@returns`, an option a rule was never given, a
  `package.json` field left blank — the place existed and was short, and the commit makes it
  whole. Where what is covered is itself new, its tests arrive with it and that is `Add`.
  Nothing is replaced either way, which is what separates `Fulfill` from `Update`, and nothing
  was wrong, which separates it from `Fix`. An option that was set and then moved is `Adjust`;
  an option that was never set is `Fulfill`.
- **`Add` covers a whole new thing and a part added to one that stands.** A test file that did
  not exist and one more case inside a file that did are both `Add`, so long as what they cover
  is itself new — coverage that was owed is `Fulfill`. Addition is deliberately left unsplit: no
  pair divides it the way `Purge` and `Kick out` divide deletion, because none is needed — the
  subject names the thing added either way, and nothing a later reader wants is hidden by the
  choice. Do not invent a verb for the partial case.

  Nor reach for `Update` there. An item added inside something else is still an addition: `Add`
  names what appeared, where `Update` names only the thing it appeared in. The more telling verb
  wins, as `Declare` and `Define` win over `Add` where they apply.
- **A word that needs several of these verbs to say what it covered is too vague for a row.**
  That is the test every exclusion below comes from: name the operations the word spans, and if
  the answer is more than one, a reader of the subject cannot tell which happened. It is not
  about how the word feels — `Refine` and `Apply` feel precise and span four verbs each, where
  `Purge` feels blunt and spans exactly one.
- **`Change`, `Remove` and `Create` are the ambiguous trio, and the table carries none of
  them.** Each is broad enough to cover whatever happened, so the subject narrows nothing, and
  each has specific verbs here to be narrowed into.
  - `Change` fits wherever any of the others would. A subject opening with it has said only
    that the tree is not what it was.
  - `Remove` reads the same whether a whole file went or one line inside it did — `Purge` and
    `Kick out` split exactly that, and `Delete` is `Remove`'s twin in this.
  - `Create` leaves open whether a class, one of its members, or something else arrived, which
    `Declare`, `Define` and `Add` settle between them. `Make` is `Create`'s twin, and
    `Make changes to syntax` is what it comes to.
- **The table carries no `Refine` either.** It claims the thing got better without saying what
  changed, so the reader is left with the writer's satisfaction and nothing else. Whatever the
  improvement was, a listed verb names it: the wording redone is `Retake`, the formatting
  straightened is `Tidy up`, the order put right is `Rearrange`, the thing itself carried
  forward is `Update`.
- **The table carries no `Migrate`.** Carrying content in from elsewhere is bracketed by the
  branch, not repeated on every commit: the marker names the origin once — `Start migrating the
  mail templates from lunas-ec-cart-backend` — and each commit inside then says what kind of
  thing arrived, `Declare` or `Define` or `Add`. A subject reading `Migrate BaseInputValidator`
  says less than `Declare BaseInputValidator` does, and the origin it gestures at is nowhere.
- **The subject names the thing that changed, not the thing that was brought to it.** This is
  why the table carries no `Apply`. `Apply flex layout to main-container` puts the layout in the
  object and leaves `main-container` — the thing that is now different — in a trailing phrase.
  And the word spans four verbs: enabling a module is `Install` or `Turn on`, switching to
  another mechanism is `Use`, evening out the spacing is `Tidy up`, and
  `Apply respective change to the branch-number usage` is the umbrella noun that
  [granularity.md](./references/granularity.md) already turns away.
- **The table carries no `Replace`.** It spans `Use`, where one mechanism gave way to another,
  and `Retake` or `Update`, where a passage of prose did. It also invites a subject that names
  only what was dropped — `Replace SCSS variables` says nothing about what stands there now,
  where `Use CSS custom properties instead of SCSS variables` says both and puts the surviving
  mechanism first.
- **The table carries no `Implement` or `Enhance`.** Both name a whole feature's worth of work,
  which is the scale of a Conventional Commits type and of a branch, not of one commit. A
  repository on that format writes `feat`, and a trunk-scale branch is named `implement/xxx`;
  the commits inside are finer than either, and each takes the verb for what it did.
- **A verb that names the purpose is not a verb that names the change.** `Keep`, `Tell` and
  `Point` read as precise and each spans several operations, so a reader is left with the
  writer's aim and no way to know what was done.
  - `Keep` covers a requirement the code was short of (`Fulfill`), a rule that had not existed
    before (`Prevent`), and a passage newly written (`Add`) — one verb over three commits a
    reader needs to tell apart.
  - `Tell` covers the ground of `Fulfill`, where the reader was short of something, and of
    `Add`, where the statement is new.
  - `Point` covers `Retake` where a passage was replaced, `Update` where it gained, and `Use`
    where a reference in code moved to another mechanism.
  - `Optimize` and `Retake` name a motive as well, and keep their rows because the motive
    settles which operation it was rather than standing in for it.
- **`Extract` leaves the logic in the tree, in a home of its own.** The call site stays and
  delegates to what was pulled out, which is what separates it from `Purge` and `Kick out` —
  nothing was deleted. Relocating something intact is `Move`; `Extract` makes a new home out of
  part of an existing one, and `Unify` is the same operation run backwards. `Cut out` is not
  in the table for the reason `Remove` and `Delete` are not: it reads as excision, so the
  subject leaves open whether the logic landed somewhere or went away.
- **`Install` and `Uninstall` name the dependency, not the file that records it.** `Install
  date-fns 4.1.0` and `Uninstall date-fns` say what the project now depends on, or no longer
  does. Reaching instead for `Add`, or for `Kick out`, would describe an edit to `package.json`,
  which is merely where the fact is written down. A version change to a dependency already
  installed is `Update`.
- **A subject describes a transition, never a state.** `Don't disable the action button when
  the competition is completed` describes a state the code should hold; `Allow the action button
  when the competition is completed` says the same change as a transition, and it completes
  *"Applying this commit will …"*, which a negative cannot. So a restriction lifted is `Allow`,
  never a negative.
  - **This is why `Put` is not in the table.** `Put the settings at the bottom` names where
    things sit once the commit is applied, not what the commit did. What it did was
    `Rearrange`, or `Move`, or `Add`, and one of those three always fits.
- **`Turn on` and `Turn off` name the switch, not what it permits.** `Turn off
  jsdoc/require-jsdoc for tests in eslint.config.js` says which setting moved; `Allow` and
  `Prevent` name the behavior that is now open or closed. Where both would be true, the switch
  is the more telling of the two, because a reader can go and find it. A value that moved along
  a scale is neither: that one is `Adjust`.
  - **`Enable` and `Disable` are not in the table**, and for a different reason than the vague
    words: each spans exactly one verb, which is `Turn on` and `Turn off`. A synonym of a
    listed verb costs nothing to write and costs a reader twice — the history splits between
    two spellings of one event, and whoever searches it for when a rule went off has to know
    both.
- **A setting changes in three degrees, and each has its own verb.** `Turn on` and `Turn off`
  flip the switch. `Adjust` moves the dial while the switch stays where it is — `Adjust MaxFiles
  to 20 in <config>`, `Adjust max-len to 120 in eslint.config.js`. `Update` is for the setting
  that changed in itself rather than in degree — `Update jest version to 30.4.2 in
  package.json`. `Adjust` carries no direction: a dial is as properly turned down as up, which
  is why `Update`'s gain does not apply to it.
- **`Start`, `Release` and `Merge` are reserved** for the commits that carry no change of their
  own — the two branch-opening markers and the merge commit, all described by the git branch
  convention. A change that
  folds two things into one takes `Unify`, never `Merge`, so that a merge commit stays
  recognizable by its subject alone.

### Referring to class members

When a subject or body names a class member, use the project's documentation notation.
Instance members are prefixed with `#`, static members with `.`.

| notation | member |
| :-- | :-- |
| `#instanceProperty` | instance property |
| `#instanceMethod()` | instance method |
| `#get:instanceGetter` | instance getter |
| `#set:instanceSetter` | instance setter |
| `.staticProperty` | static property |
| `.staticMethod()` | static method |
| `.get:staticGetter` | static getter |
| `.set:staticSetter` | static setter |

With the class name attached, write it as `SampleClass#extractValue()` or
`SampleClass.createValue()`.

```
Update BaseRestfulApiLauncher#extendRequestHooks() to return fulfilled hooks
Tidy up the JSDoc of BaseRestfulApiLauncher.get:ResponseBodyParser
```

This is the same notation used throughout documentation and error messages; see the
documentation convention.

### Referring to a skill

A skill named in a subject or body carries the **leading slash it is invoked with**.

```
Author /hoc-git-branch skill
Fulfill the article rule for a branch name in /hoc-git-branch
Add the notation for a skill name in /hoc-git-commit
```

The slash is what marks the word as a skill. Without it the same string reads as a directory,
a package, or a branch prefix — all of which a repository of skills is full of — and a reader
scanning subjects for where a convention changed has to work out which one each mention was.

### Body

- The body is **optional**. Omit it when the subject already says everything — most small,
  well-scoped commits need no body.
- When present, separate it from the subject with **one blank line** and wrap it at
  **72 characters**.
- The body explains **why**, not what. The diff already shows what changed; what it cannot
  show is the constraint, the rejected alternative, or the non-obvious consequence.
- A ticket or issue identifier may appear in the body, but the body must still explain the
  change **without** it. A reader who cannot open the ticket must still be given the
  reason.

### Trailers

- Do not add attribution trailers (`Co-authored-by:`, `Generated-with:`, and similar) unless
  the project explicitly asks for them.
- **A change that forces callers to follow takes a `BREAKING CHANGE:` trailer**, and there the
  body stops being optional: name what a caller relied on, and what it has to do instead. The
  test is whether code that passed against the old surface still passes — a parameter added
  without a default, a return value narrowed, a member gone. A surface widened compatibly is
  not this.
  - **The trailer carries it in either format.** A subject has room for what changed, not for
    what it costs, and breakage is found with `git log --grep='BREAKING CHANGE'` rather than by
    scanning subjects. Conventional Commits marks the subject as well, with `!` before the
    colon; the imperative format adds no mark, and needs none.

### Each commit stands alone

- Order commits so that **no commit depends on a later one**. A reader checking out any single
  commit should find a coherent tree.
- **What must pass before a commit is not settled here.** Which commands run before a commit,
  and which before the work is called complete, belongs to the workflows convention. This one
  settles what goes into a commit, how it is worded, and the order the commits land in — never
  whether a command's result permits the commit.

## Granularity in one line

One commit is **one decision a reviewer can accept or reject on its own**. If the subject needs
the word "and" to join two of them, the commit is two commits.

The full heuristic — what to split, what to keep together, and how to stage a mixed working
tree — is in [granularity.md](./references/granularity.md).

## Detail files

- [granularity.md](./references/granularity.md) — what belongs in one commit, splitting a mixed working tree
- [format-imperative.md](./references/format-imperative.md) — capitalized imperative subject, no type prefix
- [format-conventional.md](./references/format-conventional.md) — Conventional Commits (`type: summary`)
