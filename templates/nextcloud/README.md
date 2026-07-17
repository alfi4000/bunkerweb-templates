# Nextcloud Template

## Overview

Generate a hardened BunkerWeb configuration for Nextcloud. The template keeps TLS automation, large file
uploads, WebDAV verbs, caching, and ModSecurity CRS exclusions aligned with common Nextcloud guidance so
your instance stays reachable and protected.

## Prerequisites

- A reachable Nextcloud backend (container, VM, or service) that accepts traffic from BunkerWeb.
- Access to the BunkerWeb UI or the ability to apply settings via environment variables.
- Updated `NEXTCLOUD_TRUSTED_DOMAINS` and `TRUSTED_PROXIES` inside the Nextcloud container or pod.

## Files

- `template.json` – Template definition with default settings and the guided step layout.
- `configs/modsec/nextcloud_false_positives.conf` – Enables CRS Nextcloud exclusions and allows required WebDAV verbs.

## Setup

1. **Import the template**
   - Follow the repository's [installation guide](../../README.md#installing-templates) for the web UI or
     plugin bundle method.
2. **Assign the template** to the service that fronts Nextcloud (set `USE_TEMPLATE=nextcloud` or pick it
   from the easy-mode UI).
3. **Adjust the TLS and domain entries** so `SERVER_NAME` and certificate options match your deployment.
4. **Point the reverse proxy** at your upstream host (for example `http://nextcloud:8080`) and confirm the
   service resolves from the BunkerWeb container or pod.
5. **Review the HTTP hardening options** – keep the WebDAV verbs, upload size, rate limits, caching, and
   CRS plugin values or tailor them to your workload.
6. **Check the Nextcloud Mail step** – if you use the Nextcloud Mail app, the **HTTP - Mail** step
   exposes dedicated settings you can adjust to fit your usage.
7. **Reload BunkerWeb** and verify you can sign in to Nextcloud through the proxy.

## Customization Tips

- Set `REVERSE_PROXY_HOST` to the correct scheme and port for your Nextcloud backend.
- Raise `MAX_CLIENT_SIZE` if you expect users to upload files larger than 10G.
- Narrow `LIMIT_REQ_URL_*` paths or loosen the `LIMIT_REQ_RATE_*` values to accommodate heavy clients such
  as the desktop sync agent.
- Keep `ALLOWED_METHODS` aligned with the WebDAV verbs required by Nextcloud; remove verbs only if you are
  sure the clients do not need them.
- `SERVE_FILES=no` ensures Nextcloud handles static assets; change it only if you delegate asset delivery
  elsewhere.
- Adjust `configs/modsec/nextcloud_false_positives.conf` if you must disable the CRS plugin or tailor the allowed
  methods for niche deployments.
- Set `REVERSE_PROXY_WS` to `"yes"` to enable WebSocket support, which is required by some Nextcloud
  applications (for example Talk or Collabora).

## Validation

- Run `jq . template.json` to confirm the template definition is valid JSON before importing.
- Exercise a large file upload and a sync client login after applying the template to confirm rate limits
  and headers do not block normal usage.
