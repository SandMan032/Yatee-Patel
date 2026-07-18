# Yatee Patel — Engineering Blog

A personal tech blog documenting the design of an algorithmic trading bot, plus
notes on Go, Docker and systems design.

- **Framework:** [Hugo](https://gohugo.io/) (Extended)
- **Theme:** [Congo v2](https://github.com/jpanther/congo), installed as a **Hugo Module** (not a git submodule)
- **Hosting:** GitHub Pages, deployed via GitHub Actions (`.github/workflows/deploy.yml`)

## Prerequisites

- Hugo **Extended** (`hugo version` should show `+extended`)
- **Go** (required — Congo is a Hugo Module, so `hugo` uses Go to fetch it)

## Site structure

Content is organised into dedicated sections, each a Hugo **page bundle**
(a folder with `index.md` plus its images):

| Path | Purpose |
| --- | --- |
| `content/bot-architecture/` | Deep dives on the trading bot's design |
| `content/learning-log/` | Notes on Go, Docker, systems design |
| `content/misc/` | Shorter, essay-style posts |
| `content/about/` | About page |

Each section has an `_index.md` (its landing-page description). The homepage
intro lives in `content/_index.md`.

## Configuration

Config is split under `config/_default/`:

| File | Contains |
| --- | --- |
| `config.toml` | Base settings, permalinks, taxonomies, output formats |
| `module.toml` | Imports the Congo v2 Hugo Module |
| `params.toml` | Congo appearance & feature flags (`ocean` scheme, hero images, search, TOC…) |
| `languages.en.toml` | Language settings + the author profile card |
| `menus.toml` | Top navigation bar |

## Writing a new post

Posts are page bundles. Create a folder with an `index.md` and drop any images
alongside it (name the cover `feature.png`/`.jpg` so Congo picks it up as the hero):

```bash
mkdir -p content/learning-log/my-new-post
$EDITOR content/learning-log/my-new-post/index.md
```

Minimal front matter:

```yaml
---
title: "My New Post"
date: 2026-07-18
draft: false
tags: ["go", "docker"]
categories: ["Learning Log"]
---
```

## Local development

```bash
hugo server -D            # preview at http://localhost:1313 (includes drafts)
hugo --gc --minify        # production build into ./public
```

## Changing the colour scheme

In `config/_default/params.toml`, set `colorScheme` to one of:
`congo`, `avocado`, `cherry`, `fire`, `ocean`, `sapphire`, `slate`.

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which installs Go +
Hugo Extended, builds the site, and publishes to GitHub Pages.
