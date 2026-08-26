# API References

Rules concerning how the README's API reference is written.

- If an instance member is annotated with `@public`, the API section should describe only public members carrying `@public`. Instance members without `@public` (e.g., `get Ctor`) must not be listed in the API section.
- Static members are listed in the API section even without `@public`, if they are factory methods that create an instance of the class body, since these are part of the external interface. Other static members intended for internal use are not listed.
- Since the README is not implementation detail, descriptions of private members are omitted. Even when an explanation of internals is needed, do not mention private member names — describe them at a conceptual level.
- For members without `@public` where it is unclear whether they are called from outside, do not decide on your own —
  confirm with the user whether they should be listed in the API section.
- When there are multiple classes implemented by the application (e.g., base classes meant to be extended):
  - Split the API for each class out into `docs/en/api/<ClassName>.md` (e.g., `docs/en/api/AlphaClass.md`; per language, `docs/<xx>/api/<ClassName>.xx.md`).
  - Do not link directly from the README to each class file; bundle them together via `docs/en/api/index.md` (per language, `docs/<xx>/api/index.xx.md`). The README body (`## API`) links to this index, and the index holds the list of links to each class file.
  - If splitting out, place the "Notation of Class Members" table at the top of each language's `docs/<lang>/api/index...` (if not splitting out, place it at the top of the README's `## API`).
  - The link from the README (`## API`) to the index should use an absolute GitHub URL (`README.md` → `docs/en/api/index.md`, `README.xx.md` → `docs/<xx>/api/index.xx.md`; content under `docs/` is not bundled — see `usage.md`). Links from the index to each class file are within the same `docs/<lang>/api/` directory, so relative links are fine there.
- Class members follow the "Notation of Class Members" notation below. This table is **placed at the top** of the API reference (`## API`). Copy the template below and use it (each table row is shared across languages; only the introductory text differs per language).
- This notation is **governed by "Notation of Class Members" in the documentation convention**. The template below is the form in which it is transcribed into a README; when the governing table changes, align this template with it.

## Notation of Class Members

`README.md` (English):

````markdown
## API

Class members are written with the following notation.

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
````

`README.ja.md` (Japanese):

````markdown
## API

クラスメンバーは以下の表記に従って記述します。

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
````
