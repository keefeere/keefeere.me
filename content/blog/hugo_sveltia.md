---
title: "Integrating Sveltia CMS with Hugo: A Real-World Setup"
date: "2025-07-11"
description: >
  A complete guide to integrating Sveltia CMS with a Hugo-based static
  site. Lessons learned, problems faced, and how to avoid common pitfalls.
tags: ["Hugo", "Sveltia", "CMS", "GitHub", "Netlify"]
---

## Introduction

Hugo is a fast and flexible static site generator, but content management often
requires either committing Markdown files manually or relying on tools like
Netlify CMS. With [Sveltia CMS](https://github.com/sveltia/sveltia-cms),
it's now possible to edit Hugo content through a browser, including
drag-and-drop image uploads and GitHub-based authentication.

However, real-world usage exposed several unexpected issues. This post documents
how I configured Sveltia CMS for my Hugo blog
and what it took to get it working reliably.

## Requirements

- Existing Hugo site hosted on GitHub
- GitHub Pages or self-hosted web server
- Basic CI/CD setup (e.g., GitHub Actions)
- Netlify account (for GitHub authentication only — actual hosting not required)

## Step-by-Step Setup

### 1) Add Admin Interface to Hugo

Create `static/admin/` directory and add the following:

`static/admin/index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Sveltia CMS</title>
    <script
      src="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.js"
      onload="SVELTIA.start()"
      onerror="document.body.innerHTML='Failed to load Sveltia CMS'"
    ></script>
    <link
      href="/admin/config.yml"
      type="application/yaml"
      rel="cms-config-url"
    />
  </head>
  <body></body>
</html>
```

`static/admin/config.yml`

```yml
backend:
  name: github
  repo: yourusername/yourrepo
  branch: main
  auth:
    client_id: YOUR_GITHUB_OAUTH_CLIENT_ID

media_folder: "static/uploads"
public_folder: "/uploads"

collections:
  - name: "blog"
    label: "Blog Posts"
    folder: "content/blog"
    create: true
    slug: "{{slug}}"
    extension: "md"
    format: "yaml"
    fields:
      - label: "Title"
        name: "title"
        widget: "string"
      - label: "Date"
        name: "date"
        widget: "datetime"
      - label: "Description"
        name: "description"
        widget: "text"
        required: false
      - label: "Tags"
        name: "tags"
        widget: "list"
        required: false
      - label: "Body"
        name: "body"
        widget: "markdown"
```

### 2) Create GitHub OAuth App

1. Go to GitHub → Settings → Developer settings → OAuth Apps
2. Click "New OAuth App"
3. Name: your blog or "Sveltia Hugo CMS"
4. Homepage: `https://yoursite.example`
5. Callback URL: `https://api.netlify.com/auth/done`
6. Save and copy the `Client ID` into your config.yml

### 3) Configure Netlify Auth (No Hosting Required)

1. Log in to Netlify
2. Create any dummy site from GitHub repo (not used for hosting)
3. Go to "Site Settings" → "Identity" → Enable Git Gateway
4. Under "Services", enable GitHub as external provider
5. Set OAuth client ID matching your GitHub app

Sveltia uses Netlify's `api.netlify.com` OAuth proxy for GitHub login.

## Notes on YAML Frontmatter

Sveltia CMS is stricter than Hugo about frontmatter formatting:

- Only one YAML document allowed per file (`---` at top and bottom only)
- No multiple `---` delimiters inside body content
- Quotes must be balanced
- Avoid inline `---` unless you escape them (e.g., `\---`)

Example of valid frontmatter:

```yaml
    ---
    title: "My Post"
    date: 2025-07-10
    description: "This is a valid description."
    tags: ["tag1", "tag2"]
    ---
```

## CI: Cleaning before Hugo build

In GitHub Actions:

```yaml
- name: Clean Hugo output directory
  run: rm -rf public/
```

Cleaning avoids publishing outdated or conflicting files.

## Common Errors

- `SVELTIA is not defined`: check script tag and CDN link
- 404 on `/admin`: make sure `static/admin/` exists and is deployed
- YAML parse errors: ensure frontmatter is well-formed
- Images not uploading: convert HEIC to JPEG before upload

## Conclusion

Sveltia CMS offers an elegant interface for editing Hugo content directly in the
browser. It's not plug-and-play, but once properly configured, it simplifies
content management significantly without needing full CMS hosting.
