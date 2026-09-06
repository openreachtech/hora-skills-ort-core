# Key composition

The key is one sha256 digest per unit. A record is reused only when the digest is identical, so
everything the digest covers is an input, and everything else is a risk the audit carries.

## What the digest covers

| input | form |
| :-- | :-- |
| the unit's command | the declared string, verbatim |
| the node version and the platform | the running `node` version, plus `platform/arch` |
| keyed environment variables | `NODE_ENV`, `TZ`, `LANG` — an unset variable digests as empty |
| every git-visible file under `cwd` | its path, and a sha256 of its content |
| a path git lists that the disk lacks | the path, digested as `DELETED` |
| the installed tree | `node_modules/.package-lock.json` under the unit's `cwd`, digested for every unit |
| declared extras | each expanded path, digested like any other file |

Git-visible means `git ls-files --cached --others --exclude-standard`, asked **inside the unit's
`cwd`**: tracked files, plus untracked files the repository does not ignore, and only those under
that directory. **The repository's own `.gitignore` is what keeps `node_modules/` and build output
out of the key**, so an ignore rule that stops ignoring something widens the key by itself.

Absence is an input. A file declared as an extra that does not exist yet digests as `DELETED`, so
creating it changes the key.

## What the digest cannot see

- files the repository ignores that no unit declares as an extra
- environment variables other than the three keyed ones
- anything outside the unit's `cwd`, other than the extras it declares — a lockfile, a shared
  package or a config file at a repository root above the unit is not in its key
- **an install that lands outside the unit's directory.** `node_modules/.package-lock.json` is read
  under the unit's own `cwd`; where a workspace root holds the installed tree, that path is missing
  and digests as `DELETED` both before and after `npm install`, so a dependency change moves nothing
  in the key
- services the suite talks to over the network
- state a previous run left behind — a database, a queue, a cache, a temp directory
- binaries installed on the machine outside `node_modules`, and the versions behind them
- the wall clock and the calendar

**When one of these changed, run cold.** The audit is the backstop for the ones nobody thought of,
not the plan for the ones already known.

## The rule for a doubtful input

**When in doubt, put it in.** Widening the key costs one file hash per run, and at worst one
execution that would have been reused. Leaving an input out costs a green that was never measured.

- Declare a whole env file rather than the variables a test appears to read. Which variables a suite
  reads changes without the declaration changing.
- Declare every env file the directory can hold, not only the one in use today — `.env*` covers the
  file that appears next week.
- Prefer one broad pattern over a list that has to be maintained.

## Reading a key change

`--status` prints, per unit, how many passes are recorded, whether the current key is one of them,
and that key **shortened to its first characters**. Two keys differing is a statement that something
under the unit changed; it never says what. To find what, compare the working tree — the key is a
digest, not a diff.
