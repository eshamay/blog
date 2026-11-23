# blog

Jekyll site for GitHub Pages, published at https://eshamay.github.io/blog/.

The home page shows the most recent post in full, with a list of earlier posts (date and title) below it. `/archive/` lists every post.

## Add a post

Create `_posts/YYYY-MM-DD-slug.md`:

```markdown
---
layout: post
title: "Post title"
date: YYYY-MM-DD
---

Post body in Markdown.
```

The newest date becomes the home page. Put images in `assets/img/` and reference them with `{% raw %}{{ "/assets/img/name.png" | relative_url }}{% endraw %}`.

## Run locally

```
bundle install
bundle exec jekyll serve
```

Then open http://127.0.0.1:4000/blog/.

## Publish

Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`.
