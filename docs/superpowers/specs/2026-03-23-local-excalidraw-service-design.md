# Design: Persistent Local Excalidraw Service

**Date**: 2026-03-23
**Status**: Approved

## Goal

Run Excalidraw permanently on the local machine at `excalidraw.local` (port 80), with seamless switching between a production static build and a live Vite dev server for active development.

## Architecture

```
                        excalidraw.local:80
                              │
                           [nginx]
                           /      \
                     (prod)        (dev)
                      │               │
              static build        reverse proxy
          excalidraw-app/build/    → localhost:5173
                                       │
                                   [Vite HMR]
```

## Components

### 1. DNS: `/etc/hosts`

```
127.0.0.1 excalidraw.local
```

Static file, persists reboots natively. No DNS server or dnsmasq needed.

### 2. nginx (Homebrew)

Lightweight static server / reverse proxy. Two config files, one active at a time:

| Config | Behaviour |
|--------|-----------|
| `docs/nginx/prod.conf` | Serves `excalidraw-app/build/` as static files with SPA fallback |
| `docs/nginx/dev.conf` | Reverse proxies to Vite dev server on `localhost:5173`, including WebSocket for HMR |

The active config is symlinked into nginx's servers directory. Switching configs requires only an `nginx -s reload` — no restart, no downtime.

### 3. launchd plist

`~/Library/LaunchAgents/com.excalidraw.local.plist`

- Starts nginx on user login
- Defaults to prod mode (static build)
- Restarts on failure via `KeepAlive`
- No node process running in background by default

### 4. `excalidraw` CLI script

Installed to `/usr/local/bin/excalidraw` (or symlinked from `docs/excalidraw`).

| Command | Action |
|---------|--------|
| `excalidraw dev` | Start Vite dev server, swap nginx to proxy mode, reload |
| `excalidraw prod` | Stop Vite, rebuild production, swap nginx to static mode, reload |
| `excalidraw stop` | Stop Vite if running, stop nginx |
| `excalidraw status` | Show current mode and whether services are running |

### 5. Production build

`yarn build:app` outputs to `excalidraw-app/build/`. nginx serves this directory directly. No rebuild needed on boot — the last build persists on disk.

## Workflows

### Day-to-day usage (not developing)

Boot machine → launchd starts nginx → `excalidraw.local` serves production build. Zero intervention.

### Active development

```bash
excalidraw dev      # starts Vite, switches nginx to proxy mode
# code, save, see changes instantly at excalidraw.local
excalidraw prod     # done for the day — rebuilds, switches back to static
```

### After merging upstream changes

```bash
git pull
excalidraw prod     # rebuilds and serves new version
```

## nginx Config Details

### prod.conf

```nginx
server {
    listen 80;
    server_name excalidraw.local;

    root /Users/dwhitehouse/education/excalidraw/excalidraw-app/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml image/svg+xml;
}
```

### dev.conf

```nginx
server {
    listen 80;
    server_name excalidraw.local;

    location / {
        proxy_pass http://127.0.0.1:5173;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

WebSocket headers are required for Vite's HMR connection.

## Decisions

| Decision | Rationale |
|----------|-----------|
| No Docker | Static site + nginx uses ~2MB RAM vs Docker Desktop's ~2GB. Docker Desktop auto-start is unreliable on macOS. |
| No HTTPS | Localhost only, no security benefit. Avoids cert management complexity. |
| launchd over cron | macOS-native service manager. Supports restart-on-failure, runs at login (not just boot). |
| Symlink switching over nginx includes | Simpler, no conditional logic in config. One active config at a time. |
| Port 80 over 3000 | User preference — port 3000 conflicts with other dev servers. |
| Vite stays on 5173 | Default Vite port, only accessed via nginx proxy, no conflict. |

## File Inventory

```
docs/
  nginx/
    prod.conf           # nginx config for static build
    dev.conf            # nginx config for Vite proxy
  excalidraw            # CLI script (dev/prod/stop/status)
  rebuild.sh            # (removed — replaced by `excalidraw prod`)
~/Library/LaunchAgents/
  com.excalidraw.local.plist
/etc/hosts              # 127.0.0.1 excalidraw.local
/usr/local/bin/excalidraw  # symlink to docs/excalidraw
```

## Not In Scope

- Collaboration server (Firebase-dependent, not needed for local use)
- Custom domain TLS certificates
- Multiple environments (dev/staging/prod) — single machine, single purpose
- Automatic rebuild on git pull (manual `excalidraw prod` is sufficient)
