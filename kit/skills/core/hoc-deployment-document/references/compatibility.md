# Compatibility with the system that is already running

The rules that keep a live system working at **every moment during** a release, not just at the end.
Referenced from the compatibility phase in [SKILL.md](../SKILL.md).

They all deal with one situation: **during every release there is a period when the old code and the
new schema are live together.** How long that period lasts varies. That it happens does not.

## Schema

| Change | Rule |
| :-- | :-- |
| Any change to the structure | Add, move, remove — over **three releases**. First add the new shape, then move the data, then remove the old shape. Only remove it once nothing running still reads it |
| Adding a column | Make it nullable, or give it a default, so the `INSERT` in the running code still works. You cannot add `NOT NULL` with no default to a live table |
| Renaming | Four releases: add the new name, write to both, move the readers over, drop the old one. Never all at once |
| Dropping a column | Not in the same release that stops using it. The old code is still running while the migration runs |
| Adding an index to a big table | Say how long it locks, or use the concurrent form if your database has one. A release that looks stuck is usually this |

**Test every migration like this:** if the deployment stopped right after this step, and the old
code kept serving requests, would it still work? If not, add a step in front that only adds.

## Seeds

- **Seeds add rows. They never change them.** A seed that rewrites an existing row is really a data
  migration with the wrong name, and it will run again on the next environment.
- **Use a reserved range of ids**, so you can tell a seeded row by its id, and it cannot clash with
  real data.
- To change an existing row, write a migration. Then it is recorded, ordered, and can be undone.

## Environment variables

The ordering is the whole rule, and it is asymmetric:

| Operation | When |
| :-- | :-- |
| Adding a key | **Before** the new code starts. It reads the key on boot |
| Removing a key | **After** the old code has stopped. It is still reading it until then |

A missing key usually does not crash anything. It arrives as `null`, the application starts, and it
fails later somewhere that says nothing about configuration. Chapter 4 lists the required keys so
you catch this by reading, not by an outage.

## Secrets

**Rotation is a three-step sequence, never a substitution.**

1. Start accepting the new secret **as well as** the old one, wherever they are checked.
2. Start handing out the new one.
3. Drop the old one, once every token made with it has expired.

Replace the value in one step and every token issued so far stops working at that instant. With a
signing key, that signs out every user at once — an outage caused by a maintenance job. Say in the
document how long the overlap lasts and what decides it, which is usually the longest token
lifetime.

## API and deploy order

- **Never release a breaking API change on its own.** Add the new version, move the callers over,
  then remove the old one.
- **Say which side deploys first**, and why. Usually the backend: a frontend asking for a field that
  does not exist yet breaks, while a backend serving a field nobody asks for yet does no harm.
- If both go out in the same release, say how long they disagree and what the user sees during
  that time.

## The edge

- **Run `nginx -t` before every reload.** A syntax error found by the reload is an outage. Found by
  the test, it is a typo.
- **Reload. Do not restart.** A reload brings in new workers and lets the old ones finish their
  requests. A restart drops every request in progress, including the operator's own check.
- Restarting never fixes a config that fails the test. Put the old file back — which is why you copy
  it before editing.

## Irreversible steps

- **List them all in one place**, so the operator knows before starting how many points of no
  return there are.
- **Put a backup step right before each one.** Check the backup exists and is not empty before
  going on.
- Write the rollback around these limits: "you can go back to step 8.4; past that, restore the
  backup from 9.1 and replay". "Revert the commit" talks about the code and says nothing about the
  data, and the data is the part that does not revert.

## Downtime

What the operator answered in the interview decides which of these the document is allowed to use:

| Answer | What the release may do |
| :-- | :-- |
| None | Expand/contract only; every step reversible; the edge reloads, never restarts |
| A few minutes off-hours | A stop-migrate-start release is permitted, with the window stated and a maintenance page if there is one |
| A maintenance page is acceptable | The simple sequence is available, and the document still states the duration and the rollback |

Spreading a migration over three releases when you could have taken a maintenance window wastes two
of them. Taking the whole system down when you could not is an unplanned outage. That is why you ask
before you start writing.
