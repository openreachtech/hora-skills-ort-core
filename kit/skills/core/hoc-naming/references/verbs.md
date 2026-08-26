# Method name verbs (choosing among highly abstract verbs)

This lists how to choose among highly abstract verbs in method names. Referenced from the naming convention itself (`SKILL.md`).

Creation-related:

| Verb | Usage |
| :-- | :-- |
| `create~` | Instantiate an instance from a class |
| `generate~` | Generate a primitive value |
| `build~` | Generate a temporary object |
| `make~` | Not used |

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

- For other verbs, choose a word that is faithful to the method's responsibility.
