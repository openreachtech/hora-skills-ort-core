# Unit declaration

Units are declared in `.hora-cache.json` at the project root. The file is committed, so every machine
caches the same units; the directory it names is machine-local. The CLI reads the file from the
directory it is started in, so **the project root is also where the command runs**.

## A worked declaration

```json
{
  "cacheDir": ".hora-cache",
  "units": {
    "backend": {
      "cwd": "myproject-backend",
      "command": "npm test",
      "extras": [".env*"]
    },
    "frontend-admin": {
      "cwd": "myproject-frontend-admin",
      "command": "npm test",
      "extras": [".furo-env*"]
    },
    "frontend-public": {
      "cwd": "myproject-frontend-public",
      "command": "npm test",
      "extras": [".furo-env*"]
    }
  }
}
```

- **The unit name is an interface.** It is the argument that selects the unit on the command line,
  the directory records are written to, and the name a report cites. Renaming a unit discards its
  records.
- **`cwd` is relative to the project root**, and it is the directory git is asked about. A unit whose
  `cwd` is not inside a git working tree cannot be keyed.
- **`command` runs through a shell**, so a chained command is legal — and the whole string is part of
  the key.

## Granularity

- **One directory, one command.** A repository run by a single test command is a single unit.
- **An ordered chain stays whole.** Where a project requires suites to run in a defined order, the
  ordered chain is one unit; splitting it into per-suite units caches a passing fragment of an order
  that was never run.
- **Split a repository into two units only when the commands are independent** — neither builds,
  migrates or seeds anything the other needs.
- **Prefer the `cwd` that holds the unit's own `node_modules`.** The installed tree enters the key as
  `node_modules/.package-lock.json` beneath `cwd`, so a unit pointed at a directory whose
  dependencies are installed somewhere above it is keyed without them, and every dependency change
  is invisible until the next cold run.
- **Coarse units lose hits and keep the truth.** A unit that covers more files is missed more often,
  which costs time; a unit that covers fewer inputs than its command reads is reused wrongly, which
  costs a verdict.

## Extras patterns

`extras` names files git ignores that behavior depends on. Each entry is relative to the unit's
`cwd`.

| pattern | matches |
| :-- | :-- |
| `.env` | that one path. A path that does not exist digests as `DELETED`, so creating it changes the key |
| `.env*` | every name in the directory that starts with `.env` |
| `config/local*.json` | every matching name in `config/`. The directory part is literal — only the file name may hold `*` |

- **Extras only widen the key.** An entry that matches nothing changes nothing, and an entry that
  matches too much only costs hashing time.
- **A pattern must match files only.** Expansion lists names, not file types, and a directory the
  pattern catches is then read as a file and ends the run with a read error. Where a directory could
  answer to the pattern — `.env*` beside an `.env.d/` — narrow the pattern to the names that are
  files.
- **Declare env files, not the variables inside them.** The three environment variables the key reads
  directly are fixed; everything else reaches the suite through a file.
- **A file already visible to git does not belong in extras.** It is in the key already.
- `node_modules/.package-lock.json` is added to every unit without being declared. Declaring it again
  changes nothing.

## Wiring the scripts

```json
{
  "scripts": {
    "test": "mentsu-testcase-cache",
    "test:cold": "mentsu-testcase-cache --cold",
    "test:status": "mentsu-testcase-cache --status",
    "test:clear": "mentsu-testcase-cache --clear"
  }
}
```

Add the cache directory to `.gitignore`:

```
.hora-cache/
```

`--clear` is safe at any time: it discards records only, so the next run executes every unit.
**Clearing after a mismatch is not a fix** — the mismatch has already discarded everything, and what
remains to do is the key definition.
