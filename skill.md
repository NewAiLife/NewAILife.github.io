# Skill: AI Life Blog — Jekyll Development SOPs

## Overview

This site is built with **Jekyll** using the **Chirpy** theme, running inside a Docker container (`jekyll/jekyll`), and deployed to **GitHub Pages** at `NewAILife.github.io`.

---

## 1. Local Development

### 1.1 Start Jekyll (live-reload)

```bash
docker run --rm \
  --volume=".:/srv/jekyll" \
  -p 4000:4000 \
  -it jekyll/jekyll \
  jekyll serve --livereload
```

Then open **http://localhost:4000**.

### 1.2 One-off build

```bash
docker run --rm --volume=".:/srv/jekyll" -it jekyll/jekyll jekyll build
```

### 1.3 Shell inside container

```bash
docker run --rm --volume=".:/srv/jekyll" -it jekyll/jekyll sh
```

---

## 2. Creating a Blog Post

1. Create a markdown file in `_posts/` with the naming convention: `YYYY-MM-DD-slug.md`
2. Add front matter at the top:

```yaml
---
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [AI for Productivity]
tags: [tag1, tag2]
pin: false
---
```

3. Write the post content in Markdown below the front matter.

### Valid categories

- AI for Productivity
- AI for Health & Wellness
- AI for Photography
- AI for Learning
- AI for Families
- AI for Coding
- AI Experiments
- AI Reviews
- AI News

---

## 3. Site Configuration

Edit `_config.yml` for global settings:

| Setting | Description |
|---|---|
| `title` | Site name (e.g., "New AI Life") |
| `tagline` | Subtitle shown below title |
| `url` | Full URL once deployed (`https://newailife.github.io`) |
| `github.username` | GitHub username |
| `social.name` | Author display name |
| `social.email` | Author email |
| `avatar` | Path to sidebar avatar image |
| `timezone` | Timezone for post dates |

---

## 4. Git Workflow

```bash
# Create a new branch for your work
git checkout -b feature/my-change

# Stage and commit
git add -A
git commit -m "Brief description of change"

# Push to remote
git push origin feature/my-change

# Create a Pull Request on GitHub to merge into main
```

> **Rule**: Every commit should be one logical change. Ask before committing.

---

## 5. Deployment

The site deploys automatically via GitHub Pages when changes are pushed to `main` branch. Ensure:

- `_config.yml` has the correct `url` and `baseurl`
- Build succeeds locally before pushing
- No broken links or missing assets
