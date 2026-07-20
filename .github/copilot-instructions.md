# BunkerWeb Template Hub — Coding Agent Guide

## Mission Snapshot

- Deliver ready-to-import BunkerWeb templates that bundle `template.json` plus any referenced configs under `templates/<service>/`.
- Keep assets GPL-3.0 compatible and self-contained; do not depend on files outside the template directory.
- Favor clarity for BunkerWeb operators: document prerequisites and tuning inside service-specific `README.md` files when non-obvious.

## Repository Layout & Patterns

- Each source template contains `template.json`, optional `configs/` subfolders (e.g. `configs/modsec/`), and optional docs. Plugin packaging uses the layout documented in `README.md`.
- Existing example: `templates/wordpress/` defines core settings, guided steps, and ModSecurity snippets; reuse its structure for new services.
- JSON uses two-space indentation, double-quoted keys/strings, and avoids trailing commas (enforced by `pre-commit`).
- NGINX snippets in `configs/` should include concise comments when relaxing security defaults, following `wordpress_false_positives.conf` as a model.

## Template Authoring Expectations

- `template.json` keys must match BunkerWeb multisite setting names exactly (`SERVER_NAME`, `REVERSE_PROXY_HOST`, etc.); invalid keys break template import.
- Use the `steps` array to group related settings with user-facing `title`/`subtitle` guidance; copy the WordPress template’s tone to keep instructions actionable.
- All paths in the JSON `configs` arrays are relative to the template's `configs/` directory; ensure referenced files exist and ship with the PR.
- Provide sensible defaults (e.g. `AUTO_LETS_ENCRYPT=yes`, rate limits) while highlighting what users must override in the template README or step subtitles.

## Validation & Tooling

- Run `pre-commit install` once, then `pre-commit run --all-files` before committing; this repo relies on Prettier, Codespell, and Gitleaks.
- Validate JSON definitions locally (`jq . templates/<service>/template.json`) prior to submission; CI expects well-formed JSON.
- For new ModSecurity or NGINX snippets, validate them in the matching BunkerWeb configuration context where possible and document any assumptions.

## Contribution Workflow Notes

- Branch from `dev`; target PRs against `dev` with a clear summary plus validation notes (see `CONTRIBUTING.md`).
- Keep directory names lowercase-kebab-case and limit filenames to ASCII.
- Mention testing environment versions when they materially affect validation.
- Update `CHANGELOG.md` under `## Unreleased` for every change using the bullet format `- [@github-handle] Summary of the change` so manual releases stay accurate and attribution remains clear.

## External References

- Use the official docs for canonical guidance: [BunkerWeb templates documentation](https://docs.bunkerweb.io/latest/concepts/#templates) when explaining settings or template behavior.
- Direct contributors to the community Discord template channel for collaboration when work-in-progress feedback is helpful.
