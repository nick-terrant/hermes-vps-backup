---
name: vps-config
description: Documentation and operational context for this VPS/server configuration. Use before reading, changing, updating, hardening, debugging, or replacing any VPS/server configuration including SSH, firewall, packages, systemd services, Hermes Agent, Traefik/HTTPS, DNS, dashboard, users, backups, or credentials handling. After any server configuration change, update this skill's references and changelog.
---

# VPS Config

This skill documents how this VPS is configured so future agents can safely continue server administration without rediscovering everything from scratch.

## Mandatory workflow

Use this skill whenever the task might affect VPS/server configuration, including:

- SSH, sudo, users, groups, firewall, fail2ban, package management, unattended upgrades.
- Systemd services, timers, long-running daemons, ports, DNS, Traefik, HTTPS/TLS, reverse proxies.
- Hermes Agent install/config, Telegram gateway, dashboard, browser/runtime dependencies.
- Backups, deployment keys, secrets, credentials, API keys, or root-only files.

Before making changes:

1. Read this file.
2. Read the relevant reference file(s) below.
3. Verify current state with commands; do not assume the docs are perfectly current.
4. Avoid printing or committing secrets. Redact `.env`, API keys, bot tokens, passwords, auth files, and private keys.
5. Explain risky changes and ask before modifying SSH, firewall, systemd, Traefik, Caddy, package repositories, credentials, or public exposure.

After making any durable server configuration change:

1. Update the affected reference file(s).
2. Add an entry to `references/changelog.md`.
3. If the change touches secrets, document only the path, owner, permissions, and purpose — never the secret value.
4. If validation commands were run, record the useful result.

## Current high-level state

- Provider/hardware: KVM VM (Debian 12 bookworm)
- Hostname: my-vps
- Admin user: Hermes (sudo, passwordless)
- Runtime user: agentuser (unprivileged, in docker group)
- Public IPv4: 87.106.99.198
- Public IPv6: (none configured)
- Domain/subdomain: hermes.nickbaskett.com (DNS A → 87.106.99.198)
- Intended public ports: 22 (SSH), 80 (HTTP), 443 (HTTPS)
- Hermes gateway service: hermes-gateway.service
- Hermes dashboard service: hermes-dashboard.service (+ hermes-dashboard-bridge.service)
- Dashboard public URL: https://hermes.nickbaskett.com (Basic Auth protected)
- Backup location: /home/agentuser/hermes-backup
- GitHub backup repo: nick-terrant/hermes-vps-backup (private, deploy key)
- Reverse proxy: Coolify Traefik (coolify-proxy container on coolify Docker network)
- Coolify: Running on this VPS managing services via Traefik

## References

Read the relevant docs before acting:

- [Setup source and history](references/setup-source.md)
- [System baseline, users, packages, directories](references/system-baseline.md)
- [Security: SSH, UFW, fail2ban, unattended upgrades](references/security.md)
- [Hermes Agent configuration and services](references/hermes.md)
- [Dashboard, Traefik, DNS, TLS, and ports](references/dashboard-traefik-dns.md)
- [Backups and persistence](references/backups.md)
- [Operational commands](references/operations.md)
- [Change log](references/changelog.md)
