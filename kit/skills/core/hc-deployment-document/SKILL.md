---
name: hc-deployment-document
description: >
  Write a server deployment runbook through conversation with whoever will run it — the hosting
  and process-management profile, the first-time build, the repeatable release, migrations,
  rollback and the post-release checks. Every step carries the output that confirms it worked.
  Use when the user asks for deployment steps, a release procedure or a server setup document.
  Executing the deployment and building the CI/CD pipeline are out of scope.
---

# Deployment Document

A skill for writing a **server deployment runbook**, through conversation with the person who will
run it.

The output is not a description of the finished server. It is a document someone follows **line by
line, on a system that is already serving real traffic**. Each line says what to run, what the
right output looks like, and what to do when the output is something else.

## Core principle: the document protects every moment in between, not the end state

A deployment is not a jump from one good state to another. It is a sequence of steps, and a running
system passes through **every state in between**. After any step, the code that is live right now
and the data that exists right now must still work.

Everything else in this skill comes from that:

- Write a schema change so that the **code already deployed** still works afterwards.
- When you rotate a secret, accept **both** the old and the new one for a while.
- Mark every step you cannot undo, and put a backup step right before it.
- A release with no way back is not finished.

**A runbook that only works when every step succeeds is not a runbook.** It describes the happy
path, and nobody opens the document when things are going well.

## The skill is not finished until the operator approves

The document has two states, recorded in its header.

| State | Meaning |
| :-- | :-- |
| `DRAFT` | Written, but not yet confirmed by the person who will run it. Do not deploy from it. |
| `APPROVED` | They have read it and confirmed it. It may be followed. |

- Never mark a document `APPROVED` for them.
- A document with an open question cannot be `APPROVED`. Either answer the question, or drop the
  step and record it as out of scope.

## Procedure

Work through the five phases in order. Do not skip to drafting.

| # | Phase | What happens |
| :-- | :-- | :-- |
| 1 | Settle the profile | Ask what the target actually is — hosting, OS, edge, process manager, database, secrets, TLS. See [interview.md](./references/interview.md). |
| 2 | Read what exists | If the system is already running, read its current unit files, edge configuration, applied migrations and deployed tag before asking about them. |
| 3 | Design for compatibility | Decide, per step, whether it is reversible, whether it interrupts service, and whether it can coexist with the code already deployed. See [compatibility.md](./references/compatibility.md). |
| 4 | Draft | Produce the document from the template. See [document-template.md](./references/document-template.md). |
| 5 | Approve | Present it, ask for confirmation, then set the state to `APPROVED`. |

- **Phase 2 keeps phase 1 short.** If you can read it off the server or out of the repository, do
  not ask. Asking spends the operator's attention on something you could have looked up.
- Phases 1 and 3 repeat. Two or three rounds is normal.

## Rules for the conversation

- **Group questions by topic and ask about five at a time.** A list of twenty gets one answer.
- **Give every question a suggested answer**, so the operator can agree instead of writing. This
  skill assumes Ubuntu LTS, nginx, systemd, git on the server, a `.env` file and Let's Encrypt —
  see [interview.md](./references/interview.md).
- **Always ask two questions**, because no default is safe for either. Is this a first build, or a
  release onto a system already running? And how much downtime is allowed?
- If you ask something and get no answer, put it in the document as an **open question**. Never
  write a guess as though it were decided.

## Every step states how it is verified

If you cannot confirm a step worked, it is not a step. Every step carries all seven of these:

| Field | Why it is not optional |
| :-- | :-- |
| Purpose | One line. If nobody can say why a step is there, nobody can safely skip it or move it. |
| Preconditions | What must already be true. This is what lets someone check the order is right. |
| Command | Exactly what to run. Chapter 0 lists what every placeholder means. |
| **Expected output** | What the screen shows when it worked. Without it, people read "it ran" as "it worked". |
| On failure | What to do. Not "investigate" — which log to open, and which step to go back to. |
| Rollback | How to undo it. If it cannot be undone, say so in those words. |
| Interrupts service? | Yes or no. The operator needs to know before running it, not after. |

## The document is in two parts

Building the server the first time and releasing to it are different jobs. People read them at
different times, in a different frame of mind. Put them in one sequence and the person doing their
fortieth release has to scroll past thirty steps that ran once, a year ago.

| Part | Read | Holds |
| :-- | :-- | :-- |
| I — first-time build | Once | The instance, the OS baseline, networking, git access, environment variables, TLS and keys, the edge, the process manager |
| II — every release | Every time | Fetch, install, build, migrate, restart, reload, verify, roll back, and the checks that were deferred to production |

Chapter 0 belongs to neither part, and everyone reads it. It holds the agreed profile, the
placeholder list, the permissions needed and how long it all takes. It also says **what the
operator's own machine needs**: an SSH client, a key with the right permissions, and the right line
endings on any file they upload.

## The edge configuration chapter is an original, not a copy

The local E2E environment copies its proxy configuration from the chapter that sets up the reverse
proxy. `hb-build-e2e-test-environment` requires that copy, and requires the copy to record where it
came from.

- Write the chapter so it is easy to copy: one file, with the parts that change per environment
  (TLS termination, upstream address, host name) easy to spot.
- Say in the chapter that it is the original. If nobody knows a file is copied elsewhere, they edit
  it as though it were theirs alone.

## Where the document lives

```
docs/deployment/
└── <environment>.md      the runbook for one environment — production.md, staging.md
```

- One environment, one document. Even if two environments differ in only three values, write two
  documents. A runbook full of "unless it is staging, in which case…" gets read wrong under
  pressure.
- If the project already has a place for operational documents, use it, and keep the file name
  `<environment>.md`.
- Write it in the language the person running it speaks, as the documentation convention asks for
  any document written for a reader.

## Scope

| In scope | Out of scope |
| :-- | :-- |
| Writing the runbook | Executing the deployment |
| The server-side steps, whoever runs them | Implementing the CI/CD pipeline |
| One section on what changes when CI runs the steps instead of a person — key handling, the executing user, and what replaces an interactive confirmation | Pipeline configuration, runner setup, workflow files |

Deployment is where a wrong guess costs the most, so the line matters. This skill writes the
document. Someone allowed to break production runs it.

## Anti-patterns

| Anti-pattern | Why it fails | Instead |
| :-- | :-- | :-- |
| A step with a command but no expected output | People read "no error message" as success, even when the command did nothing at all | Say what the screen shows when it worked |
| Dropping a column in the same release that stops using it | The old code is still running while the migration runs, and it asks for a column that is gone | Add first, deploy, then remove in a later release |
| "Restart nginx" in the edge chapter | A restart drops every request in progress, including the operator's own check | Run `nginx -t`, then reload |
| One document for both production and staging | People read the differences as optional, and apply the wrong one under pressure | One document per environment |
| Rollback written as "revert the commit and redeploy" | It says nothing about the data, and the data is the part you cannot revert | Say how far back you can go, where the limit is, and put the backup before it |
| Rotating a secret by replacing the value | Every token issued so far stops working at once, so the rotation causes an outage | Accept old and new together, then drop the old |
| A first-build step left in the release part | It runs again on a live system, a year after anyone remembers what it does | Part I once, part II every time |

## Detail files

- [interview.md](./references/interview.md) — what to ask, the default profile behind each
  question, and what a different answer changes in the document.
- [document-template.md](./references/document-template.md) — the two-part chapter structure, the
  header, and the shape of one step.
- [compatibility.md](./references/compatibility.md) — the rules that keep a running system working
  at every intermediate moment: schema, seeds, environment variables, secrets, API and deploy order.
- [ubuntu-nginx-profile.md](./references/ubuntu-nginx-profile.md) — the default profile in full:
  the OS baseline, `ufw`, the systemd unit, the nginx server block, certbot and log rotation.
- [cloud-and-network.md](./references/cloud-and-network.md) — security groups, IAM, managed
  databases and load-balancer TLS termination per provider, and what replaces them on a VPS or
  on-premises.
