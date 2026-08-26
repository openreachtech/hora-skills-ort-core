---
name: hoc-classes-inflators
description: "Convention for class inflator methods (binding methods). Defines the pattern of binding the class passed as an argument and returning a derived subclass memoized via BoundCtorRegistry, along with its naming (.as / .use / .to / .of / .from / .via / .each / .by / .with / .for / .on / .onto / .into), arguments, and the policy for overriding abstract members."
---

# Classes: Inflators

This summarizes conventions related to inflator methods (binding methods).

## What is an inflator method

- An **inflator method** is a static method that binds the class passed as an argument (= binding) and returns a **memoized derived subclass**.
- The returned derived subclass is one where the base class's **abstract member has been overridden** to return the binding.
- "The same combination of bindings returns the same derived class" (memoization) is the core of this pattern.
- The implementation is delegated to `BoundCtorRegistry` (`lib/tools/BoundCtorRegistry.js`). Do not cache derived classes yourself.

```javascript
export default class UnionScalar extends BaseScalar {
  // Inflator method: binds the passed schemas and returns a memoized derived subclass
  static of (...schemas) {
    const registry = BoundCtorRegistry.create({
      BaseCtor: this,
    })

    return registry.ensureBoundCtor({
      bindings: schemas,
      deriver: ({ Ctor }) => class extends Ctor {
        /** @override */
        static get boundSchemas () {
          return schemas
        }
      },
    })
  }

  // from delegates to of
  static from (schemas) {
    return this.of(...schemas)
  }

  // abstract member overridden by the inflator
  /** @abstract */
  static get boundSchemas () {
    throw new Error(`${this.name}.get:boundSchemas must be inherited`)
  }
}
```

## Define the inflator as a static method on the target class being bound itself

- An inflator method should be defined as a **static method on the target class being bound itself**, not as an external utility.
- Unless there is a specific reason otherwise, pass **`this`** as `BaseCtor` in `BoundCtorRegistry.create({ BaseCtor })`.
  - By passing `this`, even when the inflator is called from an inheriting subclass, the derived class is generated based on that subclass (respecting the inheritance chain).
  - Hardcoding the class name (`BaseCtor: UnionScalar`) would always anchor to the base class even when called from a subclass, breaking polymorphism.

```javascript
// NG: hardcoding the class name in BaseCtor
static of (...schemas) {
  const registry = BoundCtorRegistry.create({
    BaseCtor: UnionScalar, // ❌️ anchors to UnionScalar even when called from a subclass
  })
  // ...
}

// OK: this for BaseCtor
static of (...schemas) {
  const registry = BoundCtorRegistry.create({
    BaseCtor: this,
  })
  // ...
}
```

- Exception: the **graft family** described later (`.onto()` / `.into()`) intentionally puts `BaseCtor` on the argument side (the direction is reversed).

## Values bound must be objects

- The value (binding) passed to and bound by an inflator **must be an object**. Primitives (strings, numbers, booleans, etc.) cannot be bound.
- Reason: `BoundCtorRegistry` uses the binding as a **WeakMap key** for memoization. WeakMap keys are restricted to objects (and Symbols).
- What is actually bound is typically a class (constructor = a function object), a schema object, or a config object, so this constraint is naturally satisfied.

## Memoization semantics

- The granularity of memoization is the **combination of `bindings` (i.e. reference identity of the objects)**. The same binding returns the same derived class.

```javascript
UnionScalar.of(A, B) === UnionScalar.of(A, B) // true (same binding → same class)
UnionScalar.of(A, B) === UnionScalar.of(A, C) // false (different binding → different class)
```

- The derived class inherits from the base and **retains the base class's name** (`BoundCtorRegistry` names it with `{ [BaseCtor.name]: class extends ... {} }`).

```javascript
const Bound = OriginalCtor.use(FirstConstraint)

Bound.prototype instanceof OriginalCtor // true
Bound.name // 'OriginalCtor'
```

- Since inflators are memoized, they are **safe to call every time**. There's no need for the caller to store the
  inflator's return value in a variable to cache it.

```javascript
// OK: it is fine to inflate on every call (memoization prevents wasteful creation)
inflateMessageCtor () {
  return this.MessageCtor
    .use(this.MessageDeserializerCtor)
    .toKey(this.MessageKeyCtor)
    .toValue(this.MessageValueCtor)
}
```

- **Design philosophy**: if you want to share the same derived class, **pass the same reference**. In particular, with `.by()`, which passes an object literal, if there is an intent to reuse it, make sure to implement it so the same reference is passed. Passing a disposable (one-off) temporary object each time is acceptable (once the key is released it is removed from the WeakMap and does not leak).

## Override abstract members in inflators

- In the derived class returned by `deriver`, **override the base's abstract member** to return the binding.
- When the overridden target is "a getter that holds the constructor of the delegate target class," name that getter `[TargetClassName]Ctor` (see "static getter holding the constructor used for delegation" in the accessor-definition convention). If the target is an undetermined abstract target, a generic name expressing the role (e.g. `TargetCtor`) is fine.
- Annotate overrides with `/** @override */`.

```javascript
// OK: overriding an abstract static getter within deriver ([TargetClassName]Ctor naming)
deriver: ({ Ctor }) => class extends Ctor {
  /** @override */
  static get MessageKeyCtor () {
    return MessageKeyCtor
  }
}
```

## Pass arguments flat

- Inflator methods do not use a named-argument object (`({ ... })`); they receive the arguments to pass **flat** (an **exception** to "arguments should be a single named-argument object" in the method-definition convention).
- Basically **a single argument**. Only use multiple/array arguments when dealing with variadic or array input.
  - Single argument: `use(ConstraintCtor)` / `as(Schema)` / `toKey(MessageKeyCtor)`
  - Variadic: `of(...Ctors)`
  - Single array: `from(Ctors)` (`from` delegates to `of`)
- Reason: this is so calls read **declaratively**, as in `Document.as(bindingSchema)` or `UnionScalar.of(A, B)`.
- **Exception**: as with `.by()`, when the value being bound is **configuration / arguments**, it is acceptable to pass a named-argument object (`Record<string, *>`) as that value (since the value is an object, it also satisfies the WeakMap key constraint).

```javascript
// NG: wrapping in a named-argument object
static use ({ ConstraintCtor }) { /* ... */ }

// OK: flat single argument
static use (ConstraintCtor) { /* ... */ }

// OK: variadic / array argument
static of (...Ctors) { /* ... */ }
static from (Ctors) {
  return this.of(...Ctors)
}

// OK (exception): binding of config/argument can take a named-argument object as its value
static by (argument) { /* argument: Record<string, *> */ }
```

## Naming convention (short, preposition-like names)

Inflator methods are given short, preposition-like names so that `Receiver.word(binding)` reads declaratively. Each word expresses "the relationship between the receiver and the binding." The vocabulary is classified into three families.

### Absorption family (forward direction, result is receiver family)

The receiver absorbs / becomes / uses the binding. `BaseCtor: this`, result is receiver family.

| Method | Semantics | Arguments | Example |
| :-- | :-- | :-- | :-- |
| `.as()` | The binding **matches the identity of the whole class** (becomes that thing itself) | Single | `NodeScalar.as(Constraint)` / `ResponseBody.as(schema)` |
| `.use()` | Binds an **auxiliary used internally** by the receiver (Deserializer / Constraint, etc.) — broad sense | Single | `ConsumerMessage.use(JsonMessageDeserializer)` |
| `.for()` | A narrow sense of `.use()` — the binding is the **target/purpose of specialization** (works "for" something) | Single | `Logger.for(BaseResolverCtor)` |
| `.on()` | A narrow sense of `.use()` — the binding is the **target/trigger of action** (works "on" something) | Single | `Handler.on(ClickEvent)` |
| `.by()` | Binds **input-side** config / argument (acts before the main behavior) | Single (object allowed) | `Converter.by({ precision: 2 })` |
| `.to()` | Binds a **Converter class** (the counterpart of conversion) | Single | `ConsumerMessage.to(Converter)` |
| `.of()` | Binds multiple Ctors **with an ordering (sequence)** | Variadic | `UnionScalar.of(A, B)` |
| `.from()` | **Syntactic sugar** for `.of()` (accepts a single array and delegates to `.of()`) | Single array | `UnionScalar.from([A, B])` |
| `.via()` | Applies the binding to **part of the whole schema** | Single | — |
| `.each()` | Applies one definition to **multiple locations (each element)** (a variant of `.via()`) | Single | `RecordScalar.each(valueSchema)` |
| `.with()` | **Output-side** — binds an accompanying object that **manipulates the output** (acts after the main behavior) | Single | `HttpClient.with(RetryPolicy)` |

- **When there are multiple Converters, distinguish them with an object suffix**: `.toKey()` / `.toValue()` (e.g. `ConsumerMessage.toKey(K).toValue(V)`).

### Graft family (reverse direction, result is argument family)

With `Alpha.word(Beta)`, the receiver (Alpha) is not inflated; instead, **the receiver is grafted/injected onto the argument (Beta) side**. The roles of `BaseCtor` and `bindings` are swapped, and the result becomes the **argument family**.

| Method | Semantics | Example |
| :-- | :-- | :-- |
| `.onto()` | `Alpha.onto(Beta)` — **places** Alpha onto Beta (graft). The result is Beta family | — |
| `.into()` | `Alpha.into(Beta)` — **injects** Alpha into Beta (inject / embed). The result is Beta family | — |

```javascript
// Graft family: argument for BaseCtor, this for bindings (roles reversed)
static onto (TargetCtor) {
  const registry = BoundCtorRegistry.create({
    BaseCtor: TargetCtor, // ← inflate based on the argument side
  })

  return registry.ensureBoundCtor({
    bindings: [
      this, // ← the receiver itself becomes the binding
    ],
    deriver: ({ Ctor }) => class extends Ctor {
      /** @override */
      static get boundSource () {
        return this
      }
    },
  })
}
```

### How to choose a word

- Read `Receiver.word(binding)` as English and choose the word that most naturally expresses the "relationship." If it improves declarative readability, prefer a narrow-sense word (`.for()` / `.on()`) over the broad-sense `.use()`.
- Decision rule when in doubt:
  1. **Is the result you want receiver family or argument family?** If argument family (you want to place the receiver onto the argument), use the graft family (`.onto()` / `.into()`).
  2. If forward direction, choose a word from the absorption family based on **absorb/become/use** for the binding.
     - **Becomes the binding itself** → `.as()`
     - **A tool used internally** → `.use()` (`.for()` if it's a specialization purpose, `.on()` if it's a target of action)
     - **Input-side config** → `.by()` / **output-side manipulation** → `.with()`
     - **Counterpart of conversion** → `.to()` (`.toKey()` / `.toValue()` for multiple)
     - **Multiple with an ordering** → `.of()` (`.from()` for an array)
     - **Applied to a part/each location** → `.via()` / `.each()`

## Chaining

- Multiple inflators can be **chained**. Since each inflator returns a derived subclass, the next inflator can be called on its return value.

```javascript
// OK: chaining use → toKey → toValue to obtain a derived class with three bindings bound together
const MessageCtor = ConsumerMessage
  .use(JsonMessageDeserializer)
  .toKey(SampleMessageKey)
  .toValue(SampleMessageValue)
```
