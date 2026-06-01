# Dashboard, Traefik, DNS, TLS, and Ports

## Architecture
Dashboard runs on localhost (127.0.0.1:9119), bridged to 0.0.0.0:19119 via socat, then proxied through Coolify's Traefik with Basic Auth and automatic HTTPS.

## Services
- `hermes-dashboard.service` — Python dashboard on 127.0.0.1:9119
- `hermes-dashboard-bridge.service` — socat: 0.0.0.0:19119 → 127.0.0.1:9119
- Traefik (Coolify container `coolify-proxy`) — reverse proxy on ports 80/443

## Traefik Dynamic Config
- Path: `/data/coolify/proxy/dynamic/hermes-dashboard.yaml`
- Router: `hermes-dashboard` for Host(`hermes.nickbaskett.com`)
- Middleware: `hermes-auth` (Basic Auth), `hermes-host` (Host header override to 127.0.0.1:9119)
- Backend: `http://10.0.1.1:19119` (coolify bridge gateway)
- TLS: Let's Encrypt via `certResolver: letsencrypt`

## Dashboard Credentials
- Path: `/root/hermes-dashboard/credentials.txt` (root-only, chmod 600)
- Username: hermes
- Basic Auth hash: in Traefik dynamic config

## DNS
- `hermes.nickbaskett.com` → A record → 87.106.99.198
- DNS verified and propagating correctly

## Port Summary
| Port | Interface | Service | Public |
|------|-----------|---------|--------|
| 22 | all | SSH | ✅ (key only) |
| 80 | all | HTTP (Traefik) | ✅ |
| 443 | all | HTTPS (Traefik) | ✅ |
| 8000 | all | Coolify UI | ✅ |
| 9119 | 127.0.0.1 | Hermes dashboard | ❌ |
| 19119 | 0.0.0.0 | socat bridge | ❌ (Docker bridge only) |

## How to Update Dashboard Credentials
1. Generate new hash: `DASH_HASH=$(openssl passwd -apr1 "NEW_PASS")`
2. Update `/data/coolify/proxy/dynamic/hermes-dashboard.yaml` with new hash
3. Update `/root/hermes-dashboard/credentials.txt`
4. Traefik auto-reloads (watches dynamic directory)
EOF