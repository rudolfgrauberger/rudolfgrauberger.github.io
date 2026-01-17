# grauberger.de Blog

A personal tech blog built with [Hugo](https://gohugo.io/) using the [Poison theme](https://github.com/lukeorth/poison).

## Prerequisites

- [Hugo](https://gohugo.io/installation/) (extended version recommended)
- Git

### Verify Hugo Installation

```bash
hugo version
# Expected output similar to: hugo v0.123.7+extended linux/amd64
```

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/rudolfgrauberger/rudolfgrauberger.github.io.git
cd rudolfgrauberger.github.io.git
```

### 2. Initialize Theme Submodule

The Poison theme is included as a Git submodule. After cloning, initialize it:

```bash
git submodule update --init --recursive
```

### 3. Run the Development Server

```bash
hugo serve
```

The site will be available at `http://localhost:1313/`. The server supports hot-reload, so changes are reflected immediately.

#### Additional serve options:

```bash
# Include draft posts
hugo serve -D

# Bind to all network interfaces (useful for testing on other devices)
hugo serve --bind 0.0.0.0
```

## Writing Content

### Creating a New Post

Use Hugo's built-in scaffolding:

```bash
hugo new posts/my-new-post/index.md
```

This creates a new post using the archetype template with the following front matter:

```yaml
---
title: "My New Post"
date: 2025-12-22
draft: true
---
```

### Post Front Matter Options

| Field      | Description                                    | Required |
|------------|------------------------------------------------|----------|
| `title`    | The post title                                 | Yes      |
| `date`     | Publication date (YYYY-MM-DD)                  | Yes      |
| `draft`    | Set to `false` to publish                      | Yes      |
| `tags`     | List of tags, e.g., `["hugo", "tutorial"]`     | No       |
| `series`   | Series name for multi-part posts               | No       |
| `hideToc`  | Set to `true` to hide table of contents        | No       |

### Example Post

```markdown
---
title: "Getting Started with Hugo"
date: 2025-12-22
draft: false
tags: ["hugo", "tutorial", "static-site"]
series: "Hugo Series"
---

Introduction paragraph that appears in the post list.

<!--more-->

The rest of your content goes here...
```

> **Note:** The `<!--more-->` tag marks where the excerpt ends on listing pages.

### Adding Images

Place images in the same folder as your post's `index.md`:

```
content/posts/my-new-post/
├── index.md
├── image1.png
└── screenshot.jpg
```

Reference them in your markdown:

```markdown
![Alt text](image1.png)
```

## Shortcodes

### Notice Box

Display styled notice boxes with different types:

```markdown
{{< notice note >}}
This is a note.
{{< /notice >}}

{{< notice info >}}
This is an info box.
{{< /notice >}}

{{< notice warning >}}
This is a warning.
{{< /notice >}}

{{< notice success >}}
This is a success message.
{{< /notice >}}
```

### Archive Note

For archived posts migrated from the old blog system:

```markdown
{{< archiv-note >}}
```

This displays a predefined notice about archived content.

## Building for Production

Generate the static site:

```bash
hugo
```

The output will be in the `public/` directory, ready to be deployed.

### Build Options

```bash
# Clean build (remove old files first)
rm -rf public && hugo

# Build with minification
hugo --minify
```

## Project Structure

```
.
├── archetypes/          # Templates for new content
│   └── default.md       # Default post template
├── assets/              # Assets processed by Hugo Pipes
│   └── css/
│       └── custom.css   # Custom CSS overrides
├── content/             # All content
│   └── posts/           # Blog posts
│       └── archiv/      # Archived posts from old blog
├── data/                # Data files (JSON, YAML, TOML)
├── i18n/                # Internationalization files
├── layouts/             # Custom layout overrides
│   └── shortcodes/      # Custom shortcodes
├── public/              # Generated site (git-ignored)
├── static/              # Static files copied as-is
├── themes/poison/       # Poison theme (submodule)
└── hugo.toml            # Site configuration
```

## Configuration

Main configuration is in [hugo.toml](hugo.toml). Key settings:

| Setting                  | Description                              |
|--------------------------|------------------------------------------|
| `baseURL`                | Production URL of the site               |
| `title`                  | Site title                               |
| `params.brand`           | Name shown in sidebar                    |
| `params.description`     | Default meta description                 |
| `params.dark_mode`       | Enable dark mode by default              |
| `params.menu`            | Sidebar navigation items                 |

## Workflow Summary

1. **Start server:** `hugo serve -D`
2. **Create post:** `hugo new posts/my-post-slug/index.md`
3. **Write content** in the created markdown file
4. **Preview** at `http://localhost:1313/`
5. **Set `draft: false`** when ready to publish
6. **Build:** `hugo`
7. **Deploy** the `public/` directory

## License

Content © Rudolf Grauberger. Theme: [Poison](https://github.com/lukeorth/poison).
