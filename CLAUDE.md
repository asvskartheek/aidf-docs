# AIDF Platform Documentation — Project Guide

## What This Project Is

This is the **Zensical documentation site** for the Centific AI Data Foundry (AIDF) platform, covering all six February 2026 training sessions. The content was originally authored as custom HTML files, migrated to MkDocs with the Material theme, and subsequently moved to **Zensical** (the MkDocs-compatible successor built by the Material for MkDocs team) for long-term maintainability.

## Tech Stack

| Tool | Purpose |
|------|---------|
| [Zensical](https://zensical.org/) | Static site generator (MkDocs-compatible successor by the Material for MkDocs team) |
| [uv](https://docs.astral.sh/uv/) | Python package and environment manager |
| [pymdownx extensions](https://facelessuser.github.io/pymdown-extensions/) | Admonitions, superfences, tasklists, emoji/icons, tabbed content (bundled with Zensical) |

## Directory Layout

```
Training/
├── .github/
│   └── workflows/
│       └── docs.yml       # GitHub Actions workflow for deploying to GitHub Pages
├── CLAUDE.md              # This file
├── mkdocs.yml             # Site configuration (theme, nav, extensions) — read by Zensical as-is
├── pyproject.toml         # Python project deps managed by uv
├── uv.lock                # Locked dependency versions
├── docs_src/              # Markdown source files (Zensical reads this)
│   ├── index.md           # Home page — platform overview, modules, training sessions
│   ├── first-steps.md     # 7-step walkthrough for new users
│   ├── tutorial.md        # Full code tutorial: Build a Support Ticket Classifier AI Service
│   ├── stylesheets/
│   │   └── extra.css      # Custom CSS overriding Material theme colors (#4f7cff brand)
│   ├── platform/          # One page per AIDF module
│   │   ├── data-hub.md
│   │   ├── ai-workbench.md
│   │   ├── genai-studio.md
│   │   ├── agent-workbench.md
│   │   └── genbi.md
│   ├── infra-hub/         # Infra Hub deep-dive
│   │   ├── index.md
│   │   ├── workspaces.md
│   │   ├── projects.md
│   │   ├── ai-services.md
│   │   ├── pipeline-templates.md
│   │   ├── hil-tasks.md
│   │   └── rbac.md
│   ├── developer/         # AI Service development guide
│   │   ├── index.md
│   │   ├── folder-structure.md
│   │   ├── strategy-pattern.md
│   │   ├── logging.md
│   │   ├── onboarding.md
│   │   ├── docker.md
│   │   └── manifest.md
│   ├── reference/         # Code reference for specific services
│   │   ├── reference-template.md
│   │   ├── phi4-service.md
│   │   └── suspicious-activity.md
│   └── appendix/
│       ├── tech-stack.md
│       ├── environments.md
│       └── compute-offloading.md
├── site/                  # Generated static site output (do not edit manually)
└── docs/                  # Original custom HTML files (legacy, kept for reference)
```

## Common Commands

```bash
# Local preview with live reload
uv run zensical serve

# Build static site to site/
uv run zensical build --clean
```

## Configuration Notes (`mkdocs.yml`)

Zensical reads the existing `mkdocs.yml` directly — no conversion needed.

- **`docs_dir: docs_src`** — Source is `docs_src/` (not the default `docs/`) to avoid conflict with the original HTML files in `docs/`
- **`site_dir: site`** — Output goes to `site/`
- **Theme**: Material slate (dark), primary/accent indigo, overridden to `#4f7cff` via `extra.css`
- **Key extensions enabled**:
  - `pymdownx.emoji` — required for `:material-icon-name:` syntax in pages
  - `pymdownx.tasklist` — required for `- [ ]` / `- [x]` checkbox rendering
  - `pymdownx.superfences` — fenced code blocks with syntax highlighting + Mermaid diagrams
  - `pymdownx.tabbed` — `=== "Tab"` content tabs
  - `admonition` + `pymdownx.details` — `!!! note`, `!!! warning`, collapsible blocks

## Adding a New Page

1. Create `docs_src/<section>/your-page.md`
2. Add it to the `nav:` section in `mkdocs.yml`
3. Run `uv run zensical serve` to preview

## Dependency Management

Dependencies are managed with `uv`. Do **not** manually edit `pyproject.toml` or `uv.lock`.

```bash
# Add a new package
uv add <package-name>

# Remove a package
uv remove <package-name>
```

## GitHub Pages Deployment

The site is deployed automatically via **GitHub Actions** (`.github/workflows/docs.yml`). On every push to `main`, the workflow:
1. Installs Zensical
2. Builds the static site with `zensical build --clean`
3. Deploys to GitHub Pages using the `actions/deploy-pages` action

> **Note:** Zensical does not support `gh-deploy`. The GitHub Actions workflow replaces it.
> You may need to enable GitHub Pages with "GitHub Actions" as the source in your repo settings
> (Settings → Pages → Source → GitHub Actions).

## Content Origin

The documentation was migrated from three large custom HTML files:
- `docs/index.html` — Platform reference (all 6 modules, Infra Hub guide, Developer guide, Code Reference)
- `docs/first-steps.html` — New user 7-step walkthrough
- `docs/ai-service-tutorial.html` — Full code tutorial (Support Ticket Classifier)

These HTML files are kept in `docs/` as legacy reference but are not part of the MkDocs build.
