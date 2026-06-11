# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The public documentation site for Douro Data products, built with MkDocs + the Material theme. Published at https://docs.dourodata.com.

## Commands

```bash
pip install -r requirements.txt        # install deps (mkdocs, mkdocs-material, black, nox)
mkdocs serve -f douro_data_docs/mkdocs.yml   # run the docs locally with live reload
mkdocs build -f douro_data_docs/mkdocs.yml   # build the static site
```

The MkDocs project lives in `douro_data_docs/`, not the repo root, so either pass `-f douro_data_docs/mkdocs.yml` or run from inside that directory. A local virtualenv may exist at `env/` (gitignored).

## Structure

- `douro_data_docs/mkdocs.yml` — site config and nav. New pages must be added to the `nav:` section here to appear in the site.
- `douro_data_docs/docs/` — page content. Product pages go in `products/`, the healthcare-pricing section in `healthcare-pricing/` (its `/healthcare-pricing/` URL slug is referenced by Snowflake Marketplace listings — do not rename), images in `assets/`.
- `douro_data_docs/docs/CNAME` — custom-domain file (docs.dourodata.com); must be preserved for GitHub Pages.

## Naming

External brand is Douro Data; the word Melange (an internal codename) must never appear in published docs. Sample SQL uses `YOUR_DB` as the database-name placeholder.

## Deployment

Every push to `main` triggers `.github/workflows/publish_docs.yml`, which runs `mkdocs gh-deploy` and publishes the site to GitHub Pages immediately. There is no staging — anything merged to `main` goes live.
