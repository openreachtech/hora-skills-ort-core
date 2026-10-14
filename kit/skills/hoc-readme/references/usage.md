# Usage Section

Rules concerning the README's Usage section, and the placement of files split out from the README.

## Placement of split-out files (common to all splits)

Files split out from the README (Usage / Features / API, etc.) all go under `docs/`. Since split-out files are not
bundled into the npm package, they need not live in a `readme/` directory next to the README; they are unified under
`docs/`, the standard place for documentation.

- **Separate by language directory**: the default language (English) goes under `docs/en/`, other languages under `docs/<xx>/` (`xx` is the language code; currently only `docs/ja/` exists). This keeps a given language's documentation together in one folder, making it easy to copy or move per language.
- **Place categories directly under the language directory**: `docs/<lang>/<category>/` (e.g., `docs/en/usage/`, `docs/en/features/`, `docs/en/api/`).
- **Keep the existing language suffix at the end of the filename**: the default language uses `.md`, and language `xx` uses `.xx.md`. So English is `docs/en/usage/usage.md` and Japanese is `docs/ja/usage/usage.ja.md`.
- **`docs/` is not bundled into the npm package by default** (`files` in `package.json` lists the executable code such as `lib/`, and nothing else). Documentation is meant to be read on GitHub / npmjs, so the split-out files need not travel with an install.
  - **Whether a repository ships `docs/` all the same is that repository's own decision**, taken once and written into its `files`. A package whose documentation is part of what a consumer installs has a reason the default does not cover. The default is what holds where the repository has decided nothing.
- **Links from the README body to split-out files should use absolute GitHub URLs, not relative paths** (e.g., `https://github.com/openreachtech/<repository>/blob/main/docs/en/usage/usage.md`). Relative links break inside an installed package that doesn't bundle them, whereas absolute URLs work from GitHub, npmjs, and `node_modules` alike. The link destination should match the README's language (`README.md` → `docs/en/<category>/...`, `README.xx.md` → `docs/<xx>/<category>/...`). A package distributed privately links relatively instead; see `## A package distributed privately` below.

## Splitting Usage

- Once Usage grows past a certain length, split it out into `docs/en/usage/usage.md` (and, per language, `docs/<xx>/usage/usage.xx.md`), and turn the README body into a link reference.
- When there are multiple independent features, use a Features section instead of a single Usage, splitting each feature out into `docs/en/features/<feature>.md` (and, per language, `docs/<xx>/features/<feature>.xx.md`) and linking to it (see Features in `sections.md`).
- For splitting the API, see `api-references.md` (multiple classes are split into `docs/<lang>/api/<ClassName>...` and bundled together via `docs/<lang>/api/index...`).

## A package distributed privately

A package published privately — readable by the members of an npm organization alone, from a repository that is private as well — is read where it is installed and nowhere else. Its readers cannot open the repository, so what the rules above arrange for a public package does not reach them.

- **The documents the reader needs travel in the tarball.** `files` in `package.json` lists `docs/` beside the executable code, so the README and everything it links to are read in `node_modules/` alone. This is the decision the default above leaves to the repository, taken the one way a private package can take it.
- **Links between the shipped documents are relative, and nothing links to GitHub.** An absolute GitHub URL opens a repository the reader cannot enter. A relative link reaches the file beside it in the installed package, and in the repository for whoever maintains it, so every link reaches a file the tarball carries.
