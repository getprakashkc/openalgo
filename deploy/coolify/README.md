# OpenAlgo on Coolify

One **Git source**, one **compose file**, many **Coolify applications** — each app is one broker + one user. All configuration lives in **Coolify → Environment** (no per-broker compose files, no mounted `.env`).

## Setup (once per broker/user)

1. **New resource** in Coolify → Docker Compose → Git repository `marketcalls/openalgo` (or your fork).

2. **Compose file path:** `deploy/coolify/docker-compose.yaml`

3. **Domain:** assign `https://your-subdomain.example.com` → container port **5000**.

4. **WebSocket:** route path **`/ws`** to container port **8765** (same domain), or use Coolify’s port mapping UI. Set `WEBSOCKET_URL=wss://your-subdomain.example.com/ws`.

5. **Environment:** copy keys from [`env.example`](./env.example). Required minimum:

   | Variable | Example |
   |----------|---------|
   | `OPENALGO_INSTANCE_NAME` | `kotak-alice` (unique label) |
   | `HOST_SERVER` | `https://kotak.example.com` |
   | `REDIRECT_URL` | `https://kotak.example.com/kotak/callback` |
   | `WEBSOCKET_URL` | `wss://kotak.example.com/ws` |
   | `CORS_ALLOWED_ORIGINS` | `https://kotak.example.com` |
   | `VALID_BROKERS` | `kotak` (one broker only) |
   | `BROKER_API_KEY` / `BROKER_API_SECRET` | From broker developer portal |
   | `APP_KEY` / `API_KEY_PEPPER` | New `secrets.token_hex(32)` each instance |

6. **Do not** bind-mount a host `.env` file. The image ships an empty `/app/.env`; on boot, `start.sh` writes a full `.env` from Coolify’s environment variables.

7. **Deploy.** Volumes (`openalgo_db`, etc.) are created per Coolify app — data stays isolated.

## Duplicate for N brokers

| Coolify app name | Domain | `VALID_BROKERS` | `OPENALGO_INSTANCE_NAME` |
|------------------|--------|-----------------|---------------------------|
| openalgo-kotak | kotak.example.com | `kotak` | `kotak` |
| openalgo-dhan | dhan.example.com | `dhan` | `dhan` |

Same repo and compose path every time; only env vars and domain change.

## Coolify magic URLs (optional, v4.411+)

After assigning a domain to service `openalgo`:

```env
HOST_SERVER=${SERVICE_URL_OPENALGO}
REDIRECT_URL=${SERVICE_URL_OPENALGO}/kotak/callback
WEBSOCKET_URL=${SERVICE_URL_OPENALGO_8765:-/ws}
```

Replace `kotak` in `REDIRECT_URL` with your broker id.

## Local Docker (not Coolify)

Use the root [`docker-compose.yaml`](../../docker-compose.yaml) with a local `.env` file, or set `HOST_SERVER` in the shell and use this compose file without mounting `.env`.

## Troubleshooting

- **502 / app not reachable:** `FLASK_HOST_IP` must be `0.0.0.0` (set in compose).
- **WebSocket fails:** confirm `/ws` → 8765 and `WEBSOCKET_URL` uses `wss://`.
- **OAuth callback fails:** `REDIRECT_URL` must match the broker portal exactly (HTTPS).
- **Login works on wrong instance:** reuse of `APP_KEY` or cookie names — use unique `APP_KEY`, `API_KEY_PEPPER`, and `OPENALGO_INSTANCE_NAME` per app.
