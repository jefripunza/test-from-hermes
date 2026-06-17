---
name: self-hosting-hermes
description: "Run Hermes as a background daemon (gateway) + WebUI on Linux/WSL."
version: 1.2.0
author: Hermes Agent
platforms: [linux, wsl]
metadata:
  hermes:
    tags: [hermes, daemon, gateway, webui, wsl, deployment]
    related_skills: [hermes-agent]
---

# Self-Hosting Hermes

Run Hermes Agent continuously as a background service with a web UI.

## Prerequisites

- Hermes installed (`hermes status` OK)
- WSL: `systemd=true` in `/etc/wsl.conf` (check with `cat /etc/wsl.conf`)
- WSL: `sudo loginctl enable-linger $USER` for service survival after logout

## Gateway Daemon (systemd)

### Install & Start

```bash
# Install systemd user service
printf "Y\nY\n" | hermes gateway install

# Manage
hermes gateway status
hermes gateway start|stop|restart
journalctl --user -u hermes-gateway -f
```

**PITFALL:** `hermes gateway install` has 2 interactive prompts (start now? start on boot?). No `--yes` flag. Piping `printf "Y\nY\n"` is the workaround.

**PITFALL:** WSL gateway install warns "systemd services may not survive WSL restarts" — this is cosmetic. With `systemd=true` in `/etc/wsl.conf` + `loginctl enable-linger`, it persists fine.

### Verification

```bash
hermes gateway status
# Expect: active (running)
```

## Hermes WebUI

### Install

```bash
git clone https://github.com/nesquena/hermes-webui.git ~/apps/hermes-webui
cd ~/apps/hermes-webui
```

### .env Config

Create `.env` in repo root:

```env
HERMES_WEBUI_AGENT_DIR=/path/to/hermes-agent
HERMES_WEBUI_PYTHON=/path/to/hermes-agent/venv/bin/python3
HERMES_WEBUI_HOST=0.0.0.0
HERMES_WEBUI_PORT=8787
```

- `HERMES_WEBUI_AGENT_DIR` points to the hermes source clone
- `HERMES_WEBUI_PYTHON` uses the hermes venv's python (has all deps)
- `0.0.0.0` binds on all interfaces (use `127.0.0.1` for loopback-only)

### Start (Manual)

```bash
./start.sh --foreground
```

`--foreground` keeps server in foreground (needed for systemd/supervisord or background terminal sessions).

### Systemd Service (Auto-Start)

Create `~/.config/systemd/user/hermes-webui.service`:

```ini
[Unit]
Description=Hermes WebUI - Web Dashboard
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/home/$USER/apps/hermes-webui/start.sh
WorkingDirectory=/home/$USER/apps/hermes-webui
Restart=always
RestartSec=5
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

Then enable & start:

```bash
systemctl --user daemon-reload
systemctl --user enable hermes-webui.service
systemctl --user start hermes-webui.service
```

**PITFALL:** `start.sh` uses `bootstrap.py --no-browser` which does NOT need `--foreground` — the systemd service manages detach. The service file above omits `--foreground`; `start.sh` already handles the no-browser+detach via its own logic.

**PITFALL:** WebUI binds `0.0.0.0:8787` by default via `.env`. With no password set, it warns "Anyone on the network can access your filesystem and agent" — a benign warning on LAN/WSL unless the port is exposed to the internet. Suppress with `HERMES_WEBUI_PASSWORD` or bind to `127.0.0.1`.

### Verification

```bash
curl http://127.0.0.1:8787/health
# Expect: {"status": "ok", ...}
```

Also verify via systemd:

```bash
systemctl --user status hermes-webui.service
# Expect: active (running), listening on 0.0.0.0:8787
ss -tlnp | grep 8787
# Expect: LISTEN ... 0.0.0.0:8787 ... python3
```

### Logs

```bash
journalctl --user -u hermes-webui -f
```

PITFALL: Re-running `start.sh` while server already up detects it via /health probe and exits silently — use `./ctl.sh restart` instead.

## Hermes Dashboard (systemd)

The official Hermes Dashboard runs `hermes dashboard` on port **9119** — a FastAPI/uvicorn web server for managing config, API keys, sessions, and an embedded chat TUI via PTY/WebSocket.

### Service File

Create `~/.config/systemd/user/hermes-dashboard.service`:

```ini
[Unit]
Description=Hermes Dashboard - Web Management Server (port 9119)
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/path/to/hermes-agent/venv/bin/python3 -m hermes_cli.main dashboard --port 9119 --no-open --skip-build
WorkingDirectory=/path/to/hermes-agent
Restart=always
RestartSec=5
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

**Flags explained:**
- `--port 9119` default port; `0` for OS auto-assign
- `--no-open` suppress browser launch (headless service)
- `--skip-build` use pre-built web dist at `hermes_cli/web_dist/` — avoids needing npm at runtime
- Default host is `127.0.0.1` (localhost). Use `--insecure` + `--host 0.0.0.0` for network access (DANGEROUS without auth gate)

**Prerequisites:** `fastapi` + `uvicorn` must be installed in the Hermes venv (bundled by default). The web dist must exist at `hermes_cli/web_dist/index.html` (built via `npm run build -w web` from the Hermes repo root, or pre-shipped with the install).

### Enable & Start

```bash
systemctl --user daemon-reload
systemctl --user enable hermes-dashboard.service
systemctl --user start hermes-dashboard.service
```

### Verification

```bash
systemctl --user status hermes-dashboard.service
# Active: active (running)
# Logs show: HERMES_DASHBOARD_READY port=9119

ss -tlnp | grep 9119
# LISTEN ... 127.0.0.1:9119 ... hermes

curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:9119/
# 200
```

### PITFALL: Binding to 0.0.0.0 triggers auth gate
The dashboard refuses to bind to non-localhost unless `--insecure` is passed. Without a `DashboardAuthProvider` plugin installed the auth gate has no providers and startup fails with: `Refusing to bind dashboard to 0.0.0.0 — the OAuth auth gate engages on non-loopback binds, but no auth providers are registered`. Use `127.0.0.1` (default) or install the Nous dashboard-auth plugin + pass `--insecure`.

## OpenViking Memory Provider (systemd)

OpenViking (by Volcengine/ByteDance) is a local vector-based memory context DB. When used as Hermes' memory provider (`memory.provider: openviking`), it runs as its own server.

### Install

```bash
source ~/.hermes/hermes-agent/venv/bin/activate
pip install openviking llama-cpp-python
```

### Config (`~/.openviking/ov.conf`)

```json
{
  "server": {
    "host": "127.0.0.1",
    "port": 1933,
    "auth_mode": "dev"
  },
  "storage": {
    "workspace": "~/.openviking/data",
    "vectordb": {
      "backend": "local"
    }
  },
  "embedding": {
    "dense": {
      "provider": "local",
      "model": "bge-small-zh-v1.5-f16",
      "dimension": 512
    }
  },
  "vlm": {
    "provider": "openai",
    "model": "agent",
    "api_base": "https://ai.jefripunza.com/v1",
    "api_key": "${OPENAI_API_KEY}",
    "extra_headers": {
      "X-Title": "Vikingbot-Hermes"
    }
  }
}
```

**Key choices:**
- `auth_mode: dev` — no API key needed, localhost-only binding enforced
- `embedding.provider: local` — uses `llama-cpp-python` to run GGUF locally, no API key required
- Model `bge-small-zh-v1.5-f16` auto-downloads from HuggingFace on first use (~33MB, dimension 512)
- Data stored under `~/.openviking/data/`
- `vlm` section — **required when `--with-bot` is used.** vikingbot uses this for chat agent responses. Uses `os.path.expandvars` to resolve `${VAR}` placeholders from environment.
- `vlm.provider: "openai"` — works with any OpenAI-compatible API (OpenRouter, custom proxy, etc.)
- `vlm.api_key: "${OPENAI_API_KEY}"` — expanded at runtime from env var. The systemd service must pass this env var (see Service File below).

**PITFALL:** The local embedder requires `llama-cpp-python`. Without it, embedding requests fail silently.

**PITFALL:** Config is JSON, not YAML. Use valid JSON syntax or the config parser errors with `Unknown config field`.

**PITFALL:** `vectordb` goes inside the `storage` section, not at the top level.

**PITFALL:** vlm `api_key` uses `${OPENAI_API_KEY}` env var expansion (Go template syntax). If the env var is not set in the systemd service context, `${OPENAI_API_KEY}` expands to empty string → vikingbot's chat calls fail with auth errors. Add `Environment=OPENAI_API_KEY=sk-...` to the service file. Do NOT use `.env` file paths in ov.conf — `os.path.expandvars` only reads actual env vars, not dotenv files.

### Service File

Create `~/.config/systemd/user/hermes-openviking.service`:

```ini
[Unit]
Description=OpenViking - Memory Context Database Server (port 1933, bot mode)
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
Environment=OPENAI_API_KEY=sk-your-api-key-here
ExecStart=/path/to/hermes-agent/venv/bin/openviking-server --config /home/$USER/.openviking/ov.conf --with-bot
WorkingDirectory=/home/$USER/.openviking
Restart=always
RestartSec=5
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

**PITFALL:** The `--with-bot` flag enables vikingbot gateway (Agent chat / bot mode). Without it, OpenViking returns "Bot service not enabled" when chat features are accessed. Bot API proxy runs on port 18790 internally.

**PITFALL:** The `--with-bot` flag spawns a vikingbot subprocess that reads the `vlm` section from ov.conf. Without a valid `vlm` config, vikingbot starts but chat responses fail silently (auth errors or empty responses). The `Environment=OPENAI_API_KEY=...` line must be present in the service for the `${OPENAI_API_KEY}` placeholder to resolve.

> **Model naming:** When using an OpenRouter-compatible proxy (9ROUTER, etc.), the model name is whatever the proxy defines — for example `"agent"` custom alias. No `/org/model` delimiter is required unless the API endpoint specifically needs it.

```bash
systemctl --user daemon-reload
systemctl --user enable hermes-openviking.service
systemctl --user start hermes-openviking.service
```

### Verification

```bash
systemctl --user status hermes-openviking.service
# Active: active (running)

ss -tlnp | grep 1933
# LISTEN ... 127.0.0.1:1933 ... openviking-server

curl -s http://127.0.0.1:1933/health
# {"status":"ok","healthy":true,"version":"...","auth_mode":"dev"}

# Check bot mode activated:
journalctl --user -u hermes-openviking --no-pager | grep "vikingbot"
# Expect: "Bot API proxy enabled" and "Vikingbot gateway started"
```

### Hermes Config

```bash
hermes config set memory.provider openviking
echo "OPENVIKING_ENDPOINT=http://127.0.0.1:1933" >> ~/.hermes/.env
```

Changes take effect on `/reset` (new session).

## Cloudflare Tunnel (cloudflared)

Expose Hermes services to the internet via Cloudflare Tunnel — no open ports, no public IP needed.

### Install (User-Space, No Sudo)

```bash
mkdir -p ~/.local/bin
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 \
  -o ~/.local/bin/cloudflared
chmod +x ~/.local/bin/cloudflared
```

**PITFALL:** Do NOT use `sudo apt install` or `sudo dpkg` — user-space install avoids WSL sudo auth issues and keeps cloudflared in `~/.local/bin`.

### Create Tunnel (With Token)

Get tunnel token from Cloudflare Zero Trust Dashboard → Access → Tunnels → Create or configure existing tunnel. Token is a base64-encoded JSON (contains `a`=account, `t`=tunnel-id, `s`=secret).

Save token to restricted file:

```bash
echo '<token>' > ~/.cloudflared/tunnel-token
chmod 600 ~/.cloudflared/tunnel-token
```

### Wrapper Script

Create `~/.cloudflared/run.sh`:

```bash
#!/bin/bash
exec /home/$USER/.local/bin/cloudflared tunnel run --token "$(cat /home/$USER/.cloudflared/tunnel-token)"
```

```bash
chmod +x ~/.cloudflared/run.sh
```

### Systemd Service

Create `~/.config/systemd/user/cloudflared.service`:

```ini
[Unit]
Description=Cloudflare Tunnel - Hermes Services
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/home/$USER/.cloudflared/run.sh
Restart=always
RestartSec=5
KillMode=process
KillSignal=SIGTERM
TimeoutStopSec=30
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

Enable & start:

```bash
systemctl --user daemon-reload
systemctl --user enable cloudflared.service
systemctl --user start cloudflared.service
```

### Verification

```bash
systemctl --user status cloudflared.service
# Active: active (running)

journalctl --user -u cloudflared -n 20 --no-pager
# Expect: "Registered tunnel connection" with location=sinXX
```

### Configure Hostnames (Cloudflare Dashboard)

Routing is NOT set in local config — hostnames are configured in **Cloudflare Zero Trust Dashboard** → Access → Tunnels → select tunnel → **Public Hostnames** tab. Typical mappings:

| Hostname | Service |
|----------|---------|
| `webui.yourdomain.com` | `http://localhost:8787` |
| `dashboard.yourdomain.com` | `http://localhost:9119` |

**PITFALL:** The tunnel token alone just connects to Cloudflare edge. Without public hostname config in the dashboard, the tunnel connects but routes nothing.

**PITFALL:** Secret redaction in shell `echo` can truncate the token — use `write_file` or a heredoc to persist the full base64 token. Verify with `wc -c ~/.cloudflared/tunnel-token` (expect ~185 bytes for a standard token).

### Logs

```bash
journalctl --user -u cloudflared -f
```

Common connection locations: `sin14` (Singapore), `cgk02`/`cgk07` (Jakarta), `sin12`.

## Go Auth Proxy (Reverse Proxy + Basic Auth)

Add HTTP Basic Auth to any Hermes service using a lightweight Go reverse proxy. Auth credentials read from a JSON file — edit the file, proxy auto-reloads. No restart needed.

### Use Case

Put this in front of the WebUI (or Dashboard, OpenViking, etc) when:
- The WebUI is bound to `0.0.0.0` (accessible on LAN) and needs a password
- You want auth separate from the WebUI's built-in password
- Cloudflare Tunnel points to the proxy instead of the WebUI directly (adds auth layer before Cloudflare)

### Build & Directory Structure

```bash
mkdir -p ~/apps/hermes-proxy
```

Create `~/apps/hermes-proxy/auth.json`:

```json
{
  "username": "admin",
  "password": "your-password"
}
```

Create `~/apps/hermes-proxy/main.go` — see `references/hermes-auth-proxy.md` for the complete Go source.

Compile (stripped binary for minimal size):

```bash
cd ~/apps/hermes-proxy
go build -o hermes-proxy -ldflags="-s -w" main.go
# Produces ~6MB binary
```

### Binary CLI (Flags)

```
./hermes-proxy --addr :8888 --target http://127.0.0.1:8787 --auth /path/to/auth.json
```

| Flag | Default | Description |
|------|---------|-------------|
| `--addr` | `:8888` | Listen address |
| `--target` | `http://127.0.0.1:8787` | Backend URL to proxy to |
| `--auth` | `auth.json` in binary dir | Auth credentials JSON file |

When `--auth` is omitted, looks for `auth.json` in the same directory as the binary.

### Multiple Instances

Run one binary per backend service with different `--addr`/`--target`/`--auth`:

```
~/apps/hermes-proxy/hermes-proxy --addr :8888 --target http://127.0.0.1:8787 --auth auth-webui.json
~/apps/hermes-proxy/hermes-proxy --addr :8889 --target http://127.0.0.1:1933 --auth auth-openviking.json
```

Each with its own systemd service.

### Systemd Service (WebUI)

Create `~/.config/systemd/user/hermes-proxy.service`:

```ini
[Unit]
Description=Hermes Auth Proxy - port 8888 -> 8787 (Basic Auth)
After=network-online.target hermes-webui.service
Wants=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/home/$USER/apps/hermes-proxy/hermes-proxy --addr :8888 --target http://127.0.0.1:8787
WorkingDirectory=/home/$USER/apps/hermes-proxy
Restart=always
RestartSec=5
KillMode=process
TimeoutStopSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

**Key detail:** `WorkingDirectory` must match the binary directory so the default `auth.json` lookup works.

### Systemd Service (OpenViking)

Create `~/.config/systemd/user/hermes-proxy-openviking.service`:

```ini
[Unit]
Description=Hermes Auth Proxy - OpenViking port 8889 -> 1933
After=network-online.target hermes-openviking.service
Wants=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/home/$USER/apps/hermes-proxy/hermes-proxy --addr :8889 --target http://127.0.0.1:1933 --auth /home/$USER/apps/hermes-proxy/auth-openviking.json
WorkingDirectory=/home/$USER/apps/hermes-proxy
Restart=always
RestartSec=5
KillMode=process
TimeoutStopSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

### Enable & Start

```bash
systemctl --user daemon-reload
systemctl --user enable hermes-proxy.service
systemctl --user start hermes-proxy.service
# For OpenViking instance:
systemctl --user enable hermes-proxy-openviking.service
systemctl --user start hermes-proxy-openviking.service
```

### Verification

```bash
# Without auth — must 401
curl -sv http://127.0.0.1:8888/ 2>&1 | grep "< HTTP"
# HTTP/1.1 401 Unauthorized

# With auth — must 200
curl -sv -u admin:your-password http://127.0.0.1:8888/ 2>&1 | grep "< HTTP"
# HTTP/1.1 200 OK
```

### Hot-Reload Auth (No Restart)

Edit `auth.json` — proxy detects file modification time change on every request and reloads transparently:

```json
{ "username": "admin", "password": "new-password" }
```

Old password stops working immediately; new password accepted next request.

### CORS/CSRF Fix: Host Header Preservation

The proxy MUST preserve the original `Host` header so the Hermes WebUI can validate `Origin` against `Host` (CSRF check). The old proxy code had:

```go
// BUG: overwrites Host header → CSRF failure
r.Host = p.target.Host
```

The fixed proxy uses a custom Director:

```go
proxy.Director = func(req *http.Request) {
    origDirector(req)         // rewrites URL to target
    req.Host = origHost       // preserves real domain for CSRF check
    req.Header.Set("Host", origHost)
}
```

The proxy also sets `X-Forwarded-Host`, `X-Real-Host`, and `X-Forwarded-Proto` headers so the WebUI's `HERMES_WEBUI_TRUST_FORWARDED_HOST` env var works correctly.

**Hermes WebUI required env var (already set in service):**
```env
HERMES_WEBUI_TRUST_FORWARDED_HOST=1
```
Without this, the WebUI won't trust `X-Forwarded-Host`/`X-Real-Host` headers.

Without the Host preservation fix, requests through Cloudflare Tunnel → Go Proxy fail with:
```
Cross-origin mismatch - check reverse proxy headers
```

### How It Works

| Component | Detail |
|-----------|--------|
| Port | `--addr` (default `:8888`) |
| Backend | `--target` (default `http://127.0.0.1:8787`) |
| Auth file | `--auth` (defaults to `auth.json` in binary dir), hot-reloaded on file mtime change |
| Auth scheme | HTTP Basic Auth, constant-time comparison via SHA-256 + `subtle.ConstantTimeCompare` |
| Proxy | `httputil.NewSingleHostReverseProxy` from stdlib (handles WebSocket natively in Go 1.22+) |
| Headers | Preserves original `Host`, sets `X-Forwarded-Host`, `X-Real-Host`, `X-Forwarded-Proto`, `X-Forwarded-For` |
| Cloudflare | Detects `Cf-Visitor` header for `X-Forwarded-Proto`, strips internal headers to prevent double-hops |

### Rebuilding After Source Change

```bash
cd ~/apps/hermes-proxy
go build -o hermes-proxy -ldflags="-s -w" main.go
systemctl --user restart hermes-proxy.service
systemctl --user restart hermes-proxy-openviking.service  # if exists
```

**PITFALL:** The binary reads `auth.json` from its own directory using `os.Executable()` when `--auth` is not specified. If you run the binary from a different working directory (e.g. via systemd), ensure `WorkingDirectory` in the service file points to the same directory as the binary.

**PITFALL:** Old version used positional CLI args (`./hermes-proxy [backend_url] [listen_addr]`). Current version uses `--addr`/`--target`/`--auth` flags. Verify service files use the flag syntax.

**PITFALL:** Go must be installed — see prerequisites below for how to install user-space Go.

### Go Install (User-Space, No Sudo)

```bash
curl -fsSL https://go.dev/dl/go1.24.0.linux-amd64.tar.gz -o /tmp/go.tar.gz
tar -C ~/.local -xzf /tmp/go.tar.gz
mv ~/.local/go ~/.local/go1.24
ln -sf ~/.local/go1.24/bin/go ~/.local/bin/go
ln -sf ~/.local/go1.24/bin/gofmt ~/.local/bin/gofmt
go version
# go version go1.24.0 linux/amd64
```

## WSL-Specific Notes

- Gateway + WebUI + Dashboard all run as systemd user services, survive WSL restarts with `systemd=true` in `/etc/wsl.conf`
- Linger must be enabled: `sudo loginctl enable-linger $USER`
- WebUI systemd service: `~/.config/systemd/user/hermes-webui.service` (manual create — no `hermes` CLI command for it)
- Gateway systemd service: `hermes gateway install` sets it up automatically
- Dashboard systemd service: manual create (see section above)
- OpenViking systemd service: manual create (see section above)
- After WSL restart, verify all with:
  ```bash
  systemctl --user status hermes-gateway.service hermes-webui.service hermes-dashboard.service hermes-openviking.service cloudflared.service hermes-proxy.service
  ss -tlnp | grep -E '1933|8787|8888|9119'
  ```
- No special config beyond that — all services work the same as native Linux
