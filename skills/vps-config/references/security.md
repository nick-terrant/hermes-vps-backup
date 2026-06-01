# Security Configuration

## SSH
- Hardening config: `/etc/ssh/sshd_config.d/99-agent-vps-hardening.conf`
- Root login: disabled (PermitRootLogin no)
- Password auth: disabled (PasswordAuthentication no)
- Pubkey auth: required
- Max auth tries: 4
- Client alive interval: 300s, count: 2
- X11 forwarding: disabled

## SSH Keys
| User | Key Path | Purpose |
|------|----------|---------|
| Hermes | `~/.ssh/id_ed25519` | Admin SSH access |
| agentuser | `~/.ssh/hermes_backup_deploy_key` | GitHub backup deploy key |

## UFW Firewall
- Status: active
- Default: deny incoming, allow outgoing
- Allowed ports: 22/tcp (SSH), 80/tcp (HTTP), 443/tcp (HTTPS)
- Internal ports: 19119 on br-4a0e87b744a7 and docker0 (Hermes dashboard bridge)

## fail2ban
- Config: `/etc/fail2ban/jail.d/sshd.local`
- SSH jail: enabled, 5 retries, 10m findtime, 1h bantime

## Unattended Upgrades
- Enabled via `/etc/apt/apt.conf.d/20auto-upgrades`
- Auto-update packages: daily
- Auto-upgrade: daily

## Sudo
- Hermes user: passwordless (`/etc/sudoers.d/hermes`)
- agentuser: no sudo access (by design)
EOF