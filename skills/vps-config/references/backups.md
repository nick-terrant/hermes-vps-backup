# Backups and Persistence

## Git Backup
- Local repo: `/home/agentuser/hermes-backup`
- Remote: `git@github.com-hermes-backup:nick-terrant/hermes-vps-backup.git`
- Branch: `main`
- Timer: `hermes-git-backup.timer` (hourly, systemd)

## Tracked Files
- `config.yaml`
- `SOUL.md`
- `cron/`
- `memories/`
- `skills/`

## Excluded Files (intentional)
- `.env` (API keys, tokens)
- `auth.lock`, `gateway.lock`, `gateway.pid`
- `gateway_state.json`, `channel_directory.json`
- `logs/`, `audio_cache/`, `image_cache/`, `cache/`
- `hermes-agent/`, `node_modules/`, `__pycache__/`

## Sync Script
- Path: `/home/agentuser/hermes-backup/sync-hermes-backup.sh`
- Runs: rsync → git add → git commit (if changed) → git push

## Deploy Key
- Private key: `/home/agentuser/.ssh/hermes_backup_deploy_key`
- Public key: added to GitHub repo as deploy key with write access
- SSH config: Host `github.com-hermes-backup` in `~agentuser/.ssh/config`

## Manual Backup Run
```bash
sudo -u agentuser /home/agentuser/hermes-backup/sync-hermes-backup.sh
```
EOF