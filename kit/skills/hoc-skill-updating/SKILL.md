---
name: hoc-skill-updating
description: "Conventions for creating new skills (SKILL.md) or updating existing ones. Defines how to name a skill (the domain prefix, and the folder name that must equal `name:`), where the folder goes among the three domain directories, and the conventions to follow when writing the body and the description."
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

## Keep skills self-contained

- Write a skill so that it is meaningfully complete on its own. **Information that is not self-contained within the skill must not be put into `SKILL.md`.**
- Specifically, do not cite links to external websites (URLs to internal documents, tickets, articles, etc.) in `SKILL.md` as the basis or reference for a convention. If the reader cannot access the link destination, the meaning of the convention is lost.
- Information that exists only at the link destination (tables, concrete examples, rationale, etc.) should not be replaced by a link; instead, transcribe the necessary content into `SKILL.md` (or into `references/` within the same skill) so it is self-contained.
- Cross-skill relative path references are prohibited. A link that traverses to another skill via `../` describes this repository's source tree, which the consuming repository does not have: installed skills are siblings under `.claude/skills/`, so the path is wrong the moment it ships.
- Referring to another skill **by its name** is fine, and is the clearer choice when the reader has to go there ("see `hoc-jsdoc`"). The name is stable — it changes only by a deliberate rename, which is a breaking change decided on purpose, never as a side effect of reorganizing the source. Where the reader only needs to know that a convention exists elsewhere, name the concept instead ("see the documentation convention").
- Relative references to files within the same skill folder (e.g., `./references/structure.md`) are permitted, since they move together with the skill and their relative relationship is preserved.
