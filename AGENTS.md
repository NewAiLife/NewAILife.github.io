# Agents Guide

This is a website for sharing how to use AI in life. The topics this site may cover are the following.

- AI for Productivity
- AI for Health & Wellness
- AI for Photography
- AI for Learning
- AI for Families
- AI for Coding
- AI Experiments
- AI Reviews
- AI News

This site will be hosted on github pages as NewAILife.github.io. The building template is jekyll and the building tool is a docker container jekyll/jekyll

## Environment

| Item | Path / Value |
|---|---|
| Project root | `F:\ai\Projects\NewAILife.github.io` |
| Jekyll Docker image | `jekyll/jekyll` |
| Shell | Git Bash (MinGW64) |
| Shell | Jekyll Docker container |

## File Organization Rules

| Directory / File | Purpose |
|---|---|
| `_posts/` | Blog posts in markdown (filename: `YYYY-MM-DD-slug.md`) |
| `_tabs/` | Top navigation tabs (e.g., About, Categories, Tags) |
| `_data/` | Site data files (locales, contact, share) |
| `_layouts/` | Page layout templates |
| `_includes/` | Reusable HTML partials |
| `_sass/` | SCSS stylesheets |
| `_javascript/` | JavaScript source files |
| `assets/` | Static assets (images, compiled CSS/JS) |
| `docs/` | Project documentation (excluded from build) |
| `tools/` | Build tooling scripts (excluded from build) |

## Steps and Milestone
Each milestone should be one or more git commit. Any commands, file editing, and code commit should ask for permission before taking actions.
| Step | Milestone | Status |
|---|---|---|
|1|Create a development environment for local test, GitHub deployment| TODO |
|2|Create document, README.md for local build, test| TODO |
|3|Update Agents.md and skill.md| DONE |
|4|Update document for how to configure GitHub Actions to auto deploy after push change| TODO |

## Starting jekyll container.
```
# Git Bash (Windows / MinTTY): prefix with winpty
export site_name=NewAILife
winpty docker run --rm --volume=".:/srv/jekyll" -it jekyll/jekyll sh

# Linux / macOS:
# export site_name=NewAILife
# docker run --rm --volume=".:/srv/jekyll" -it jekyll/jekyll sh
```
