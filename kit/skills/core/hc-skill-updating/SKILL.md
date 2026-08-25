---
name: hc-skill-updating
description: "Conventions for creating new skills (SKILL.md) or updating existing ones. Defines how to name a skill (the domain prefix, and the folder name that must equal `name:`), where the folder goes among the three domain directories, and the conventions to follow when writing the body and the description."
---

# Skill Updating

This gathers the conventions for creating new skills (`SKILL.md`) or updating existing ones.

- When writing or updating `SKILL.md`, follow the documentation convention.

## Naming a skill

`name:` is the skill's official command name — what a user types as `/name` — and it is also the
skill's own folder name and the folder name it installs as. **One string, three places.** Choose it
for the reader who sees a flat list of skills and never sees this repository.

- **The name begins with its domain's prefix**: `hc-` for a skill under `core/`, `hb-` under
  `backend/`, `hf-` under `frontend/`. The prefix is what tells a reader of that flat list which
  skills came from this library, and which domain each belongs to.
- **`name:` and the folder name must be the same string.** They are not two facts to keep in sync
  by habit: the build that produces the installable output and the repository's skill-name audit
  both refuse to proceed on a mismatch.
- **The name is 4–64 characters of `[a-z0-9-]`**, of which the prefix takes 3. A single case only.
- **Uniqueness needs no thought.** One directory cannot hold two folders of one name, and the three
  prefixes cannot overlap, so no two skills in the library can collide.
- **Renaming is a breaking change** for repositories that already invoke the skill, so choose
  deliberately the first time. Moving a skill to another domain is a rename too — the prefix
  changes with it.

After the prefix, name the subject, not the source tree:

- **Leave out words that only place the skill within its domain.** A backend skill does not repeat
  the framework it is for, and a `core` skill does not carry a classifying word like
  `declarations`, `shared` or `members` — `hc-async`, `hc-accessors`, `hb-agent-loop`.
- **Keep a classifying word when it is what a reader would search for**: `hc-classes-constructor`,
  `hc-modules-exports`, `hf-css-units`.
- **Prefer a short marker over a spelled-out grouping word**: `hf-cp-table` for a component,
  `hf-css-props-naming` for CSS custom properties.
- **One word after the prefix is enough when it names the concept unambiguously** (`hc-async`,
  `hc-jest`). Qualify it when sibling skills have an equal claim to the same word: CSS prohibitions
  are `hf-css-prohibits` and `hf-css-props-prohibits`, not two skills both called prohibits.

## Writing and updating the description

- `description:` should indicate the skill's scope (which domain, and what it defines), not name an individual rule.
- When a skill is updated (rules added or removed), update `description:` to reflect what was added or removed. Do not let `description:` go stale by changing only the body.

### Keep the description under 500 characters

- **Every description is loaded into every conversation**, whether the skill is used or not, because the agent needs them to route. The cost is paid up front by all users of the library, so a description is not free space.
- **Never summarize the procedure in the description.** A description complete enough to act on gets acted on *instead of* the skill: it is already in context, and opening `SKILL.md` is an extra step the agent can skip. A skill that says "run three passes" in its body but "reviews correctness and conventions" in its description will get one pass.
- Do not enumerate the body's sections, steps, checks, or file layout. That is a table of contents, and it belongs in the body.
- Write only what decides routing: **scope** (domain, and what kind of artifact), **triggers** (the words and situations that should reach this skill), and the **boundary** (what this skill is not, and which convention owns that instead). What discriminates between sibling skills is the boundary, never the list of internal checks — siblings share those.
- 500 characters is the ceiling, not the target. Most skills route correctly in 250–400. Length is a proxy: if the description is long, it is almost always because it started summarizing the procedure.
- Refer to a neighbouring convention descriptively ("input validation belongs to the resolver input-validator convention"), never by path.
- Quote the value (`description: "..."`) or use a block scalar (`description: |-`). A bare scalar containing `: ` is not valid YAML and the frontmatter will fail to parse.

## Directory structure

- A single skill is one directory containing `SKILL.md`.
- Supplementary reference documents go under `references/` within that skill's directory, and any
  executable helpers under `scripts/`.
- A skill directory holds no `SKILL.md` below its own top level. Its subdirectories are its own
  files, never more skills.
- There are exactly three domain directories, and a skill folder sits **directly** inside one of
  them — one level, with no grouping directories in between:

```
lib/skills/
├── core/      hc-*   Common/foundational (conventions applied across backend/frontend)
├── backend/   hb-*   Backend-specific
└── frontend/  hf-*   Frontend-specific
```

So every skill in the library is at `lib/skills/<domain>/<name>/SKILL.md`, and nowhere else. There
is no `backend/<framework>/`, no `frontend/<library>/components/`: what a nested directory would
have said belongs in the name instead, where the reader of an installed skill can see it.

## Placement of skills

- Common conventions that do not depend on a specific domain go under `core/`.
- Conventions specific to a particular domain go under that domain's directory.
- The prefix must match the directory. Deciding the domain and deciding the first three characters
  of the name are the same decision.

## Keep skills self-contained

- Write a skill so that it is meaningfully complete on its own. **Information that is not self-contained within the skill must not be put into `SKILL.md`.**
- Specifically, do not cite links to external websites (URLs to internal documents, tickets, articles, etc.) in `SKILL.md` as the basis or reference for a convention. If the reader cannot access the link destination, the meaning of the convention is lost.
- Information that exists only at the link destination (tables, concrete examples, rationale, etc.) should not be replaced by a link; instead, transcribe the necessary content into `SKILL.md` (or into `references/` within the same skill) so it is self-contained.
- Cross-skill relative path references are prohibited. A link that traverses to another skill via `../` describes this repository's source tree, which the consuming repository does not have: installed skills are siblings under `.claude/skills/`, so the path is wrong the moment it ships.
- Referring to another skill **by its name** is fine, and is the clearer choice when the reader has to go there ("see `hc-jsdoc`"). The name is stable — it changes only by a deliberate rename, which is a breaking change decided on purpose, never as a side effect of reorganizing the source. Where the reader only needs to know that a convention exists elsewhere, name the concept instead ("see the documentation convention").
- Relative references to files within the same skill folder (e.g., `./references/structure.md`) are permitted, since they move together with the skill and their relative relationship is preserved.
