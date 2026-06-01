# Hermes Agent Configuration and Services

## Install Location
- Home: `/home/agentuser/.hermes/`
- Code: `/home/agentuser/.hermes/hermes-agent/`
- Binary: `/home/agentuser/.local/bin/hermes`
- Venv: `/home/agentuser/.hermes/hermes-agent/venv/`

## Model Configuration
- Model: `glm-5-turbo`
- Provider: `zai` (ZhipuAI / z.ai)
- API key location: `/home/agentuser/.hermes/.env` (GLM_API_KEY)
- Endpoint: `https://api.z.ai/api/coding/paas/v4` (auto-detected)

## Telegram Configuration
- Bot token: `/home/agentuser/.hermes/.env` (TELEGRAM_BOT_TOKEN)
- Allowed users: 8618340015 (Nick)
- Home channel: 8618340015 (DM mode)

## Services
| Service | Description | Status |
|---------|-------------|--------|
| `hermes-gateway.service` | Long-running gateway (Telegram + cron) | enabled, active |
| `hermes-dashboard.service` | Dashboard backend on 127.0.0.1:9119 | enabled, active |
| `hermes-dashboard-bridge.service` | socat bridge: 0.0.0.0:19119 → 127.0.0.1:9119 | enabled, active |
| `hermes-git-backup.timer` | Hourly Git backup | enabled, active |

## Config Files
- `/home/agentuser/.hermes/config.yaml` — main config
- `/home/agentuser/.hermes/.env` — secrets (API keys, tokens)
- `/home/agentuser/.hermes/SOUL.md` — agent personality

## Skills
- Directory: `/home/agentuser/.hermes/skills/`
- Includes bundled skills (90+) and custom `vps-config` skill
EOF