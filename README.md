# AIDF Platform Documentation

Documentation site for the **Centific AI Data Foundry (AIDF)** platform, covering all six February 2026 training sessions.

**Live site:** https://asvskartheek.github.io/aidf-docs/

## What's Covered

- Platform overview — 6 modules: Data Hub, AI Workbench, GenAI Studio, Agent Workbench, Infra Hub, GenBI
- First steps walkthrough for new users
- Infra Hub deep-dive — Workspaces, Projects, AI Services, Pipeline Templates, HIL Tasks, RBAC
- Developer guide — folder structure, strategy pattern, logging, Docker, manifest
- Code reference — Reference Template, Phi-4 Fine-Tuning, Suspicious Activity Detection
- Full tutorial: build and deploy a Support Ticket Classifier AI Service from scratch

## Local Development

```bash
# Preview with live reload
uv run mkdocs serve

# Build static site
uv run mkdocs build

# Deploy to GitHub Pages
uv run mkdocs gh-deploy
```

## Project Structure

```
docs_src/        # Markdown source files
mkdocs.yml       # Site configuration
pyproject.toml   # Python dependencies (managed by uv)
```
