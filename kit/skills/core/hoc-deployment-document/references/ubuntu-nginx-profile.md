# The default profile: Ubuntu, nginx, systemd

The default answers in full — the OS baseline, the firewall, the unit file, the server block,
certificate renewal and log rotation. Referenced from the profile in [SKILL.md](../SKILL.md).

> **These are examples.** `<app-dir>`, `<deploy-user>`, `<domain>` and the ports are placeholders,
> filled in by chapter 0. Copy the shape, not the text.

## OS baseline

| Step | Note |
| :-- | :-- |
| A deploy user that is not `root`, with sudo limited to what the release needs | A release should not need full root. Chapter 0 lists what it does need |
| SSH by key only — `PasswordAuthentication no`, `PermitRootLogin no` | Turn this on **after** checking the key works in a second session. Locking yourself out of a new server is the classic first-day outage |
| Swap, sized for the instance | Without it, a build that briefly needs more memory is killed instead of slowed down |
| Timezone and NTP set | Matching up logs across services depends on it, and so do scheduled jobs |
| Automatic security upgrades | **Decide on purpose** whether they may restart services. The default can restart the application in the middle of a request |

## Firewall

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from <bastion-or-fixed-ip> to any port 22 proto tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

- **Never open port 22 to everyone.** Only a bastion or a fixed address.
- **Never open the database port at all.** If the application and the database share a host, they
  talk over loopback or a socket. If they do not, the security group handles it, not `ufw`.
- Run `ufw status numbered` afterwards, and paste its output into the step as the expected
  output.

## The systemd unit

```ini
[Unit]
Description=<app> application server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=<deploy-user>
WorkingDirectory=<app-dir>
EnvironmentFile=<app-dir>/.env
ExecStart=/usr/bin/node server/index.js
Restart=on-failure
RestartSec=5
KillSignal=SIGTERM
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

- **`EnvironmentFile` does not run shell syntax.** `KEY=$OTHER` arrives exactly as written. Quote
  values with spaces, and take care with `#`.
- **Use `Restart=on-failure`, not `always`.** `always` restarts a process that stopped on purpose,
  turning a clean shutdown into a loop.
- **Set `TimeoutStopSec` longer than your slowest request.** Otherwise `SIGKILL` arrives while a
  request is still being served.
- **Give every process its own unit.** Do not start workers and daemons from a shell next to the
  application. They need the same restart policy and the same logging.

## The nginx server block

```nginx
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

server {
  listen 443 ssl http2;
  server_name <domain>;

  ssl_certificate     /etc/letsencrypt/live/<domain>/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;

  client_max_body_size 30m;
  proxy_read_timeout   300s;

  location / {
    proxy_pass http://127.0.0.1:3000;

    proxy_http_version 1.1;
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Upgrade           $http_upgrade;
    proxy_set_header Connection        $connection_upgrade;
  }
}

server {
  listen 80;
  server_name <domain>;
  return 301 https://$host$request_uri;
}
```

**The local E2E environment copies its proxy configuration from this block.** Only three things
differ there: TLS termination, the upstream address and `server_name`. Everything else is copied as
it is, and that is what lets the E2E environment catch a bug in this file.

- **One `proxy_set_header` inside a `location` throws away every header inherited from the `server`
  block.** This is how a header quietly stops being forwarded on one route only.
- **`$connection_upgrade` must be the mapped value**, not a plain `upgrade`, or keep-alive breaks
  for normal requests.
- Deploy with `nginx -t && sudo systemctl reload nginx`. Never `restart`.
- Copy the old file before editing, so putting it back is a step and not a scramble.

## Certificates

```bash
sudo certbot --nginx -d <domain>
systemctl list-timers | grep certbot
```

- **Check the renewal timer exists**, and put that in the step. Getting the certificate works.
  Renewing it is what fails quietly, ninety days later.
- Put `certbot renew --dry-run` in chapter 13 as a routine check, not just at install time.
- **If TLS ends at a load balancer, none of this belongs here.** The certificate lives with the
  provider, nginx listens on 80, and `X-Forwarded-Proto` becomes the only way the application knows
  whether the request was HTTPS. That makes it essential, not just informative.

## Log rotation

```
<app-dir>/logs/*.log {
  daily
  rotate 14
  compress
  delaycompress
  missingok
  notifempty
  copytruncate
}
```

- Use `copytruncate` when the process holds the file open and has no reopen signal. If it does have
  one, use `postrotate` with that signal instead — `copytruncate` can lose the lines written while
  it copies.
- If logs go to journald instead, `SystemMaxUse` in `journald.conf` sets the limit. It is **close to
  unlimited by default**, and it really does fill disks months later.
- Put a disk-usage check in chapter 13. A deployment that ran fine for six months and then stopped
  is usually this.
