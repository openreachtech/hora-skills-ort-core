# Asynchronous creation `.createAsync(...)`

This covers the convention for `.createAsync(...)`, defined when the arguments passed to the constructor need to be generated via asynchronous processing. Referenced from the method-definition convention itself (`SKILL.md`).

- When the arguments passed to the constructor need to be generated via asynchronous processing, define `static createAsync (...)`.
- The JSDoc of `.createAsync(...)` should conform to that of `.create(...)` (the return value becomes `Promise<InstanceType<T>>`).
- Calling `new this(...)` directly from `.createAsync(...)` is prohibited. In principle, `.createAsync(...)` should
  return the return value of `.create(...)`.

```javascript
// NG: calling new this(...) directly from createAsync
static async createAsync () {
  const characters = await this.resolveCharacters()

  return new this({ // ❌️
    characters,
  })
}

// OK: prepare the arguments asynchronously, then return the return value of create()
static async createAsync () {
  const characters = await this.resolveCharacters()

  return this.create({
    characters,
  })
}
```

Example:

```javascript
export default class RandomTextGenerator {
  /**
   * Constructor.
   *
   * @param {{
   *   characters: string
   * }} params - Parameters.
   */
  constructor ({
    characters,
  }) {
    this.characters = characters
  }

  /**
   * Factory method.
   *
   * @template {X extends typeof RandomTextGenerator ? X : never} T, X
   * @param {{
   *   characters?: string
   * }} [params] - Parameters for the factory method.
   * @returns {InstanceType<T>} Instance of this class.
   * @this {T}
   * @public
   */
  static create ({
    characters = this.DEFAULT_CHARACTERS,
  } = {}) {
    return /** @type {InstanceType<T>} */ (
      new this({
        characters,
      })
    )
  }
}
```
