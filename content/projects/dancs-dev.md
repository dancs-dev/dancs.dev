+++
title = 'dancs.dev'
date = 2025-07-28T14:42:49+01:00
draft = false
summary = "My portfolio and blog, made using the open-source static site generator Hugo."
[params]
    repo = 'https://github.com/dancs-dev/dancs.dev'
+++

## Introduction

I wanted a platform - my platform - to showcase my projects and share technical insights with others. After trying out many different options including Next.js, Django, and Zola, I chose to build *dancs.dev* using [Hugo](https://gohugo.io/), an open-source static site generator written in Go. I was drawn to Hugo for several reasons - content is written in Markdown, builds are fast, and static sites are straightforward to host and maintain. There is no database or server-side runtime to manage, just HTML and a small collection of static files.

## Technology stack

- **Hugo:** static site generator.
- **Markdown:** for content creation with TOML frontmatter.
- **CSS:** minimal styling without heavy frameworks.
- **Vanilla JavaScript:** provides additional features but not required for core functionality.
- **GitHub Actions:** automated build and deployment.
- **GitHub Pages:** hosting.

## Creating a custom theme: Ikigai

Rather than using an existing theme, I built [Ikigai](https://github.com/dancs-dev/ikigai) from scratch. Ikigai is a minimalist Hugo theme designed for personal portfolios and blogs. Creating my own theme has given me full control over the design as well as enables me to add a custom feature set to suit my site.

Key design decisions:
- **Optional JavaScript:** the site is fully usable without the user having to enable JavaScript.
- **Minimal dependencies:** assets are provided with the theme rather than loaded from external resources.
- **Accessibility:** the default styling and semantic HTML in templates support accessibility best practices.

## Deployment pipeline

The site is built and deployed automatically via GitHub Actions when I push to `main`, allowing me to quickly and simply update the site.

## Challenges and lessons learnt

### Managing the theme

Splitting off the design into a separate repository as a standalone Hugo theme has improved the maintainability of the site as well as enables re-use across other projects. Getting the theme submodule to play well with GitHub Actions required me to load it using a relative URL, i.e., `url = ../ikigai.git`. This allows the submodule be cloned using the same cloning method as the main project (e.g., SSH), but ensures the submodule can be cloned in GitHub actions (e.g., HTTPS).

### Removing external resources

Hosting all of the required resources locally rather than relying on external resources adds some complexity, however I think this is important to do, to ensure that I am in full control of the site as well as that the privacy of users are respected. Fortunately, there are lots of tools to help do this, such as [Mario Ranftl's Google Webfonts Helper](https://github.com/majodev/google-webfonts-helper).

## What's next

I'm continuously improving both the theme and the site (as well as adding more content to the site). Recent additions include contact links to the footer and linking to project repositories via frontmatter settings.