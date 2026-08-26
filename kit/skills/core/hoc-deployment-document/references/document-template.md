# The document template

The chapter structure, the header, and the shape of a single step. Referenced from the drafting
phase in [SKILL.md](../SKILL.md).

## Header

```markdown
# Deployment — production

| | |
| :-- | :-- |
| State | DRAFT |
| Environment | production |
| Applies to | the release procedure from v1.4.0 onward |
| Last updated | 2026-08-23 |
| Approved by | — |
```

`State` stays `DRAFT` until the person who will run it confirms it. One unanswered question
anywhere in the document keeps it `DRAFT`.

## Chapter 0 — target and assumptions

Read by everyone, part of neither part.

- **The agreed profile**, as a table: hosting, OS, edge, process manager, database, secret store,
  TLS. One row per answer from the interview.
- **The placeholder list.** Every `<host>`, `<domain>`, `<deploy-user>` and `<app-dir>` used below,
  with its real value — or the words "ask the infrastructure owner".
- **The permissions needed**, and who grants them.
- **How long it takes**, and which steps take most of that.
- **What the operator's own machine needs.** An SSH client. A private key at `600` — **Windows
  cannot set that**, so fix the key's ACL there instead. And LF line endings on anything you upload:
  a shell script or unit file with CRLF fails in ways that never mention line endings.

## Part I — first-time build

Read once. Every chapter here is skipped on a system that already exists.

| # | Chapter | Holds |
| :-- | :-- | :-- |
| 1 | Infrastructure | The instance, OS baseline, the deploy user and its sudo rights, SSH keys, `ufw`, swap, timezone, the language runtime, middleware, directories and ownership |
| 2 | Cloud and network | Security groups: port 22 from a bastion or a fixed address only, 80 and 443 public, and **the database port never public**. Also outbound rules, network ACLs, IAM roles, managed-database subnets, and the load balancer's health-check path |
| 3 | Git access | A read-only deploy key, `known_hosts` set up in advance, which branches and tags to use, who owns the repository directory, and whether a shallow clone is right here |
| 4 | Environment variables | Where the file lives, who owns it, its `600` mode, how it maps to the environment name, systemd's `EnvironmentFile`, and every required key. Note that **a missing key arrives as `null` and the application starts anyway** |
| 5 | Security and keys | Getting and renewing the certificate, HSTS and cipher settings, making and placing signing and encryption keys with the right permissions, encryption at rest, and how to rotate a key while **accepting both old and new for a while** |
| 6 | The edge | The server block, reverse proxy, WebSocket upgrade, body size, timeouts, static files and single-page fallback, then `nginx -t` and reload. **The E2E environment copies its proxy configuration from this chapter** |
| 7 | Process management | The unit file, the restart policy, **every** process the product needs including workers and daemons, and where logs go and how they rotate |

## Part II — every release

Read every time. This is the part that gets worn out.

| # | Chapter | Holds |
| :-- | :-- | :-- |
| 8 | Updating the code | The main sequence: fetch the tag, install dependencies, build, migrate, restart, reload. The **order**, and whether each stage interrupts service |
| 9 | Migrations and seeds | Take the backup first, check the environment name, spread structural changes over releases, say how long a big table locks, and remember seeds only ever add rows |
| 10 | Verification | One observable check per step — an HTTP status, a line in a log, a row in the database, a process state |
| 11 | Rollback | How far back you can go, where the limit is, and the backup that sits right before it |
| 12 | Checks left until production | What you could not test beforehand, and so have to test here |
| 13 | Routine operations | Backups, certificate renewal checks, log rotation, monitoring, disk usage |
| — | Appendix | What changes when CI runs these steps instead of a person |

## The shape of one step

````markdown
### 8.3 Apply the migrations

**Purpose** — bring the schema to the revision the new tag expects.

**Preconditions** — 8.2 completed; the backup from 9.1 exists and its size is non-zero.

```bash
cd <app-dir> && NODE_ENV=production npx sequelize-cli db:migrate
```

**Expected output** — one `== <name>: migrated` line per pending migration, then no error. No
pending migrations prints nothing and exits 0.

**On failure** — do not re-run. Read `<app-dir>/logs/migrate.log`, and go to 11.2.

**Rollback** — `db:migrate:undo` restores the previous revision **only if** the migration was
reversible; the ones that are not are listed in chapter 9.

**Interrupts service?** No, but a table over roughly a million rows locks for the duration —
see 9.4.
````

Every step has all of these fields. Without **Expected output** nobody can tell whether it worked.
Without **Interrupts service?** the operator finds out by watching the alerts.

## Open questions

Anything asked and not answered goes in a section of its own, with the chapter it blocks:

```markdown
## Open questions

| # | Question | Blocks | Asked |
| :-- | :-- | :-- | :-- |
| Q1 | Does the managed database's automatic backup window overlap the release window? | 9.1 | 2026-08-23 |
```

Never replace one of these with an answer that merely sounds right. Someone runs a guess in a
deployment document. Nobody reviews it.

## What "out of scope" means here

The document stops where your authority stops. Say plainly that it does not cover running the
deployment, and does not cover building the pipeline. Then nobody waits for a workflow file that
was never coming.
