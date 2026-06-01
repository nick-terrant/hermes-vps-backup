# Operational Commands

## Hermes Gateway
```bash
# Check status
sudo systemctl status hermes-gateway.service
sudo -H -u agentuser bash -lc 'export PATH="$HOME/.local/bin:$PATH"; hermes gateway status'

# View logs
sudo journalctl -u hermes-gateway.service -f --no-pager
sudo -H -u agentuser bash -lc 'tail -f /home/agentuser/.hermes/logs/gateway.log'

# Restart
sudo systemctl restart hermes-gateway.service
```

## Hermes Dashboard
```bash
# Check status
sudo systemctl status hermes-dashboard.service
sudo systemctl status hermes-dashboard-bridge.service

# View logs
sudo journalctl -u hermes-dashboard.service -f --no-pager

# Restart (both services)
sudo systemctl restart hermes-dashboard.service hermes-dashboard-bridge.service
```

## Backup
```bash
# Check timer
systemctl list-timers hermes-git-backup.timer

# Run manually
sudo -u agentuser /home/agentuser/hermes-backup/sync-hermes-backup.sh

# View backup logs
sudo journalctl -u hermes-git-backup.service --no-pager

# Check repo status
sudo -H -u agentuser bash -lc 'cd /home/agentuser/hermes-backup && git log --oneline -5 && git status -sb'
```

## Hermes Diagnostics
```bash
sudo -H -u agentuser bash -lc 'export PATH="$HOME/.local/bin:$PATH"; hermes doctor'
sudo -H -u agentuser bash -lc 'export PATH="$HOME/.local/bin:$PATH"; hermes version'
```

## Firewall
```bash
sudo ufw status verbose
sudo ufw allow/deny <port>
```

## SSH
```bash
sudo sshd -t              # Test config
sudo systemctl reload ssh  # Reload after config change
```

## Docker/Traefik
```bash
sudo docker ps                             # List containers
sudo docker logs coolify-proxy --tail 20   # Traefik logs
sudo docker restart coolify-proxy          # Restart Traefik
```

## System
```bash
sudo DEBIAN_FRONTEND=noninteractive apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y
sudo reboot
```
EOF