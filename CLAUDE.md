# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Community-curated collection of ready-to-import BunkerWeb security templates for popular services (WordPress, Drupal, Nextcloud, Jellyfin, Tomcat, NetBird). Each template bundles reverse proxy, TLS, rate limiting, and ModSecurity configurations. GPL-3.0 licensed.

## Development Commands

```bash
pre-commit install                        # Register git hooks (one-time setup)
pre-commit run --all-files                # Run full validation suite (Prettier, Codespell, Gitleaks)
jq . templates/<name>/template.json       # Validate template JSON syntax
```

No build step or automated test harness exists. Validation is manual: `docker compose config`, `kubectl apply --server-dry-run -f`, or `nginx -t -c <config>` for service-specific checks.

## Repository Architecture

**Template structure** — each template follows this layout:

```
templates/<service-name>/
├── template.json                    # BunkerWeb template definition (required)
├── README.md                        # Service-specific docs
└── configs/                         # Optional NGINX/ModSecurity snippets
    └── modsec/
        └── <service>_false_positives.conf
```

**template.json schema**:

- `id`: lowercase hyphenated identifier
- `name`: user-facing label
- `settings`: BunkerWeb multisite settings as key-value pairs (keys must match BunkerWeb setting names exactly)
- `configs`: array of relative paths to NGINX config files
- `steps`: ordered workflow steps with `title`, `subtitle`, `settings`, and `configs` for guided UI import

Use `templates/wordpress/` as the reference template for structure, tone, and step organization.

**CI/CD** — two GitHub Actions workflows:

- `dev-template-prerelease.yml`: auto-packages templates on push to `dev` branch
- `manual-template-release.yml`: workflow dispatch for production releases

## Conventions

- Directory names: lowercase-kebab-case, ASCII only
- JSON: two-space indent, double-quoted keys, no trailing commas (Prettier-enforced)
- Commits: imperative mood ("Add Jellyfin template"), single concern per commit
- Changelog: all changes go under `## Unreleased` in `CHANGELOG.md` as `- [@github-handle] Summary`
- Config file naming: `<service>_false_positives.conf` for ModSecurity exclusion rules
- All config paths in `template.json` are relative to the template root; referenced files must exist in-repo
- NGINX snippets should include comments when relaxing security defaults

## Git Workflow

- Main branch: `main` (releases), development branch: `dev` (auto pre-releases)
- Feature branches: `feature/<name>`, bugfix branches: `bugfix/<name>`
- PRs target `main` with validation notes and linked issues
- Pre-commit suite must pass before opening PRs

## External References

- [BunkerWeb templates documentation](https://docs.bunkerweb.io/latest/concepts/#templates)
- `TEMPLATE_GUIDE.md` for detailed template authoring specification
- `CONTRIBUTING.md` for full contribution workflow
