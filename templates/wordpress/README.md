# WordPress Template

## Overview

Provision a curated BunkerWeb configuration for WordPress. This template ships sensible defaults for TLS, reverse proxying, request throttling, crawler whitelisting, and CRS exclusions tuned for WordPress traffic so you can get a secure site online quickly.

## REST API PUT/DELETE disabled by default

The default `ALLOWED_METHODS` value permits `GET`, `POST`, `HEAD`, and `OPTIONS` only, so the WordPress REST API stays reachable for reads and `POST` writes. Clients that rely on `PUT` or `DELETE` verbs (for example some headless integrations or block-editor plugins) are blocked until you add those methods.

Default (`PUT`/`DELETE` blocked):

```text
"ALLOWED_METHODS": "GET|POST|HEAD|OPTIONS"
```

Updated (`PUT`/`DELETE` allowed):

```text
"ALLOWED_METHODS": "GET|POST|HEAD|OPTIONS|PUT|DELETE"
```

## Prerequisites

- A running WordPress upstream accessible on your network (for example a container named `mywp`).
- The BunkerWeb UI or the ability to edit multisite settings directly.
- Domain name(s) that will serve the WordPress instance.

## Files

- `template.json` – BunkerWeb template definition containing default settings, configs, and guided steps.
- `configs/modsec/wordpress_false_positives.conf` – ModSecurity CRS tuning for WordPress admin, cron, and XML-RPC traffic.

## Setup

1. **Import the template**
   - Follow the repository's [installation guide](../../README.md#installing-templates) for the web UI or
     plugin bundle method.
2. **Assign the template** to your WordPress service via the easy-mode UI or by setting `USE_TEMPLATE=wordpress`.
3. **Customize the settings** highlighted in the template steps (domains, upstream host, TLS options).
4. **Reload the service** and verify WordPress loads through BunkerWeb.

## Customization Tips

- Update `REVERSE_PROXY_HOST` to the URL of your WordPress upstream (e.g. `http://wordpress:80`).
- Adjust `MAX_CLIENT_SIZE` if you need to support larger media uploads.
- Tune `LIMIT_REQ_RATE` or change the protected `LIMIT_REQ_URL` (defaults to `/`) if you want to rate-limit only `wp-login.php` or `xmlrpc.php`.
- Add or prune domains in `WHITELIST_RDNS` if you want to control which crawlers bypass security checks.
- Review and narrow the `admin-ajax` exclusions in `configs/modsec/wordpress_false_positives.conf` for your installation.

## Validation

Run `jq . template.json` to confirm the JSON definition is valid before importing via the UI.
