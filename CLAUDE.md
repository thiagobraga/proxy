# AI Instructions for proxy

## Common commands
- Project needs a proxy network: `docker network create proxy`
- Generate Certs: `docker compose run --rm mkcert`
- Start (detached): `docker compose up -d`
- Stop (fast): `docker compose down --timeout 0`
- Logs: `docker compose logs -f --tail=200`

## General instructions
- Keep answers short.
- When asked to restart the project: `docker compose down --timeout 0` then `docker compose up -d`.
