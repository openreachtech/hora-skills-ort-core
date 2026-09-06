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
| Which cloud — AWS / GCP / Azure / OCI / Sakura | — | Security groups, IAM, managed database, where the certificate comes from, whether TLS ends at a load balancer |
| OS | Ubuntu LTS | Package manager, firewall command (`ufw` vs `firewalld`), service names |
| Edge | nginx | Configuration layout, reload procedure, where certificates live |
| Process manager | systemd | How to start, restart, log and recover. **pm2** swaps the unit file for an ecosystem file. **Docker** changes the whole release: build an image, tag it, and roll back by using the previous tag |
| Database — your own, or managed | — | How backups work, where migrations run from, and who tunes the settings |
| How code reaches the server | git on the server | Whether chapter 3 is a deploy key, or artifact delivery from CI |
| Where secrets live | a `.env` file on the server | The environment-variable chapter, its permissions, and the rotation procedure |
| TLS | Let's Encrypt | The certificate chapter and how it renews. If TLS ends at a load balancer, it leaves nginx entirely |

## What not to ask

- Anything you can read off the server or out of the repository.
- Anything whose answer would not change a chapter, a step or an order.
- Which directory, which unit name, which user. Suggest these with the profile and let the operator
  correct you. Correcting is quicker than writing from scratch.

## When the answer is "Docker"

Choosing Docker is not just swapping one command for another. It changes the shape of the whole
release:

| Chapter | With systemd | With Docker |
| :-- | :-- | :-- |
| Code update | fetch a tag, install, build in place | build an image, tag it, push it, pull it |
| Restart | `systemctl restart` | recreate the container from the new tag |
| Rollback | check out the previous tag and rebuild | run the previous tag again — usually the fastest rollback there is |
| Environment variables | `EnvironmentFile` | how the value reaches the container, and where it lives on the host |
| Logs | journald | the container log driver and its rotation |

Ask early. Find out while drafting and you rewrite part II.

## When CI runs the steps

The runbook still covers the server. One section says what changes when a pipeline runs the steps
instead of a person.

- **Keys** — a person uses their own key. A runner needs one of its own, and a way to revoke it.
- **The user running the steps** — rewrite any step that assumes someone is logged in at a terminal.
- **What replaces "check this looks right"** — turn each one into an automated check or a real
  approval gate. You cannot just delete it.

The pipeline's own configuration is out of scope. Say so in the document, so nobody waits for a
workflow file that is never coming.
