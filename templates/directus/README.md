# Directus Template

## Overview

Generate a Directus-focused BunkerWeb configuration that is fine tunned for public api, requiring to make some tweaks heavily depends on the use case.

## Prerequisites

- A reachable Directus application (container, VM, or bare metal) that trusts the
  BunkerWeb proxy IP.
- Access to the BunkerWeb UI or environment variables to apply template
  settings.

## Setup

1. **Import the template**
   - Follow the repository's [installation guide](../../README.md#installing-templates)
     for the web UI or plugin bundle method.
2. **Assign the template** to the service that fronts Drupal (`USE_TEMPLATE=drupal`
   via environment variables or pick it in the UI).
3. **Review settings and make some tweaks if needed** and ensure `SERVER_NAME` reflect your public hostname(s).
4. **Point the reverse proxy** at the Directus backend (for example
   `http://directus:8055`) and confirm BunkerWeb can resolve the target.
5. **Reload BunkerWeb** and verify you can browse the Drectus admin dashboard
   through the proxy.

## Customization Tips

- Double check and modify if needed the Rate limits and Maximum body size to your needs.
- Do not enable Antibot as Directus resolves both admin dashboard and api on the same domain you can not reliably use the antibot feature bunkerweb provides(Directus has some security measures in place).
- You may want to double check the security features enabled at example.com/admin/settings/project.

## Validation

- Run `jq . template.json` to confirm the template definition is valid JSON.
- Sign in to the admin UI and exercise its API-backed dashboard through
  BunkerWeb to confirm routing, client IP attribution, and rate limits.

## Raw Config

```env
IS_DRAFT=no
SERVER_NAME=example.com
BAD_BEHAVIOR_THRESHOLD=20
CLIENT_CACHE_ETAG=no
USE_CORS=yes
CORS_ALLOW_METHODS=GET, POST, HEAD, PUT, DELETE, PATCH, OPTIONS
CORS_ALLOW_HEADERS=Content-Type,Authorization,X-Requested-With,Accept,Origin,Cache-Control
CORS_ALLOW_CREDENTIALS=yes
USE_CROWDSEC=yes
INTERCEPTED_ERROR_CODES=404 405 413 429 500 501 502 503 504
GZIP_PROXIED=expired no-cache no-store private auth
KEEP_UPSTREAM_HEADERS=Content-Security-Policy Strict-Transport-Security X-Frame-Options X-Content-Type-Options Referrer-Policy Access-Control-Allow-Origin Access-Control-Allow-Credentials Access-Control-Allow-Headers Access-Control-Allow-Methods Access-Control-Expose-Headers Access-Control-Max-Age
CONTENT_SECURITY_POLICY_REPORT_ONLY=yes
AUTO_LETS_ENCRYPT=yes
EMAIL_LETS_ENCRYPT=registration@example.com
LIMIT_CONN_MAX_HTTP1=50
LIMIT_CONN_MAX_HTTP2=500
LIMIT_CONN_MAX_HTTP3=500
LIMIT_REQ_RATE=25r/s
ALLOWED_METHODS=GET|POST|HEAD|PUT|DELETE|PATCH|OPTIONS
MAX_CLIENT_SIZE=1024m
SERVE_FILES=no
MODSECURITY_SEC_RULE_ENGINE=DetectionOnly
USE_REVERSE_PROXY=yes
REVERSE_PROXY_INTERCEPT_ERRORS=no
REVERSE_PROXY_HOST=http://directus:8055
REVERSE_PROXY_WS=yes
REVERSE_PROXY_BUFFERING=no
REVERSE_PROXY_REQUEST_BUFFERING=no
REVERSE_PROXY_READ_TIMEOUT=3600s
REVERSE_PROXY_SEND_TIMEOUT=3600s
USE_ROBOTSTXT=yes
```

## Disclaimer

Be aware we are not the Directus support therefore for questions please contact them through their official channels.
