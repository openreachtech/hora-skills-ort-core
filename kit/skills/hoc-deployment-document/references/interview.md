# Interview

What to settle before drafting, the default behind each question, and what a different answer
changes. Referenced from the procedure in [SKILL.md](../SKILL.md).

## Running a round

1. **Read the server and the repository first.** If the system is already running, you can read the
   unit file, the edge configuration, the list of applied migrations and the deployed tag. Every one
   you read is a question you do not have to ask.
2. **Put the likely answer in the question.** "systemd, unless you are already on pm2?" takes one
   word to answer. "How do you manage processes?" costs the operator a paragraph.
3. **Ask what changes the document**, not what fills it in. If both answers give you the same
   chapters, it is a placeholder value, not a question.
4. **Five per round.** Group them by topic so one round can be answered in one sitting.
5. **Write every answer into the profile table straight away.** An answer left in the conversation
   gets asked again two rounds later.

## The two that are always asked

Neither has a safe default, and both change the shape of the whole document.

| Question | Why no default is safe |
| :-- | :-- |
| **First build, or a release onto a system already serving traffic?** | It decides whether you write part I at all, and whether every step in part II has to work alongside the code already deployed. Guess "first build" and you write a document that breaks a live system. |
| **How much downtime is allowed — none, a few minutes at night, or a maintenance page?** | It decides whether schema changes have to be spread over three releases, or whether you can stop, migrate and start. This is a business answer, not a technical one. |

## The profile

Each of these has a default. Ask the operator to confirm it, rather than asking an open question.

| Question | Default | What a different answer changes |
| :-- | :-- | :-- |
| Hosting — cloud, VPS only, on-premises, hybrid | — *(always ask)* | Whether there is a security-group chapter at all, and every network assumption under it |
| What runs the application (a server you manage, a managed container service, or a serverless function) | a server you manage | Whether the OS, edge and process-manager rows below apply at all. A managed container service or a serverless function drops them and brings its own chapters, as the last two sections of this file lay out |
| Which cloud — AWS / GCP / Azure / OCI / Sakura | — | Security groups, IAM, managed database, where the certificate comes from, whether TLS ends at a load balancer |
| OS | Ubuntu LTS | Package manager, firewall command (`ufw` vs `firewalld`), service names |
| Edge | nginx | Configuration layout, reload procedure, where certificates live |
| Process manager | systemd | How to start, restart, log and recover. **pm2** swaps the unit file for an ecosystem file. **Docker** changes the whole release: build an image, tag it, and roll back by using the previous tag |
| Database — your own, or managed | — | How backups work, where migrations run from, and who tunes the settings |
| How code reaches the server | git on the server | Whether chapter 3 is a deploy key, or artifact delivery from CI |
| Where secrets live | a `.env` file on the server | The environment-variable chapter, its permissions, and the rotation procedure |
| TLS | Let's Encrypt | The certificate chapter and how it renews. If TLS ends at a load balancer, it leaves nginx entirely |

The OS, edge and process-manager rows apply only when the application runs on a server you manage.

## What not to ask

- Anything you can read off the server or out of the repository.
- Anything whose answer would not change a chapter, a step or an order.
- Which directory, which unit name, which user. Suggest these with the profile and let the operator
  correct you. Correcting is quicker than writing from scratch.

## When the answer is "Docker"

Choosing Docker is not just swapping one command for another. It changes the shape of the whole
release. Docker here runs on a server you still manage. A container the platform runs for you is
the next section.

| Chapter | With systemd | With Docker |
| :-- | :-- | :-- |
| Code update | fetch a tag, install, build in place | build an image, tag it, push it, pull it |
| Restart | `systemctl restart` | recreate the container from the new tag |
| Rollback | check out the previous tag and rebuild | run the previous tag again — usually the fastest rollback there is |
| Environment variables | `EnvironmentFile` | how the value reaches the container, and where it lives on the host |
| Logs | journald | the container log driver and its rotation |

Ask early. Find out while drafting and you rewrite part II.

## When the answer is a managed container service

Cloud Run, ECS on Fargate, Azure Container Apps, or App Runner for a service that already exists
(AWS no longer opens App Runner to new customers). The platform runs the container, so there is no
operating system, firewall, nginx or unit file to write. Part I shrinks to what the platform needs,
and part II becomes build, deploy a revision, and move traffic.

| Chapter | On a server you manage | On a managed container service |
| :-- | :-- | :-- |
| 1 Infrastructure | OS baseline, deploy user, `ufw`, swap | None of it is written. The chapter holds the project or account, the registry, and the identity the service runs as |
| 2 Cloud and network | Security groups | Who may reach the service (public or authenticated), the private path to the database, outbound access, and what the runtime identity may do: pull the image and read secrets |
| 3 Git access | A deploy key, git on the server | Image build and registry: where the image is built, the tag scheme (a git tag or commit, never `latest`), who may push and who may pull |
| 4, 5 Environment and keys | A `.env` file, key files | The platform's secret manager, referenced by name and never copied. A changed secret reaches the application only when a new revision or task starts, and the step says which |
| 6 The edge | nginx | No proxy file. The platform's managed TLS and domain binding, or its load balancer. The chapter records what the platform fixes: port, body size, request timeout, health-check path |
| 7 Process management | A systemd unit | The service definition: CPU and memory, the instance range, concurrency, timeout, health check. Logs go through the platform, with no rotation to write |
| 8 Updating the code | Fetch a tag, build in place, restart | Build the image once, tag it, push it, deploy it as a new revision |
| 9 Migrations | Run on the server | A one-off job or task started from the image being released, with its own timeout, secrets and database path. Never the serving container's start command, which every instance would run |
| 11 Rollback | Check out the previous tag | Send traffic back to the previous revision (a task definition revision on ECS). Moving traffic back can skip the checks a deploy runs, so verification runs after a rollback too |
| 13 Routine operations | certbot, `logrotate` | No certificate or log rotation. Backups of a managed database, secret expiry and registry cleanup stay |

- **Name the path to the database in chapter 2.** On Cloud Run it is the built-in Cloud SQL
  connection or a VPC connection. Elsewhere it is the network the database accepts connections
  from, never a public database port.
- **A revision switch is still a moment when old code and the new schema are live together.**
  [compatibility.md](./compatibility.md) applies unchanged, and rollback still stops at the data.

Ask early. Find out while drafting and you rewrite parts I and II.

## When the answer is a serverless function

AWS Lambda, or Cloud Run functions. The chapter changes are those of a managed container service,
except these:

| Chapter | Becomes |
| :-- | :-- |
| 3 Git access | Build the package or image and publish it as a version |
| 6 The edge | Which of the function URL or the gateway terminates TLS, and what it forwards |
| 7 Process management | The function configuration: memory, timeout, concurrency limits, the trigger |
| 8 Updating the code | Publish a version, then move the alias or the traffic to it |
| 11 Rollback | Point the alias at the previous version (Cloud Run functions deploy as Cloud Run services, so they roll back by revision) |

A function adds three questions that a long-running service answers by itself.

- **Cold start.** The first request after idle is slower. Time it as its own verification step, and
  say whether warm instances are kept: provisioned concurrency on Lambda, which needs a published
  version or alias, or minimum instances on Cloud Run.
- **Concurrency.** Each concurrent request can hold its own database connection. Chapter 2 says what
  stands between the function and the database, such as a connection proxy or a concurrency cap, so
  a traffic spike does not use up the database's connections.
- **Timeouts.** The function and every gateway in front of it each have a maximum. Confirm both
  against the longest request, and write the numbers in chapter 0. Work that outlasts them moves to
  a job or a queue.

Two more rules follow from the same model.

- **Keep no state on the instance.** Local disk and memory disappear between instances, and work
  started before the response may never finish after it.
- **Never run migrations from the function.** Its timeout bounds them. Chapter 9 runs them as a
  one-off container job, or from a machine with a network path to the database.

## When CI runs the steps

The runbook still covers the server. One section says what changes when a pipeline runs the steps
instead of a person.

- **Keys** — a person uses their own key. A runner needs one of its own, and a way to revoke it.
- **The user running the steps** — rewrite any step that assumes someone is logged in at a terminal.
- **What replaces "check this looks right"** — turn each one into an automated check or a real
  approval gate. You cannot just delete it.

The pipeline's own configuration is out of scope. Say so in the document, so nobody waits for a
workflow file that is never coming.
