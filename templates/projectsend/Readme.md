# Projectsend Template

## Overview

This template proxies Projectsend through BunkerWeb with automatic Let's Encrypt certificates, reverse proxying, a 8192m request size limit and custom configuration files.

## Prerequisites

- A running Projectsend server that BunkerWeb can reach.
- A public hostname pointing to BunkerWeb.
- Access to the BunkerWeb web UI or service environment variables.

## Setup

1. Import `template.json` (Templates → **Create new template** → **Raw**, then upload or paste it).
2. When the **Missing Custom Configs** dialog appears, upload every file listed in the [Custom configs](#custom-configs) section from `configs/`, then save.
3. Assign `USE_TEMPLATE=projectsend` to the service, or select **Projectsend** in the web UI.
4. Replace `SERVER_NAME`, `EMAIL_LETS_ENCRYPT` and `REVERSE_PROXY_HOST` with values for your deployment.
5. Reload BunkerWeb, open the public URL, and test sign-in plus an upload and download.

## Placeholders

The values below are examples. Replace them with your own before using the template:

- `example.com` → your own domain name
- `admin@example.com` → your own email address
- `http://projectsend` → the name of the service's container on the shared Docker network, or its IP address and port, for example `http://10.0.0.5:8080`
- `replace-with-your-crowdsec-api-key` → the API key generated in CrowdSec (for example with `cscli bouncers add <name>`)
- `example-user` → your own username
- `replace-with-a-long-random-value` → your own long random secret

## Notes

- **Certificates:** the HTTP challenge needs port 80 on BunkerWeb to be reachable from the internet for every domain in `SERVER_NAME`.
- **Upstream:** `http://projectsend` is a container name. It only resolves when BunkerWeb and the application share a Docker network; otherwise replace it with the IP address and port of the service, for example `http://10.0.0.5:8080`. Add the port if the application does not listen on 80.
- **Uploads:** `MAX_CLIENT_SIZE=8192m` limits each request, not the total size of a file sent by a chunked client. Raise it only if clients need larger single requests.
- **Methods:** only the listed HTTP methods are accepted; anything else is rejected before reaching the application.
- **Custom configs (`modsec-crs`):** ModSecurity rules loaded before the OWASP Core Rule Set (typically rule exclusions / false-positive fixes).

## Settings

### Web service - Front door

Configure server / core settings.

| Setting | Value | Description |
| --- | --- | --- |
| `SERVER_NAME` | `projectsend.example.com` | Domain name(s) this service answers for. |

### SSL / Let's Encrypt

Configure ssl / let's encrypt settings.

| Setting | Value | Description |
| --- | --- | --- |
| `AUTO_LETS_ENCRYPT` | `yes` | Obtain and renew Let's Encrypt certificates automatically. |
| `EMAIL_LETS_ENCRYPT` | `admin@example.com` | Contact email registered with Let's Encrypt. |

### Reverse Proxy

Configure reverse proxy settings.

| Setting | Value | Description |
| --- | --- | --- |
| `USE_REVERSE_PROXY` | `yes` | Forward requests to an upstream application. |
| `REVERSE_PROXY_INTERCEPT_ERRORS` | `no` | — |
| `REVERSE_PROXY_HOST` | `http://projectsend` | Upstream application: container name, or IP address and port. |
| `REVERSE_PROXY_BUFFERING` | `no` | — |
| `REVERSE_PROXY_REQUEST_BUFFERING` | `no` | — |
| `REVERSE_PROXY_READ_TIMEOUT` | `3600s` | — |
| `REVERSE_PROXY_SEND_TIMEOUT` | `3600s` | — |

### Rate Limiting

Configure rate limiting settings.

| Setting | Value | Description |
| --- | --- | --- |
| `LIMIT_CONN_MAX_HTTP1` | `50` | — |
| `LIMIT_CONN_MAX_HTTP2` | `500` | — |
| `LIMIT_CONN_MAX_HTTP3` | `500` | — |
| `LIMIT_REQ_RATE` | `25r/s` | Allowed request rate for the matching URL (for example 15r/s). |

### Security

Configure security settings.

| Setting | Value | Description |
| --- | --- | --- |
| `USE_ANTIBOT` | `javascript` | Challenge visitors to block bots. |
| `ANTIBOT_IGNORE_URI` | `^/build/ ^/api/ ^/share/ ^/uploads ^/zip-downloads` | — |
| `BAD_BEHAVIOR_THRESHOLD` | `20` | — |
| `USE_CROWDSEC` | `yes` | Enable the CrowdSec integration. |
| `CROWDSEC_API` | `http://127.0.0.1:8080` | — |
| `CROWDSEC_API_KEY` | `replace-with-your-crowdsec-api-key` | API key generated in CrowdSec for the BunkerWeb bouncer. |

### Performance

Configure performance settings.

| Setting | Value | Description |
| --- | --- | --- |
| `CLIENT_CACHE_ETAG` | `no` | — |
| `GZIP_PROXIED` | `expired no-cache no-store private auth` | — |
| `MAX_CLIENT_SIZE` | `8192m` | Maximum size of a single request body. |
| `SERVE_FILES` | `no` | Serve static files from BunkerWeb itself (off when proxying an app). |

### Whitelist / Allowlist

Configure whitelist / allowlist settings.

| Setting | Value | Description |
| --- | --- | --- |
| `WHITELIST_IP` | `194.163.173.229 57.129.118.35 57.129.118.37` | — |

### Other

Configure other settings.

| Setting | Value | Description |
| --- | --- | --- |
| `KEEP_UPSTREAM_HEADERS` | `Content-Security-Policy Strict-Transport-Security X-Frame-Options X-Content-Type-Options Referrer-Policy` | — |
| `PROXY_BUFFERS` | `8 32k` | — |
| `PROXY_BUFFER_SIZE` | `32k` | — |
| `PROXY_BUSY_BUFFERS_SIZE` | `64k` | — |
| `USE_ROBOTSTXT` | `yes` | — |

### Fale Positives - False Positives Exclusion config

Modsec CRS Config file to prevent false positives.

Custom configs: `modsec-crs/modsec-crs-exclude-projectsend.conf`

### Other settings

Additional settings for this service.

| Setting | Value | Description |
| --- | --- | --- |
| `CROWDSEC_APPSEC_URL` | `http://127.0.0.1:7422/` | — |
| `INTERCEPTED_ERROR_CODES` | `404 405 413 429 500 501 502 503 504` | — |
| `CONTENT_SECURITY_POLICY_REPORT_ONLY` | `yes` | — |
| `COOKIE_FLAGS_2` | `projectsend_xsrf SameSite=Lax` | Flags (HttpOnly, Secure, SameSite…) added to cookies. |
| `ALLOWED_METHODS` | `GET\|POST\|HEAD\|PUT\|DELETE\|PATCH\|OPTIONS` | HTTP methods that are accepted; all others are rejected. |

## Custom configs

Files are referenced relative to the template's `configs/` directory and grouped by BunkerWeb config type.

| Type | File in repository | Reference in `template.json` | Purpose |
| --- | --- | --- | --- |
| `modsec-crs` | `templates/projectsend/configs/modsec-crs/modsec-crs-exclude-projectsend.conf` | `modsec-crs/modsec-crs-exclude-projectsend.conf` | ModSecurity rules loaded before the OWASP Core Rule Set (typically rule exclusions / false-positive fixes). |

## Docker Compose example

Usernames and passwords below are placeholders. Replace them with your own values before deploying.

```yaml
services:
  app:
    image: projectsend/projectsend:latest
    container_name: projectsend-app
    restart: unless-stopped
    ports:
      - "9999:80"                                # Maps host port 8080 to container port 80
    environment:
      # -- App --
      APP_URL: "https://filo.example.com"     # ASSUMPTION: pick the real hostname (matches SERVER_NAME below)
      TZ: "Europe/Berlin"

      # -- Tells ProjectSend a reverse proxy (BunkerWeb) sits in front. --
      # Safe as "*" here because only BunkerWeb can reach this container's
      # port on the bw-services network below — nobody else can connect
      # directly and spoof X-Forwarded-For.
      TRUSTED_PROXIES: "*"

      # -- Database --
      DB_HOST: "db"
      DB_NAME: "projectsend"
      DB_USER: "example-user"
      DB_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD}"
      DB_ROOT_PASSWORD: "${DB_ROOT_PASSWORD:?Set DB_ROOT_PASSWORD}"

      # -- Redis (sessions, cache, queue) --
      REDIS_HOST: "redis"
    volumes:
      # Bind-mounted per the persistence doc: this ONE mount carries both
      # the uploaded files AND storage/.env (APP_KEY). Do not mount just
      # storage/app/files/ — that leaves APP_KEY inside the container.
      - ./projectsend-default.conf:/etc/nginx/http.d/default.conf:ro
      - /srv/projectsend/storage:/var/www/html/storage
    depends_on:
      - db
      - redis

  db:
    image: mysql:8.0
    container_name: projectsend-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD:?Set MYSQL_ROOT_PASSWORD}"
      MYSQL_DATABASE: "projectsend"
      MYSQL_USER: "example-user"
      MYSQL_PASSWORD: "${MYSQL_PASSWORD:?Set MYSQL_PASSWORD}"
    volumes:
      - /srv/projectsend/mysql:/var/lib/mysql

  redis:
    image: redis:7-alpine
    container_name: projectsend-redis
    restart: unless-stopped
```

Set the required environment variables before starting the stack:

```bash
export DB_PASSWORD='replace-with-a-long-random-value'
export DB_ROOT_PASSWORD='replace-with-a-long-random-value'
export MYSQL_ROOT_PASSWORD='replace-with-a-long-random-value'
export MYSQL_PASSWORD='replace-with-a-long-random-value'
docker compose config
docker compose up -d
```

## Validation

Before importing the template:

```bash
jq . template.json
jq -e --arg expected "$(basename "$PWD")" '.id == $expected' template.json
jq -e '(.settings | keys | sort) == ([.steps[] | .settings[]] | sort)' template.json
for f in $(jq -r '.configs[]' template.json); do test -f "configs/$f" || echo "missing: $f"; done
```

After applying it:

- Confirm the BunkerWeb scheduler reload completes without a template or unknown-setting error.
- Sign in and test an upload and download.
- Check BunkerWeb logs for rate-limit, bad-behavior, and ModSecurity events before changing security controls.


## Additional information
For uploads above 9MB you'll need to remove ```env CROWDSEC_APPSEC_URL=http://127.0.0.1:7422/ ``` along with increasing the `MAX_CLIENT_SIZE` but that is on your own risk and you should know what you do.
