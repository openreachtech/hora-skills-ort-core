# README Sections

Rules concerning the section structure of a README.

## Scope

The README should describe only the functionality and API exposed to package users. Test/development-only code (e.g., under `app/`, `playground/`) is not something users care about, so it must not be included in the README content. Other directories with a similar nature (e.g., temporary code for samples or verification) are treated the same way.

## Section order

Write sections in the following order, from top to bottom. `*if needed*` marks an optional section, and should be
written at the corresponding position depending on the content.

1. Title + overview (one-line description)
2. Table of contents *if needed* (when there are many sections)
3. Concept / Overview *if needed* (when supplementing the purpose/background)
4. Installation
5. Usage / Features *pick one* (when there are multiple independent features, use Features instead of Usage, and split each feature out into a separate file. See Section rules below.)
6. API
7. Explanation of how it works (e.g., How memoization works) *if needed* (when understanding internal behavior is useful)
8. Contribution
9. License
10. Developer
11. Copyright

The sections listed here are a **basic set**, not an exhaustive one. Sections judged to be useful given the nature and purpose of the repository may be added autonomously, without waiting for instructions (inserted at the appropriate position).

## Section rules

For sections whose content is fixed boilerplate, copy the following templates into the README and replace `<...>`.

### Installation

Almost entirely fixed text. Use the template below, replacing `<node-version>` (e.g., `20.x`) and `<package-name>`. Packages are published to npmjs.com, so `npm install` is all a consumer needs — do not add registry configuration or an authentication step to this section.

`README.md` (English):

`````markdown
## Installation

Requires Node.js <node-version> (the version the CI builds against).

```sh
npm install @openreachtech/<package-name>
```

It is an ES module (`"type": "module"`); import it with ESM `import` syntax.
`````

`README.ja.md` (Japanese):

`````markdown
## インストール

Node.js <node-version> が必要です（CI がビルド対象とするバージョン）。

```sh
npm install @openreachtech/<package-name>
```

ES モジュール（`"type": "module"`）です。ESM の `import` 構文でインポートしてください。
`````

### Features

For libraries with multiple independent features, use `Features` in place of `Usage`, and list only the feature list in the README body. Split the details of each feature out into a separate file, `docs/en/features/<feature>.md` (and, for each language, `docs/<xx>/features/<feature>.xx.md`), and link to it.

- Link destinations should match the README's language (`README.md` links to `docs/en/features/<feature>.md`; `README.ja.md` links to `docs/ja/features/<feature>.ja.md`).
- Links should use absolute GitHub URLs. Split-out files live under `docs/` and are not bundled into the npm package (see `usage.md`).

`README.md` (English):

```markdown
## Features

### (1) <Feature Name>

[Usage of <feature>](https://github.com/openreachtech/<repository>/blob/main/docs/en/features/<feature>.md)

### (2) <Feature Name>

[Usage of <feature>](https://github.com/openreachtech/<repository>/blob/main/docs/en/features/<feature>.md)
```

`README.ja.md` (Japanese):

```markdown
## 機能一覧

### (1) <機能名>

[<機能> の使い方](https://github.com/openreachtech/<repository>/blob/main/docs/ja/features/<feature>.ja.md)

### (2) <機能名>

[<機能> の使い方](https://github.com/openreachtech/<repository>/blob/main/docs/ja/features/<feature>.ja.md)
```

### Contribution

`README.md` (English):

````markdown
## Contribution

Bug reports, feature requests, and code contributions are welcome.

Feel free to contact us through GitHub Issues.

```sh
git clone https://github.com/openreachtech/<repository>.git
cd <repository>
npm install
npm run lint
npm test
```
````

`README.ja.md` (Japanese):

````markdown
## コントリビューション

バグ報告・機能要望・コード貢献を歓迎します。

GitHub Issues からお気軽にご連絡ください。

```sh
git clone https://github.com/openreachtech/<repository>.git
cd <repository>
npm install
npm run lint
npm test
```
````

### License

Write according to `license` in `package.json`.

- `UNLICENSED`: the body is the single word `UNLICENSED`.
- `Apache-2.0`: use the template below.
- `MIT`: use the template below (the current README's format).

`README.md` (Apache-2.0):

```markdown
## License

This project is released under the Apache License 2.0.

For more details, please see [in the LICENSE file](./LICENSE).
```

`README.ja.md` (Apache-2.0):

```markdown
## ライセンス

本プロジェクトは Apache License 2.0 で公開されています。

詳細は [LICENSE ファイル](./LICENSE) を参照してください。
```

`README.md` (MIT):

```markdown
## License

This project is released under the MIT License.

For more details, please see [in the LICENSE file](./LICENSE).
```

`README.ja.md` (MIT):

```markdown
## ライセンス

本プロジェクトは MIT ライセンスで公開されています。

詳細は [LICENSE ファイル](./LICENSE) を参照してください。
```

### Developer

Placed before Copyright. The content is the same across languages (heading is `## Developer` in English / `## 開発者` in Japanese).

```markdown
[Open Reach Tech Inc.](https://openreach.tech)
```

### Copyright

An independent section. The content is the same across languages (heading is `## Copyright` in English / `## 著作権` in Japanese; `<README作成年>` is the year the README was created). Do not guess the year — check the current year with something like `date +%Y` and fill it in.

```markdown
© <README作成年> Open Reach Tech Inc.
```
