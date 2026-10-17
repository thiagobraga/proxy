<div align="center">
  <img src="https://raw.githubusercontent.com/traefik/traefik/master/docs/content/assets/img/traefik.logo.png" alt="Traefik Logo" width="300">
  <h1>Proxy</h1>
</div>

A shared Traefik instance for local development using Docker Compose and HTTPS.

## 1. Prerequisites

- [**Docker**](https://docs.docker.com/engine/install/ubuntu/)
- [**Docker Compose plugin**](https://docs.docker.com/compose/install/linux/)

## 2. Installation

```bash
# Install dependencies and mkcert
sudo apt update
sudo apt install -y curl libnss3-tools
curl -L -o mkcert "https://dl.filippo.io/mkcert/latest?for=linux/amd64"
chmod +x mkcert
sudo mv mkcert /usr/local/bin/mkcert

# Trust the root CA
mkcert -install

# Generate certificates
mkcert \
  -cert-file certs/localhost.pem \
  -key-file certs/localhost-key.pem \
  127.0.0.1 localhost "*.local"

# Ensure the proxy Docker network exists
docker network create proxy 2>/dev/null || true

# Start Traefik
docker compose down --timeout=0
docker compose up -d
```

## 3. Usage

To connect another project to this Traefik instance:

1. Add `proxy` network to the project's `compose.yml`:
    ```yaml
    networks:
      proxy:
        external: true
    ```
2. Add labels to the service:
    ```yaml
    services:
      web:
        image: app
        networks:
          - proxy
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.app.rule=Host(`app.local`)"
          - "traefik.http.routers.app.entrypoints=websecure"
          - "traefik.http.routers.app.tls=true"
    ```
3. Add the host to `/etc/hosts` (or equivalent):
    ```
    127.0.0.1 app.local
    ```

## 4. Dashboard
- [http://localhost:8090](http://localhost:8090)

## 5. Optional: Generate certificates using Docker

```bash
docker run --rm \
  -v $(pwd)/certs:/certs \
  alpine/mkcert \
  -cert-file /certs/localhost.pem \
  -key-file /certs/localhost-key.pem \
  127.0.0.1 localhost "*.local"
```

Or using docker compose:
`compose.yml`
```yaml
mkcert:
  image: alpine/mkcert
  container_name: mkcert
  command: -cert-file /certs/localhost.pem -key-file /certs/localhost-key.pem localhost 127.0.0.1 "*.local"
  volumes: [./certs:/certs, "${HOME}/.local/share/mkcert:/root/.local/share/mkcert"]
```
Note that by doing this, you need to add certificates to browsers manually.

## 6. Troubleshooting
If your browser shows a "Not Secure" warning:
1. Ensure you ran `mkcert -install` on your host.
2. Restart your browser entirely.
3. Ensure you used the `websecure` (port 443) entrypoint in your project's labels.
