---
title: "How to Set Up Your Blog with Hugo and GitHub Pages"
date: "2025-01-30"
description: "A step-by-step guide to creating a static
blog using Hugo and deploying it via GitHub Pages."
tags: ["Hugo", "Blog", "Static Site", "GitHub Pages", "DevOps"]
---

Hugo is a fast and flexible static site generator that is perfect
for creating blogs. This guide will walk you through setting up a
Hugo-based blog and deploying it to GitHub Pages with CI/CD automation.

## Prerequisites

Before you start, ensure you have the following installed:

* [Go](https://go.dev/dl/)
* [Hugo](https://gohugo.io/getting-started/installing/)
* [Git](https://git-scm.com/downloads)
* A GitHub account

## Step 1: Create a New Hugo Site

Run the following command to create a new Hugo site:

```sh
hugo new site my-blog
cd my-blog
```

This initializes a new Hugo project with a default folder structure.

## Step 2: Choose and Install a Theme

Hugo uses themes to style websites. You can find themes at
[Hugo Themes](https://themes.gohugo.io/). For example, to install
the "Paper" theme:

```sh
git submodule add https://github.com/nanxiaobei/hugo-paper themes/hugo-paper
```

Then, set the theme in `config.toml`:

```toml
theme = "hugo-paper"
```

## Step 3: Create Your First Post

Run the command:

```sh
hugo new posts/my-first-post.md
```

Edit the generated file in `content/posts/my-first-post.md` and add content.

## Step 4: Preview Your Blog Locally

Start a local server with:

```sh
hugo server -D
```

Visit `http://localhost:1313/` to see your blog.

## Step 5: Deploy to GitHub Pages

### 5.1 Initialize a Git Repository

```sh
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### 5.2 Configure GitHub Actions for Deployment

Create `.github/workflows/deploy.yml` with the following content:

```yaml
name: Deploy Hugo Site

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
      - name: Build
        run: hugo --minify
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

Commit and push this file. GitHub Actions will now build and deploy your site automatically.

## Step 6: Configure GitHub Pages

* Go to your GitHub repository
* Open "Settings" → "Pages"
* Set the source branch to `gh-pages`
* Your blog will be available at `https://YOUR_USERNAME.github.io/YOUR_REPO/`

## Conclusion

You now have a fully functional static blog powered by Hugo and
GitHub Pages! 🎉 Happy blogging!
