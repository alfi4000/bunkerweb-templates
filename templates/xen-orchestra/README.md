# Xen Orchestra Template

## Overview

Provision a BunkerWeb configuration tailored for Xen Orchestra so HTTPS
automation, reverse proxy, WebSocket upgrades, and JSON-RPC rate limiting work
out of the box. Bad behavior detection is disabled to avoid false positives
caused by XO's frequent API polling.

## Prerequisites

- A Xen Orchestra backend reachable from the BunkerWeb instance (typically
  HTTPS with a self-signed certificate).
- Access to the BunkerWeb UI or environment variables to assign template
  settings.
- Confirm XO listens on HTTPS and that `REVERSE_PROXY_SSL_SNI` is set to `no`
  when using a self-signed upstream certificate.

## Files

- `template.json` – Template definition with steps for TLS, reverse proxy,
  rate limiting, and security tuning.
- `configs/modsec/xen-orchestra_false_positives.conf` – CRS exclusions for
  the `/jsonrpc` endpoint to prevent false positives from JSON-RPC API traffic.

## Setup

1. **Import the template**
   - *UI import (recommended)*: open the BunkerWeb `Templates` page, click
     **Create new template**, switch to **Raw** mode, paste the contents of
     `template.json`, and save.
   - *Plugin bundle*: copy `xen-orchestra/` into your BunkerWeb plugin
     `templates/` directory.
2. **Assign the template** to the site serving Xen Orchestra
   (`USE_TEMPLATE=xen-orchestra` or choose it in the UI).
3. **Adjust TLS automation** so `SERVER_NAME` and certificate options reflect
   the domain you expose.
4. **Update upstream target**: point `REVERSE_PROXY_HOST` to your Xen Orchestra
   instance (e.g. `https://192.168.1.1`) and confirm connectivity from
   BunkerWeb.
5. **Review rate limits**: the `/jsonrpc` endpoint is given a higher rate
   (`15r/s`) to accommodate XO's API polling; adjust if needed.
6. **Reload BunkerWeb** and log into Xen Orchestra to verify dashboard access,
   VM console websockets, and live migration actions work through the proxy.

## Customization Tips

- Raise or lower `MAX_CLIENT_SIZE` depending on whether you transfer ISOs or
  large files through the proxy.
- Adjust `LIMIT_REQ_RATE_1` for `/jsonrpc` if you have many concurrent users
  or heavy automation scripts hitting the API.
- `USE_BAD_BEHAVIOR` is set to `no` because XO's polling pattern triggers false
  positives; re-enable only after confirming it does not block legitimate
  traffic.
- Modify `CONTENT_SECURITY_POLICY` and `PERMISSIONS_POLICY` if you embed XO in
  an iframe or need additional origins.
- Set `REVERSE_PROXY_SSL_SNI` to `yes` if your upstream uses a valid
  certificate matching the hostname.
- Edit `configs/modsec/xen-orchestra_false_positives.conf` if future CRS
  updates require different rule IDs.

## Validation

- Run `jq . template.json` to ensure the definition is valid JSON.
- Test a dashboard login, VM console session (websocket), and a live migration
  or snapshot action through BunkerWeb to confirm proxying, rate limits, and
  security settings behave as expected.
