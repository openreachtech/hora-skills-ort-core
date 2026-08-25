---
name: hc-classes-prohibits
description: "Convention on prohibitions in class definitions. Establishes the policy of prohibiting static-only classes and classes without properties (state), and the reasons for them."
---

# Classes: Prohibits

This summarizes prohibitions in class definitions.

## Static-only classes are prohibited

- Classes where every member is static and instantiation is not intended (static-only classes) are prohibited.
- Do not introduce a static-only class merely as a place to put reusable logic. First determine "the object that should own that responsibility," and design it as delegation to that object.

Exception:

- A case where the ancestor class is not a static-only class, and only static members are being overridden using the Template Pattern, is not considered a static-only class.

Reasons:

### (1) A class is the place where "state-dependent behavior" is defined

- A class is the place that defines an object that behaves depending on state. A static-only class has no instance state and is never instantiated, so it does not fulfill a class's true responsibility and is merely a namespace.
- If you just want to reuse some logic, extract it into a single-responsibility class and delegate to it (see reason (3) for details).

- Static-only classes tend to invite global mutable state (module-local variables become referenceable from every static method). Shared mutable state is an anti-pattern that reduces testability, hides dependencies, and produces side effects.
- Note that the same problem applies to mutable variables at ESM module scope, so module-level mutable state should be treated with the same design guideline.

### (2) It hinders polymorphism

- Since a static-only class has no instances, it cannot benefit from polymorphism.
- This makes it hard to accommodate future implementation swaps (test doubles, cached implementations, switching between DB/in-memory, branching by configuration, etc.).
- Since the caller is tightly coupled to the class name, extension requires modifying existing code or adding conditional branches, increasing the cost of change.
- If there is even a slight possibility that multiple implementations will be needed, design with an instance class from the start.

### (3) Static methods should be placed where the responsibility belongs

- If only one class uses a given piece of logic, implement it as that class's own method (private or a regular instance method). There is no reason to extract it into a static-only class or a function.
- If multiple classes use it and they belong to the same inheritance hierarchy, place the shared implementation in the
  base class. Expressing it through inheritance makes the location of responsibility clearer.
- If it does not naturally belong to an inheritance hierarchy and is shared across multiple unrelated classes, extract it into a dedicated single-responsibility class (`Validator` / `Formatter` / `Normalizer` / `Resolver`, etc.), and have each class delegate to it. This makes dependencies explicit, allows state to be held if needed, and makes extension/mocking/testing easier.

### (4) It invites argument forwarding

- Since static methods cannot hold state, they end up having to receive arguments they don't use themselves, purely to forward them to downstream methods.
- An instance class can hold arguments as properties, eliminating the need to forward them around.

```javascript
// NG: static-only (beta is forwarded around by firstMethod)
class StaticOnlyClass {
  static entryPointMethod ({ alpha, beta }) {
    return { first: this.firstMethod({ alpha, beta }) }
  }

  static firstMethod ({ alpha, beta }) {
    return alpha
      + this.secondMethod({ beta })
  }

  static secondMethod ({ beta }) {
    return beta * 100
  }
}

// OK: instance class (arguments held as properties)
class InstanceClass {
  constructor ({ alpha, beta }) {
    this.alpha = alpha
    this.beta = beta
  }

  static create (params) {
    return new this(params)
  }

  entryPointMethod () {
    return { first: this.firstMethod() }
  }

  firstMethod () {
    return this.alpha
      + this.secondMethod()
  }

  secondMethod () {
    return this.beta * 100
  }
}
```

### (5) It increases the cost of rewriting

- When changing a static class into an instance class, every call site must be rewritten.
- Conversely, even if an instance class's design is changed to a static class, the call sites do not need modification and can continue operating as-is.
- Therefore, designing with an instance class from the start is more resilient to change.

### (6) It increases review cost

- Whether the design of a static class is appropriate must be examined every time in a pull-request review.
- If static classes are not used, this cost can be reduced to zero.

## Do not create classes without properties

- Do not create a class that is meant to be instantiated but sets no properties on `this` at all in its constructor (a class with no state).
- A class is the place where "state-dependent behavior" is defined (see reason (1) of "Static-only classes are prohibited" above). A class with no state does not fulfill that responsibility and is merely a container bundling methods together.
- If it cannot hold any properties, that is a sign that "this class has no state of its own." Implement the logic as a method of an appropriate state-holding class, or delegate it to a dedicated single-responsibility class (`Validator` / `Formatter` / `Normalizer` / `Resolver`, etc.) (following reason (3)).

Exception:

- A design such as an abstract base class that holds no state itself while its derived classes hold the properties (state) is not considered a class without state (treated the same as the Template Pattern).

Reason:

- A class with no state always behaves the same on every instantiation, so there is no point in instantiating it. It has essentially the same problems as a static-only class (becoming a namespace, argument forwarding).
