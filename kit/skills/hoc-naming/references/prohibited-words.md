# Prohibited words and redundant compound words

This lists words that must not be used in naming (as prefixes/suffixes) and the redundant compound words that contain them. Referenced from the naming convention itself (`SKILL.md`).

## Prohibited words (prefixes/suffixes)

The following words are prohibited as prefixes or suffixes in naming.

- `info` (`information`): Adds no new meaning (`User` alone conveys "information about the user"). Also, since `information` has the same singular and plural form, the plural of `UserInfo` cannot be distinguished.
- `data`: Same problem as `info` in that it gives the reader no explanation whatsoever.
- `helper`: Adds no new information to responsibility/role.
- `manager`: A well-known naming anti-pattern that easily leads to a god class.
- `item`: Since an element of a List is already expressed as `item`, there is no need to include it in the name.
- `list`: Name array variables with "the plural form of the word denoting the element" (an array of `User` should be `users`, not `UserList`).
- `util` (`utils`): Adds no new information to responsibility/role.
- `type`: Prohibited as a suffix. Use `Category` instead (`granteeCategory` / `GranteeCategory`, `eventCategory`). `type` collides with the JSDoc type annotation, so a reader cannot tell whether the name means "a JavaScript type" or "a business classification." **The only exception is a name borrowed verbatim from an external vocabulary**, where renaming would cost a translation table between the code and the standard it implements: `mimeType` is the MIME standard's own term, `contentType` is the HTTP `Content-Type` header, and a constant mirrored from an external package keeps that package's word (an external `COLUMN_TYPE` stays `columnTypeName`).

## Do not use redundant compound words

- A "word that adds no new meaning" is redundant. Do not use compound words containing redundant words.

| Redundant compound name | Reason | Good example |
| :-- | :-- | :-- |
| UserInfo | `user` alone conveys "information about the user" | userDetail / userPayment |
| SaveData | Of course data is what gets saved. Describe specifically what | saveUser / saveMessages |
| FileUtil | It's clear it deals with files, but describe specifically what it does | FileNameCollector |
