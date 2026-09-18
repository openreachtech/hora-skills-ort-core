---
name: hoc-skill-updating
description: "Conventions for creating new skills (SKILL.md) or updating existing ones. Defines how to name a skill (the prefix its library owns, the folder name that must equal `name:`, and the practice the rest of the name has to state), which library repository a skill belongs to, where one skill stops and a neighbouring convention takes over, the flat layout every library keeps under `kit/skills/`, and the conventions to follow when writing the body and the description."
---

# Skill Updating

This gathers the conventions for creating new skills (`SKILL.md`) or updating existing ones.

- When writing or updating `SKILL.md`, follow the documentation convention.

## Naming a skill

`name:` is the skill's official command name — what a user types as `/name` — and it is also the
skill's own folder name and the folder name it installs as. **One string, three places.** Choose it
for the reader who sees a flat list of skills and never sees this repository.

- **The name begins with its library's prefix.** Each library repository owns one prefix, and the
  prefix is what tells a reader of that flat list which library a skill came from. Deciding which
  library a skill belongs to and deciding the first characters of its name are one decision, taken
  once — see the placement section below.
- **`name:` and the folder name must be the same string.** They are not two facts to keep in sync
  by habit: the build that produces the installable output and the repository's skill-name audit
  both refuse to proceed on a mismatch.
- **The name is 4–64 characters of `[a-z0-9-]`**, of which the prefix takes the first three or
  four. A single case only.
- **Uniqueness needs no thought.** One directory cannot hold two folders of one name, and no two
  libraries share a prefix, so no two skills across the libraries can collide.
- **Renaming is a breaking change** for repositories that already invoke the skill, so choose
  deliberately the first time. Moving a skill to another library is a rename too — the prefix
  changes with it.

After the prefix, name the subject, not the source tree:

- **Leave out words that only repeat what the prefix already says.** A backend skill does not
  restate the framework it is for, and a foundational skill does not carry a classifying word like
  `declarations`, `shared` or `members` — `hoc-async`, `hoc-accessors`, `hor-agent-loop`.
- **Keep a classifying word when it is what a reader would search for**: `hoc-classes-constructor`,
  `hoc-modules-exports`, `hof-css-units`.
- **Prefer a short marker over a spelled-out grouping word**: `hof-cp-table` for a component,
  `hof-css-props-naming` for CSS custom properties.
- **One word after the prefix is enough when it names the concept unambiguously** (`hoc-async`,
  `hoc-jest`). Qualify it when sibling skills have an equal claim to the same word: CSS prohibitions
  are `hof-css-prohibits` and `hof-css-props-prohibits`, not two skills both called prohibits.
- **A word another convention already leads with is not freed by qualifying it.** Qualification
  settles siblings that hold a word equally; where one convention has built its own rule on the
  word, a second skill carrying it leaves the word pointing at two places and naming neither. A
  convention for moving dependency versions took `raise` rather than `bump` on this ground —
  `bump` heads the publishing convention's rule about a release's own version.

### The name is read as "how to do X"

**A reader expands the name into an instruction.** `hoc-git-commit` reads as how to commit,
`hoc-npm-publish` as how to publish, and every skill in the libraries is read the same way. So
**the subject after the prefix has to be a practice**, and the check on a candidate is
mechanical: expand it and read the result.

**The expansion reads the words in the order they are written.** Two shapes manage it: a verb
with its object behind it, and an established compound noun — `hoc-code-review` expands to how
to do a code review without anything being moved. What fails is a name that is neither, whose
words have to be swapped before the instruction appears. `hoc-npm-deps-raise` was defended as
*how to raise deps*, which is the name read backwards; `hoc-npm-raise-deps` is the same three
words in the order that reads. **The swap is easy to perform without noticing**, because whoever
supplies it already knows what the skill does, and a reader meeting the name in a flat list does
not.

**Naming a skill after the outcome it exists to prevent inverts it.** A skill written to keep a
refactor from moving a published interface was proposed as `hoc-breaking-change` — which expands
to *how to make a breaking change*, instructions for producing the very thing. The argument for
it was that a person's actual question is "is this a breaking change?", and that a name matching
the question routes best. It does not survive the expansion: the question a reader arrives with
and the instruction they leave with are not the same sentence.

- **This does not reach a skill named for a body of rules.** `hof-css-prohibits` expands to how
  to handle the CSS prohibitions, and observing them is the practice. What fails the test is a
  name whose subject is a result nobody wants, not one whose subject is a set of rules about
  results.

**A name must not promise an extent the convention does not reach.** A skill for bringing a
project's declared dependency versions up was proposed as `hoc-npm-deps-latest`, while the
convention it carried takes each entry only as far as its own declaration already permits and
leaves anything past that for a decision of its own. A pass that finishes under it is therefore
not at the latest of everything: the name promised the one thing the body spends a section
refusing.

- The fault is not the word but its grammar. **The word states a result where the body states a
  bound**, so check a candidate against the rule that limits the practice, never against the
  rule that describes it — the describing rule is what suggested the name in the first place.
- Where the name has to carry a direction, take a verb that gives one without naming a
  destination. `raise` says which way and stops there.

**Where the library already has a word for the thing, the name takes that word.** The skill above
became `hoc-retake-declaration`, from the verb the commit convention defines: a `Retake` replaces
something poor or hurried with what should have been written and *claims no gain beyond that*,
where an `Update` carries an implementation forward and the gain is the point.

That name does work a descriptive one cannot. **It decides membership.** Asked whether a defect
found mid-work may be fixed, the name answers on its own — fixing it is a gain, a gain is an
`Update`, and this is a `Retake`. A name assembled out of fresh words has to be read alongside
the body before it settles anything, and a rule the body forgot to state is then simply absent.

- The cost is that the reader must know the word. That is paid once, by the convention that
  defines it, and it is the same cost the vocabulary was already worth paying — **a term the
  library defines and never names a skill after is a term nobody meets.**

## Writing and updating the description

- `description:` should indicate the skill's scope (what subject it covers, and what it defines), not name an individual rule.
- When a skill is updated (rules added or removed), update `description:` to reflect what was added or removed. Do not let `description:` go stale by changing only the body.

### Keep the description under 500 characters

- **Every description is loaded into every conversation**, whether the skill is used or not, because the agent needs them to route. The cost is paid up front by all users of the library, so a description is not free space.
- **Never summarize the procedure in the description.** A description complete enough to act on gets acted on *instead of* the skill: it is already in context, and opening `SKILL.md` is an extra step the agent can skip. A skill that says "run three passes" in its body but "reviews correctness and conventions" in its description will get one pass.
- Do not enumerate the body's sections, steps, checks, or file layout. That is a table of contents, and it belongs in the body.
- Write only what decides routing: **scope** (the subject, and what kind of artifact), **triggers** (the words and situations that should reach this skill), and the **boundary** (what this skill is not, and which convention owns that instead). What discriminates between sibling skills is the boundary, never the list of internal checks — siblings share those.
- 500 characters is the ceiling, not the target. Most skills route correctly in 250–400. Length is a proxy: if the description is long, it is almost always because it started summarizing the procedure.
- Refer to a neighbouring convention descriptively ("input validation belongs to the resolver input-validator convention"), never by path.
- Quote the value (`description: "..."`) or use a block scalar (`description: |-`). A bare scalar containing `: ` is not valid YAML and the frontmatter will fail to parse.

## Directory structure

- A single skill is one directory containing `SKILL.md`.
- Supplementary reference documents go under `references/` within that skill's directory, and any
  executable helpers under `scripts/`.
- A skill directory holds no `SKILL.md` below its own top level. Its subdirectories are its own
  files, never more skills.
- **`kit/skills/` is flat.** A skill folder sits directly inside it, and there is no level in
  between:

```
kit/skills/
├── <name>/
│   ├── SKILL.md
│   ├── references/
│   └── scripts/
└── <another-name>/
    └── SKILL.md
```

So every skill is at `kit/skills/<name>/SKILL.md`, and nowhere else. There is no grouping
directory for a framework, a component family or a category: what such a directory would have
said belongs in the name instead, where the reader of an installed skill can see it. **An
installed skill has no directory above it either** — skills arrive as siblings under
`.claude/skills/`, so a layer the source tree keeps is a layer that does not survive shipping.

## Placement of skills

**A skill belongs to one library repository, and the library is what its prefix names.** There is
no directory to choose: the repository is the choice, and the flat `kit/skills/` below it holds
whatever that repository carries.

- Conventions that hold whatever is being built go to the foundational library.
- Conventions that only make sense within one stack, platform or product go to that library.
- Conventions about how work is carried out and handed over — writing a document, an issue, a
  manual — go to the support library.
- **The prefix must match the library.** Deciding which library a skill belongs to and deciding
  the first characters of its name are the same decision, and getting it wrong later costs a
  rename.

**A subject that spans two libraries is two skills, not one.** One prefix cannot name two
libraries, so there is no name a single skill could take. Split it where the libraries divide,
and let each half name the other by name where a reader has to go there.

## Where a skill stops

A `description:` that names its boundary has said where the skill stops. **It has not said when
the reader gets there**, and that missing sentence is what lets the neighbour's material back in.

**Naming a boundary does not discharge it.** A convention that routed advisories next door in its
opening paragraph still carried the neighbouring mechanism through its body — the reach of an
override, the test for whether a consumer survives one — in wording that was nearly the
neighbour's own. The boundary was written and the body walked past it, because nothing in the
body said at which point the reader leaves.

**A rule whose reason lives in another convention is that convention's rule.** The test is not
whether the rule is true, nor whether readers of this skill will meet the situation. It is
whether *this* convention supplies the reason the rule exists. The skill above had a row sending
a dependency the project never asked for to an override, and said nowhere why such a dependency
would move at all; the only reason the library gives for moving one is an advisory, which belongs
next door. **A rule supported from outside is a rule that belongs outside.**

- This is what a duplicate looks like before it becomes one. Two skills stating a rule in nearly
  the same words is the late symptom. The early symptom is a rule sitting in a skill that cannot
  say why it is there.

**Specify the handoff by what this convention could not reach.** A boundary written as a subject
— *advisories belong next door* — leaves the reader to judge when a subject has arrived. A
boundary written as a residue does not: run the convention to its end, and whatever is left is
the neighbour's by construction. The dependency convention hands on what the audit still reports
**after** its pass, which is exactly what raising a declared version could not reach, and that is
also the case the neighbouring convention opens with. The two halves meet without either
restating the other.

- The residue is usually the neighbour's own opening case. Where it is not, the boundary is in
  the wrong place.
- **A convention that mostly clears what the neighbour handles still needs the sentence.** Stating
  what the practice *adds* gets the direction backwards, and sends the reader next door for the
  cases the practice was about to settle itself.

## Keep skills self-contained

- Write a skill so that it is meaningfully complete on its own. **Information that is not self-contained within the skill must not be put into `SKILL.md`.**
- Specifically, do not cite links to external websites (URLs to internal documents, tickets, articles, etc.) in `SKILL.md` as the basis or reference for a convention. If the reader cannot access the link destination, the meaning of the convention is lost.
- Information that exists only at the link destination (tables, concrete examples, rationale, etc.) should not be replaced by a link; instead, transcribe the necessary content into `SKILL.md` (or into `references/` within the same skill) so it is self-contained.
- Cross-skill relative path references are prohibited. A link that traverses to another skill via `../` describes this repository's source tree, which the consuming repository does not have: installed skills are siblings under `.claude/skills/`, so the path is wrong the moment it ships.
- Referring to another skill **by its name** is fine, and is the clearer choice when the reader has to go there ("see `hoc-jsdoc`"). The name is stable — it changes only by a deliberate rename, which is a breaking change decided on purpose, never as a side effect of reorganizing the source. Where the reader only needs to know that a convention exists elsewhere, name the concept instead ("see the documentation convention").
- Relative references to files within the same skill folder (e.g., `./references/structure.md`) are permitted, since they move together with the skill and their relative relationship is preserved.
