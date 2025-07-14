---
title: "Adding Comments to Hugo Blog with Giscus"
date: "2025-07-07"
description: "Guide to add comments to Hugo blog"
tags: ["Hugo", "Blog", "giscus", "GitHub"]
---

If you're using Hugo (e.g. with the
[m10c](https://github.com/vaga/hugo-theme-m10c) theme) and want to enable
GitHub-powered comments, `Giscus` is a great privacy-respecting and lightweight
option. This guide shows how to integrate it properly — including Hugo
overrides, theming, and comment moderation.

## What is Giscus

[Giscus](https://giscus.app) is a comment system powered by GitHub Discussions.
Visitors comment using their GitHub account, and each post maps to a discussion
thread. Benefits:

- Lightweight and static-site-friendly
- No tracking, no ads
- Comments are stored in your GitHub repo
- Supports dark/light themes

## Prerequisites

- A Hugo blog (e.g. using `m10c` theme)
- Your content is versioned on GitHub
- Discussions enabled on your GitHub repo

## Step 1 — Enable GitHub Discussions

1. Go to your blog’s GitHub repo.
2. Navigate to **Settings → Features**.
3. Enable **Discussions**.

## Step 2 — Configure Giscus

1. Go to <https://giscus.app>
2. Click “Sign in with GitHub”
3. Under “Install the GitHub App”, click “Install”
   Select your blog repository from the list and grant access
   (This is required so Giscus can read and write Discussions in your repo)
4. After installing the app:
   1) Select your repo
   2) Choose the Discussion category (or create one)
   3) Pick your preferred options:
   4) Mapping: pathname or title
   5) Theme: preferred_color_scheme, dark_dimmed, etc.
   6) Enable Reactions, metadata, etc.
5. Copy the generated `<script>` block.

Example:

```html
<script src="https://giscus.app/client.js"
        data-repo="your-username/your-blog-repo"
        data-repo-id="..."
        data-category="General"
        data-category-id="..."
        data-mapping="pathname"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-theme="preferred_color_scheme"
        crossorigin="anonymous"
        async>
</script>
```

⚠️ If you skip installing the GitHub App, Giscus will render a blank
comment box or fail to load.

## Step 3 — Add single.html with layout override

Create a layout override so the Giscus widget appears below each blog post.

File:

`layouts/_default/single.html`

Content:

```html
{{ define "main" }}
  <main>
    {{ .Content }}
    {{ partial "giscus.html" . }}
  </main>
{{ end }}
```

This override works with Hugo themes that use baseof.html (like m10c).
Don’t edit files inside themes/ directly.

⸻

## Step 4 — Create giscus.html Partial

File:

`layouts/partials/giscus.html`

Paste your script there:

```html
<div id="comments" style="margin-top: 3rem;">
  <script src="https://giscus.app/client.js"
          data-repo="your-username/your-blog-repo"
          data-repo-id="..."
          data-category="General"
          data-category-id="..."
          data-mapping="pathname"
          data-reactions-enabled="1"
          data-emit-metadata="0"
          data-theme="preferred_color_scheme"
          crossorigin="anonymous"
          async>
  </script>
</div>
```

## Optional — Match Site Theme

Use data-theme="preferred_color_scheme" to match user’s system preference.

Other available themes:
 • light
 • dark
 • dark_dimmed
 • transparent_dark
 • noborder_dark

See Giscus theme preview for examples.

## Moderation & Spam Control

All comments are stored as GitHub Discussions in your repo. You can:

- Delete or hide comments
- Lock a thread (per post)
- Report or block users (via GitHub profile)
- Use GitHub moderation tools

There is no anonymous commenting, which greatly reduces spam.

## Upgrading Hugo Theme — Safe Overrides

As long as your custom single.html and giscus.html are in the
top-level `layouts/`
directory, Hugo will use them instead of the theme’s defaults.

Theme updates will not overwrite your custom layout files.

## Result

You now have a fully working comment system in your Hugo blog — open, transparent,
and Git-powered.

Need more control? Consider lazy-loading Giscus or hiding comments behind a button
for minimal UIs.
