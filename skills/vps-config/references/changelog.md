# Change Log

## 2026-06-01 — Initial VPS Setup

Complete guided setup from bare Debian 12 VPS to running Hermes Agent.

### Phase 1: VPS Inspection
- Debian 12 (bookworm), KVM VM, 237GB disk, 7.7GB RAM
- Admin user: Hermes, sudo group

### Phase 2: Security
- Updated all system packages
- Installed essential tools (git, curl, tmux, htop, etc.)
- Configured UFW: SSH (22), HTTP (80), HTTPS (443) only
- Enabled fail2ban (5 retries / 1h ban)
- Hardened SSH: disabled root login, password auth, X11 forwarding
- Enabled unattended security upgrades

### Phase 3: Runtime Dependencies
- Docker 29.5.2 + Compose v2 (pre-installed via Coolify)
- Node.js v22.22.3, npm 10.9.8
- ffmpeg 5.1.9, Playwright Chromium headless shell
- Browser libraries (NSS, ATK, CUPS, etc.)

### Phase 4: Agent User
- Created `agentuser` (unprivileged, in docker group)
- Created standard directories under /opt/agents, /var/lib/agents, etc.

### Phase 5: Hermes Install
- Installed Hermes Agent v0.15.1 via official installer
- Built web dashboard assets
- Installed python-telegram-bot for Telegram gateway
- All core tools available (browser, terminal, code_execution, etc.)

### Phase 6: Configuration
- Model: GLM 5 Turbo via z.ai (provider: zai)
- Telegram: bot token configured, user 8618340015 allowlisted
- Home channel: DM mode (user's Telegram ID)
- Verified Telegram connectivity and GLM API response

### Phase 7: Gateway Service
- Installed `hermes-gateway.service` as systemd service
- Runs as agentuser, enabled at boot
- Telegram connected in polling mode
- Cron ticker active

### Phase 8: Protected Dashboard
- Dashboard on 127.0.0.1:9119 (systemd service)
- socat bridge: 0.0.0.0:19119 → 127.0.0.1:9119
- Traefik reverse proxy via Coolify dynamic config
- Domain: hermes.nickbaskett.com with Let's Encrypt TLS
- Basic Auth protection (credentials at /root/hermes-dashboard/credentials.txt)

### Phase 9: GitHub Backups
- Local backup repo at /home/agentuser/hermes-backup
- Dedicated ed25519 deploy key for GitHub
- Remote: nick-terrant/hermes-vps-backup (private)
- Hourly systemd timer with random 5m delay
- Tracks config, SOUL.md, cron, memories, skills
- Excludes .env, secrets, logs, caches

### Phase 10-12: Finalization
- All health checks passed
- VPS config tracking skill created with 7 reference documents
- Admin sudo configured passwordless for Hermes user
EOF