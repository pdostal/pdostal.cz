# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal blog/website for Pavel Dostál, built with **Hugo** static site generator. Bilingual (English and Czech), using the Harbor theme as a Git submodule. Every push to `master` triggers a GitHub Actions build and release deployment.

## Common Commands

**Build the site:**
```bash
hugo --minify
```

**Local development server:**
```bash
podman run --rm -v "$PWD":/src -p 0.0.0.0:1313:1313 -w /src docker.io/klakegg/hugo:ext serve
```

**Lint markdown:**
```bash
markdownlint '**/*.md'
```

**Lint YAML:**
```bash
yamllint .
```

**Lint shell scripts:**
```bash
shellcheck <script.sh>
```

**Format code:**
```bash
prettier --check .
```

## Architecture

- `config.toml` — Hugo config: base URL, bilingual setup (`defaultContentLanguage = "cs"`), Harbor theme, Matomo analytics, permalink structure
- `content/` — Blog posts and pages, organized by language (`cs/` and `en/` subdirectories within post directories)
- `layouts/` — Custom layout overrides for the Harbor theme
- `archetypes/` — Templates for new content
- `static/` — CSS overrides, images, SSH/PGP public keys, configuration file examples
- `themes/harbor/` — Git submodule; do not edit directly
- `public/` — Hugo output directory (gitignored)

## CI/CD

- **`.github/workflows/build.yml`** — Builds with Hugo on every push (validates the build)
- **`.github/workflows/deploy.yml`** — On `master` pushes: builds site, packages `site.tar.gz`, creates a timestamped GitHub release
- **`.github/workflows/claude.yml`** — Claude Code responds to `@claude` mentions in issues/PRs
- **`.github/workflows/claude-code-review.yml`** — Automated code review on PRs

Hugo version pinned in CI: **v0.147.4**

## Content Guidelines

- Posts are bilingual; each post directory contains language-specific files (e.g., `index.cs.md`, `index.en.md`)
- Front matter uses TOML format with fields: `title`, `date`, `tags`, `series`, `image`
- Code blocks should use fenced syntax with language identifiers (including `ini` for systemd-networkd configs)
- Linting rules allow long lines (120 chars) and inline HTML (`MD033` disabled for Hugo shortcodes)
