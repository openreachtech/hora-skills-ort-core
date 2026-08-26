# Sample Code

Rules concerning the code examples (sample code) in a README.

- Variable names in code examples should take the form `prefix + main suffix`, with emphasis on the main suffix (the trailing word).
  - Main suffix (trailing): a word representing the kind of value the variable holds. In most cases this corresponds to the return type of a method (context-dependent; e.g., if the return value is a constructor, use `Ctor`). In the examples within this rule, the generic word `Class` is used to illustrate it.
  - Prefix (leading): a word used to distinguish individual instances. Instead of sequential letters like `A`, `B`, use `Alpha`, `Beta`, `Gamma`, etc.
  - Thus `AlphaClass`, `BetaClass` are correct. A form like `ClassA`, `ClassB` — placing the main suffix first and appending a sequence number at the end — violates this rule.
