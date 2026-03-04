# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Hexo** blog deployed to GitHub Pages. The site is built with Hexo 6.3.0 and uses the landscape theme.

## Common Commands

```bash
# Development server (live reload)
npm run server

# Build static files
npm run build

# Clean generated files and cache
npm run clean

# Build and deploy to GitHub Pages
npm run deploy

# Combined: clean, build, and deploy
npm run clean && npm run build && npm run deploy
```

## Project Structure

- `source/` - Blog posts (Markdown), images, and other assets
- `themes/landscape/` - Hexo landscape theme
- `public/` - Generated static files (do not edit directly)
- `_config.yml` - Main Hexo configuration

## Configuration

- Site title: 三山半落
- Language: zh-CN
- URL: https://pself.net
- Deployment: GitHub Pages (gh-pages branch)

## Writing Posts

Create new posts in `source/_posts/` with `.md` extension. Front matter format:
```yaml
---
title: Post Title
date: YYYY-MM-DD
categories:
  - category-name
tags:
  - tag-name
---
```

## Deployment

The site automatically deploys to `gh-pages` branch on `git@github.com:p-self/p-self.github.io.git`. Use `npm run deploy` which runs `hexo deploy`.
