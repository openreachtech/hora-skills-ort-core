# Cloud and network

What the network chapter contains per provider, and what replaces it when there is no provider.
Referenced from the profile in [SKILL.md](../SKILL.md).

## The rules that hold everywhere

Whatever your provider calls its parts, the network chapter answers the same four questions.

| Question | The answer that is always wrong |
| :-- | :-- |
| What can reach port 22? | `0.0.0.0/0`. Always a bastion or a fixed address instead |
| What can reach the application? | Its own port directly. Only 80 and 443 are public, and the application listens on loopback behind the edge |
| What can reach the database? | Anything outside the private network. **Never make the database port public**, on any provider, in any environment |
| What can the server reach outbound? | Everything, without anyone deciding so. It needs package mirrors, the git host and the certificate authority. Everything else is a choice |

**Anyone who can reach the database has the data.** No amount of application-level security makes
up for an open database port. That is why this rule holds whatever your provider calls things.

## Per provider

The names differ. The chapter is the same length either way.

| Concern | AWS | GCP | Azure | OCI / Sakura |
| :-- | :-- | :-- | :-- | :-- |
| Ingress rules | Security Group | Firewall rule (by tag) | Network Security Group | Security List / packet filter |
| Subnet-level rules | Network ACL | — (firewall only) | NSG on the subnet | Security List |
| Instance identity | IAM instance profile | Service account | Managed identity | Instance principal |
| Managed database | RDS | Cloud SQL | Azure Database | managed offering or self-hosted |
| Certificate | ACM (with ALB) | Google-managed | App Service / Front Door | provider's own, or certbot |
| Where TLS ends | ALB, usually | Load balancer | Application Gateway | often on the instance |

## When TLS ends at a load balancer

This is where runbooks most often go subtly wrong, because the server itself never sees HTTPS.

- **nginx listens on 80**, and the redirect to HTTPS moves to the load balancer. Leave the redirect
  on the server and you get a loop.
- **`X-Forwarded-Proto` now matters a lot.** It is the only way the application knows whether the
  request was HTTPS, so generated links and cookie `Secure` flags depend on a header instead of on
  the connection.
- **Only trust that header from the load balancer.** If you accept it from anywhere, the client can
  set it.
- **The health-check path must work without a login**, and must not touch the database. A health
  check that runs a query turns one slow query into a server dropped from the pool.

## Managed databases

- **Check the subnet, not just the security group.** If the application cannot route to the
  database's subnet, the connection times out instead of being refused, and that looks like the
  application hanging.
- **The provider owns the backups, and the schedule.** Chapter 9 says whether the pre-migration
  backup relies on the automatic one or takes its own. If it relies on the automatic one, say
  whether the release window overlaps it.
- **Changing a parameter can restart the database.** A parameter group edit applied "immediately"
  sometimes does. Write whether it does into the step, rather than leaving it to memory.
- **The maintenance window limits when you can deploy.** A release scheduled inside it competes
  with the provider's own restart.

## VPS only, or on-premises

There is no security-group layer, so the host firewall is the only barrier. Chapter 1 therefore
carries weight that chapter 2 would share on a cloud.

| What the cloud provided | What replaces it |
| :-- | :-- |
| Security groups | `ufw`, now the **only** thing between the network and the database port |
| Instance identity for credentials | A file on disk, with the permissions and rotation that means |
| Managed backups | A scheduled dump, **plus a restore you have actually tried**. An untested backup is not a backup |
| Load-balancer TLS | certbot on the server, and the renewal timer is now yours to check |
| Automatic instance replacement | Nothing. Recovery is manual, and belongs in chapter 11 |

**Hybrid is not a third case. It is both cases, one component at a time.** Write down which
component sits where. A reader who assumes everything is in one place applies the wrong half of the
chapter.
