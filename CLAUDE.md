# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

German-language technical blog built with Hugo static site generator. Uses the Poison theme as a Git submodule.

## Common Commands

```bash
# Start development server with hot-reload
hugo serve

# Include draft posts in preview
hugo serve -D

# Create new post
hugo new posts/my-post-slug/index.md

# Production build
hugo

# Build with minification
hugo --minify

# Initialize/update theme submodule
git submodule update --init --recursive
```

Development server runs at `http://localhost:1313/`

## Architecture

**Content Organization:**
- Posts use Hugo page bundles: `content/posts/<post-slug>/index.md`
- Images and assets are stored alongside post in same directory
- Archived posts (migrated from legacy Jekyll) are in `content/posts/archiv/`
- Front matter uses TOML format (`+++` delimiters)

**Theme:**
- Poison theme located in `themes/poison/` (Git submodule)
- Do not modify theme files directly; use `layouts/` for overrides

**Custom Components:**
- `layouts/shortcodes/notice.html` - Styled notice boxes (note, info, warning, success)
- `layouts/shortcodes/archiv-note.html` - Archive notice for migrated posts
- `assets/css/custom.css` - Custom CSS overrides

**Configuration:**
- Main config in `hugo.toml`
- Site language: German (`de-de`)
- Taxonomies: `tags` and `series`

## Post Front Matter

```toml
+++
title = 'Post Title'
date = 2025-01-17
draft = false
tags = ['tag1', 'tag2']
series = 'Optional Series Name'
hideToc = false
+++
```

Use `<!--more-->` in post content to define excerpt break.

## Shortcode Usage

```markdown
{{< notice note >}}Hinweis content{{< /notice >}}
{{< notice info >}}Info content{{< /notice >}}
{{< notice warning >}}Warnung content{{< /notice >}}
{{< notice success >}}Erfolg content{{< /notice >}}

{{< archiv-note >}}  <!-- Shows archive notice from config -->
```
