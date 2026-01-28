# dancs.dev

A personal portfolio and blog showcasing software engineering projects, technical insights, and knowledge sharing. Built with Hugo static site generator and deployed to GitHub Pages.

[![Licence](https://img.shields.io/badge/licence-GPL--3.0-blue.svg)](LICENCE)
[![Hugo](https://img.shields.io/badge/Hugo-0.123.7+-ff4088?logo=hugo)](https://gohugo.io/)
[![Deployed on GitHub Pages](https://img.shields.io/badge/deployed%20on-GitHub%20Pages-222?logo=github)](https://dancs.dev/)

## Overview

This site serves as a portfolio for software engineering projects and a platform for sharing technical content. It features:

- **Project showcases:** detailed write-ups of personal projects including software projects and hardware integrations., homelab setups, and software solutions.
- **Technical blog:** in-depth articles on topics such as self-hosting.
- **Minimalist design:** using my custom [Ikigai](https://github.com/dancs-dev/ikigai) theme focused on clean and responsive design.
- **RSS feeds:** automatically generate an RSS feeds to allow people to subscribe to updates for blog posts and projects.
- **Automated deployment:** CI/CD pipeline using GitHub Actions for automatically publishing changes in `main`.

## Prerequisites

- [Hugo](https://gohugo.io/installation/) version 0.123.7 or higher.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/dancs-dev/dancs.dev.git
cd dancs.dev
```

### 2. Initialise the theme submodule

The site uses a custom theme called Ikigai, which is managed as a Git submodule.

```bash
git submodule update --init --recursive --remote
```

## Configuration

The site is configured through `hugo.toml` in the project root. Key configuration options include:

```toml
baseURL = 'https://dancs.dev/'
languageCode = 'en-gb'
title = 'dancs.dev'
theme = 'ikigai'
favicon = '/favicon.ico'
```

### Navigation menu

The main navigation is defined in `hugo.toml`:

```toml
[[menus.main]]
name = 'Home'
pageRef = '/'
weight = 10

[[menus.main]]
name = 'Projects'
pageRef = '/projects'
weight = 20

[[menus.main]]
name = 'Blog'
pageRef = '/blog'
weight = 30
```

### Site parameters

Author, feedback, and contact settings are configured under `[params]`:

```toml
[params.author]
    name = 'dancs-dev'

[params.feedback]
    enabled = true
    repository = 'https://github.com/dancs-dev/dancs.dev'

[params.contact.github]
    name = 'GitHub'
    href = 'https://github.com/dancs-dev'
    icon = 'bi bi-github'
```

## Development

### Running the development server

Start the Hugo development server with live reload:

```bash
hugo server -D
```

The site will be available at [http://localhost:1313/](http://localhost:1313/). Changes to content, layouts, or configuration will trigger automatic rebuilds.

### Project structure

```
dancs.dev/
├── archetypes/       # Content templates
│   └── default.md    # Default frontmatter template
├── content/          # Site content
│   ├── _index.md     # Home page
│   ├── blog/         # Blog posts
│   └── projects/     # Project showcases
├── static/           # Static assets (favicon, etc.)
├── themes/
│   └── ikigai/       # Custom theme (Git submodule)
├── .github/
│   └── workflows/
│       └── hugo.yml  # GitHub Actions deployment workflow
├── hugo.toml         # Site configuration
└── LICENCE           # GPL-3.0 licence
```

### Content organisation

- **Home page:** `/content/_index.md` - introduction and about page.
- **Projects:** `/content/projects/` - posts relating to personal projects.
- **Blog posts:** `/content/blog/` - posts relating to knowledge sharing.

### Creating new content

Hugo provides a command to generate new content with the correct frontmatter:

**Create a blog post:**
```bash
hugo new blog/example-blog-post.md
```

**Create a project:**
```bash
hugo new projects/example-project-post.md
```

New content is created with `draft = true` by default. This should be set to `draft = false` when ready to publish to ensure it is included in the production build.

### Content frontmatter format

All content uses TOML frontmatter. Here's the standard format:

**Blog Post:**
```toml
+++
title = 'Post Title'
date = 2025-07-28T14:42:49+01:00
draft = false
summary = 'Short description for listings and RSS feeds'
+++
```

**Project:**
```toml
+++
title = 'Project Name'
date = 2025-10-20T17:13:08+01:00
draft = false
summary = 'Brief project description'
[params]
    repo = 'https://github.com/dancs-dev/project-repo'
+++
```

The `summary` field is used for listing pages and RSS feed descriptions. The optional `repo` parameter adds a repository link to project pages.

### Building for production

Generate the static site with minification:

```bash
hugo --minify --baseURL "https://dancs.dev/"
```

The built site will be output to the `public/` directory.

## Theme

The site uses [Ikigai](https://github.com/dancs-dev/ikigai), my custom minimalist theme designed for my portfolio. The theme is managed as a Git submodule.

**Theme features:**
- Minimalist, responsive design.
- Self-hosted static content (e.g. no external font requests).
- Code snippets.
- Contact links with icons from [Bootstrap Icons](https://icons.getbootstrap.com/).
- Optional feedback section with automatic GitHub issue links.

**Update theme to latest version:**
```bash
git submodule update --remote themes/ikigai
```

**Theme repository:**
The theme is hosted in a separate repository (referenced in `.gitmodules` as `../ikigai.git`).

## Deployment

The site uses GitHub Actions for automated deployment to GitHub Pages.

### Automated Deployment

1. Push changes to the `main` branch.
2. GitHub Actions workflow (`.github/workflows/hugo.yml`) is triggered.
3. Hugo builds a production-ready version of the site.
4. Built site is deployed to GitHub Pages.
5. Site is live at https://dancs.dev/.

## Contributing

This is a personal portfolio site, but suggestions and bug reports are welcome. Please open an issue.

## Commit convention

This project follows [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` new features or content.
- `fix:` bug fixes.
- `chore:` routine tasks, dependency updates.
- `ci:` CI/CD pipeline changes.
- `docs:` documentation updates.
- `style:` code-formatting changes.

Example:
```bash
git commit -m "feat: add new blog post on Pi-hole"
git commit -m "fix: fix typo in air quality monitor project post"
```
