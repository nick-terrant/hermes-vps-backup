# System Baseline

## OS and Hardware
- OS: Debian GNU/Linux 12 (bookworm)
- Kernel: Linux 6.1.0-44-amd64 (x86_64)
- Virtualization: KVM
- Disk: 237 GB total, ~195 GB free
- RAM: 7.7 GB total, ~4 GB available
- Swap: 2 GB

## Users
- `Hermes` (UID 1000): Admin user, sudo group, passwordless sudo
- `agentuser` (UID 1001): Unprivileged runtime user, in docker group, no password

## Key Directories
| Path | Owner | Purpose |
|------|-------|---------|
| `/home/agentuser/.hermes/` | agentuser | Hermes config, data, code, sessions, logs |
| `/home/agentuser/.hermes/hermes-agent/` | agentuser | Hermes source code |
| `/home/agentuser/.hermes/.env` | agentuser | API keys and secrets (NEVER commit to git) |
| `/home/agentuser/.hermes/config.yaml` | agentuser | Main Hermes configuration |
| `/home/agentuser/hermes-backup/` | agentuser | Git backup repo |
| `/var/log/agents/` | agentuser | Log directory (configured, not actively used) |
| `/etc/agents/` | root:agentuser | Config directory (configured, not actively used) |

## Installed Packages (notable)
- Docker 29.5.2 + Compose v2
- Node.js v22.22.3, npm 10.9.8
- Python 3.11.2
- ffmpeg 5.1.9
- Playwright Chromium (headless shell v1223)
- Coolify + Traefik reverse proxy
- rsync, tmux, htop, btop, ripgrep, jq, git

## Docker Networks
- `coolify` (10.0.1.0/24) — main Coolify network, Traefik runs here
- `docker0` (10.0.0.0/24) — default bridge

## Docker Containers
- coolify-proxy (Traefik v3.6) — ports 80, 443
- coolify — self-hosted PaaS on port 8000
- coolify-db, coolify-realtime, coolify-redis, coolify-sentinel
- hermes-dcwdf2ibg4e53f1e4ql8pjlq — Hermes container (Coolify-managed)
- Other services: Rocket.Chat + MongoDB
EOF