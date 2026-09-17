# Seafile Template

## Overview

Provision a BunkerWeb configuration tailored for Jellyfin so HTTPS automation,
reverse proxy buffering, websocket upgrades, payload limits, and CRS exclusions
match typical media streaming workloads without sacrificing security headers.

## Prerequisites

- A setup Seafile instance example docker compose can be found at the bottom.
- Access to the BunkerWeb UI or environment variables to assign template
  settings.
- Confirm Jellyfin trusts the proxy IP which can be found below.

## Files

- `template.json` – Template definition with steps for TLS, upstreams, and
  header tuning.
- `configs/modsec-crs/seafile_false_positives.conf` – Removes CRS rules that
  interfere with the Seafile uploading.

## Setup

1. **Import the template**
   - Follow the repository's [installation guide](../../README.md#installing-templates)
     for the web UI or plugin bundle method.
2. **Assign the template** to the site serving Seafile (`USE_TEMPLATE=seafile`
   or choose it in the UI).
3. **Adjust TLS automation** so `SERVER_NAME` and certificate options reflect
   the domains you expose.
4. **Update upstream targets**: point `REVERSE_PROXY_HOST` (and the websocket
   entry) to your Seafile service and confirm connectivity from BunkerWeb.
5. **Review streaming limits**: keep `REVERSE_PROXY_BUFFERING=no` and the
   elevated timeouts unless your deployment has specific limits.
6. **Reload BunkerWeb** and upload a file to ensure websocket connection is work through the proxy.

7. **Post docker compose up -d instruction below the Docker Compose Example area.


## Customization Tips

- Raise `MAX_CLIENT_SIZE` if you proxy uploads larger than 20 MiB (for example for larger file uploads).
- Keep `REVERSE_PROXY_WS_1=yes` for `/socket` so Jellyfin websockets upgrade
  correctly.
- Adjust `LIMIT_REQ_RATE` to match your concurrent stream expectations; set it
  higher for multi-user households.
- Modify `CONTENT_SECURITY_POLICY` and `PERMISSIONS_POLICY` only if you embed
  Seafile in another application and need additional origins.
- Edit `configs/modsec-crs/jellyfin_false_positives.conf` if future CRS updates
  require different rule IDs.

## Docker Compose Example

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: seafile-mysql
    environment:
      - MYSQL_ROOT_PASSWORD=db_dev  # Required, set the root's password of MySQL service.
      - MYSQL_LOG_CONSOLE=true
      - MARIADB_AUTO_UPGRADE=1
    volumes:
      - /opt/seafile-mysql/db:/var/lib/mysql  # Required, specifies the path to MySQL data persistent store.
    networks:
      - seafile-net

  memcached:
    image: memcached:1.6.18
    container_name: seafile-memcached
    entrypoint: memcached -m 256
    networks:
      - seafile-net

  seafile:
    image: seafile-mc-s3:11.0-latest
    container_name: seafile
    ports:
      - "7841:80"
#     - "443:443"  # If https is enabled, cancel the comment.
    volumes:
      - /opt/seafile-data:/shared   # Required, specifies the path to Seafile data persistent store.
    environment:
      - DB_HOST=db
      - DB_ROOT_PASSWD=db_dev  # Required, the value should be root's password of MySQL service.
      - TIME_ZONE=Etc/UTC  # Optional, default is UTC. Should be uncomment and set to your local time zone.
      - SEAFILE_ADMIN_EMAIL=admin@domain.com # Specifies Seafile admin user, default is 'me@example.com'.
      - SEAFILE_ADMIN_PASSWORD=password     # Specifies Seafile admin password, default is 'asecret'.
      - SEAFILE_SERVER_LETSENCRYPT=false   # Whether to use https or not.
      - SEAFILE_SERVER_HOSTNAME=domain.com # Specifies your host name if https is enabled.
    depends_on:
      - db
      - memcached
    networks:
      - seafile-net

networks:
  seafile-net:
```

## Additional Tweaks required to get Seafile to trust the reverse proxy.

After the first time running docker compose up -d do the following:

nano /opt/seafile-data/seafile/conf/seahub_settings.py

Check for ```FILE_SERVER_ROOT = "https://domain.com/seafhttp"```
The url should contain https not http.

Add at the bottom:
```yaml
CSRF_TRUSTED_ORIGINS = ['https://domain.com/']
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```
Don't forgot changing domain.com to your desired domain.

## Validation
