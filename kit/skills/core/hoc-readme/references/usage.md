# Usage Section

Rules concerning the README's Usage section, and the placement of files split out from the README.

## Placement of split-out files (common to all splits)

Files split out from the README (Usage / Features / API, etc.) all go under `docs/`. Since split-out files are not
bundled into the npm package, they need not live in a `readme/` directory next to the README; they are unified under
`docs/`, the standard place for documentation.

- **Separate by language directory**: the default language (English) goes under `docs/en/`, other languages under `docs/<xx>/` (`xx` is the language code; currently only `docs/ja/` exists). This keeps a given language's documentation together in one folder, making it easy to copy or move per language.
- **Place categories directly under the language directory**: `docs/<lang>/<category>/` (e.g., `docs/en/usage/`, `docs/en/features/`, `docs/en/api/`).
- **Keep the existing language suffix at the end of the filename**: the default language uses `.md`, and language `xx` uses `.xx.md`. So English is `docs/en/usage/usage.md` and Japanese is `docs/ja/usage/usage.ja.md`.
- **Do not bundle into the npm package** (`files` in `package.json` should only include executable code such as `lib/`, and must not include `"docs/"`). Documentation is meant to be read on GitHub / npmjs.
- **Links from the README body to split-out files should use absolute GitHub URLs, not relative paths** (e.g., `https://github.com/openreachtech/<repository>/blob/main/docs/en/usage/usage.md`). Relative links break inside an installed package that doesn't bundle them, whereas absolute URLs work from GitHub, npmjs, and `node_modules` alike. The link destination should match the README's language (`README.md` → `docs/en/<category>/...`, `README.xx.md` → `docs/<xx>/<category>/...`).

## Splitting Usage

- Once Usage grows past a certain length, split it out into `docs/en/usage/usage.md` (and, per language, `docs/<xx>/usage/usage.xx.md`), and turn the README body into a link reference.
- When there are multiple independent features, use a Features section instead of a single Usage, splitting each feature out into `docs/en/features/<feature>.md` (and, per language, `docs/<xx>/features/<feature>.xx.md`) and linking to it (see Features in `sections.md`).
- For splitting the API, see `api-references.md` (multiple classes are split into `docs/<lang>/api/<ClassName>...` and bundled together via `docs/<lang>/api/index...`).
