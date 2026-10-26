# Method name verbs (choosing among highly abstract verbs)

This lists how to choose among highly abstract verbs in method names. Referenced from the naming convention itself (`SKILL.md`).

Creation-related:

| Verb | Usage |
| :-- | :-- |
| `create~` | Instantiate an instance from a class |
| `generate~` | Generate a primitive value |
| `build~` | Generate a temporary object |
| `make~` | Not used |
| `define~` | Define a function (a handler, a callback) that is called later |
| `wrap~` | Not used. It names the mechanism, not what the function is for; name the function with `define~` |

- `build~` produces an object. A method that returns a function is `define~`, never `build~`.

Retrieval-related:

| Verb | Usage |
| :-- | :-- |
| `find~` | Retrieve an entity from the DB (anything wrapping `Model.findOne()`/`Model.findAll()` uses `find`, not `get`/`fetch`) |
| `fetch~` | Access an external API to retrieve data |
| `extract~` | Extract data from a variable/property |

Update-related:

| Verb | Usage |
| :-- | :-- |
| `save~` | Wraps processing that persists to the DB |
| `send~` | Access an external API to update external data |
| `set~` | Property update (rarely used since setters are prohibited) |

Processing-related:

| Verb | Usage |
| :-- | :-- |
| `process~` | A hook that transforms a value on its way to where it is used; its default does nothing (`#processHandlerArguments()`) |
| `filter~` | A hook that narrows a collection on its way; its default keeps everything |

- A hook does not take `build~`. That verb belongs to the entry point of a class that builds, and a hook inside such a class speaking it too leaves two members saying they produce the class's result.

- For other verbs, choose a word that is faithful to the method's responsibility.
