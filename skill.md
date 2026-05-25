# Skill: AI Life Blog — Jekyll Development SOPs

## Overview

This site is built with **Jekyll** using the **Chirpy** theme, running inside a Docker container (`jekyll/jekyll`), and deployed to **GitHub Pages** at `NewAILife.github.io`.

---

## 1. Local Development

> **Git Bash (Windows) notes:**
> - Interactive commands (`-it`) need `winpty` prefix
> - Non-interactive commands need `MSYS_NO_PATHCONV=1` to prevent path mangling
> - Always use `$PWD` (not `.`) for volume mounts

### 1.1 Start Jekyll (live-reload)

```bash
# Windows Git Bash:
winpty docker run --rm \
  --volume="$PWD:/srv/jekyll" \
  -p 4000:4000 \
  -it jekyll/jekyll \
  bundle exec jekyll serve --livereload

# Linux / macOS:
# docker run --rm --volume="$PWD:/srv/jekyll" -p 4000:4000 -it jekyll/jekyll bundle exec jekyll serve --livereload
```

Then open **http://localhost:4000**.

> **Note:** The container is ephemeral. Gems are installed on each run. For development, it's more efficient to open a shell (section 1.3) and run commands interactively.

### 1.2 One-off build (CI / verification)

```bash
# Windows Git Bash (non-interactive):
MSYS_NO_PATHCONV=1 docker run --rm \
  --volume="$PWD:/srv/jekyll" \
  -e HOME=/tmp \
  jekyll/jekyll \
  sh -c "bundle install --quiet && bundle exec jekyll build"

# Linux / macOS:
# docker run --rm --volume="$PWD:/srv/jekyll" jekyll/jekyll sh -c "bundle install --quiet && bundle exec jekyll build"
```

Output is in `_site/`.

### 1.3 Shell inside container

```bash
# Windows Git Bash:
winpty docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll sh

# Once inside:
#   bundle install          # install gems
#   bundle exec jekyll serve --livereload
#   bundle exec jekyll build
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
- AI for Nature
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

---

## 6. Gemfile Requirements

The Gemfile needs these additions beyond the default Chirpy setup:

```ruby
gem "webrick"                          # Required for Jekyll serve with Ruby 3.x
gem "jekyll-sass-converter", "~> 2.0"  # Avoid sass-embedded broken pipe on musl/Alpine
```

`Gemfile.lock` is gitignored and regenerated inside the container on each run.

---

## 7. Troubleshooting

| Symptom | Fix |
|---|---|
| `the input device is not a TTY` | Prefix with `winpty` |
| `includes invalid character for a local volume name` | Use `$PWD` instead of `.` for volume mount |
| `C:/Program Files/Git/srv/jekyll` path error | Add `MSYS_NO_PATHCONV=1` before `docker run` |
| `cannot load such file -- webrick` | Add `gem "webrick"` to Gemfile |
| `Broken pipe` SCSS error | Pin `jekyll-sass-converter` to `~> 2.0` |
| `detected dubious ownership in repository` | Cosmetic warning, not fatal. Use `-e HOME=/tmp` + `git config --global --add safe.directory /srv/jekyll` to suppress |
