# Drupal Template

## Overview

Generate a Drupal-focused BunkerWeb configuration that keeps TLS automation, sane
reverse proxy defaults, installer rate limiting, and ModSecurity CRS exclusions
aligned with the CMS so logins, content editing, and cache management remain
reliable.

## Prerequisites

- A reachable Drupal application (container, VM, or bare metal) that trusts the
  BunkerWeb proxy IP.
- Access to the BunkerWeb UI or environment variables to apply template
  settings.
- Updated Drupal `settings.php` so `$settings['reverse_proxy'] = TRUE;` and the
  proxy address is listed in `$settings['reverse_proxy_addresses']`.

## Files

- `template.json` – Template definition with guided steps for TLS, upstream, and
  hardening options.

## Setup

1. **Import the template**
   - Follow the repository's [installation guide](../../README.md#installing-templates)
     for the web UI or plugin bundle method.
2. **Assign the template** to the service that fronts Drupal (`USE_TEMPLATE=drupal`
   via environment variables or pick it in the UI).
3. **Review TLS automation** and ensure `SERVER_NAME` and certificate options
   reflect your public hostname(s).
4. **Point the reverse proxy** at the Drupal backend (for example
   `http://drupal:8080`) and confirm BunkerWeb can resolve the target.
5. **Keep the installer rate limits** on `/core/install.php` unless you are
   performing automated installs.
6. **Reload BunkerWeb** and verify you can browse the Drupal admin dashboard
   through the proxy.

## Customization Tips

- Set `REVERSE_PROXY_HOST` to the internal scheme and port of your Drupal
  service.
- Increase `LIMIT_REQ_RATE_*` temporarily if scripted migrations or updates need
  higher throughput.
- Leave `SERVE_FILES=no` so Drupal handles asset delivery; enable it only if you
  move static files to an object store behind BunkerWeb.
- `MODSECURITY_CRS_PLUGINS=drupal-rule-exclusions` keeps Drupal management
  actions functional—change it only if you tune CRS rules manually.

## Validation

- Run `jq . template.json` to confirm the template definition is valid JSON.
- Check the Drupal status report and attempt cache clears or content edits to
  ensure rate limits and CRS exclusions do not block normal operations.
