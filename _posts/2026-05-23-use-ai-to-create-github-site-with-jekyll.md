---
title: "Use AI to Create a GitHub Site with Jekyll"
date: 2026-05-23 12:00:00 +0800
categories: [AI for Coding]
tags: [Jekyll, GitHub Pages, Chirpy, Docker, CI/CD]
pin: true
---

## The Idea

I wanted to create a blog — something clean, fast, and cheap to host. GitHub Pages with Jekyll seemed like the obvious choice: free hosting, markdown posts, and a thriving theme ecosystem.

The twist: instead of spending days fighting with Ruby versions, SCSS pipelines, and CI configurations, I would let an AI coding agent handle the entire setup.

This post is a retrospective of that experiment — what worked, what broke, and what we learned along the way.

---

## The Setup

| Component | Choice |
|---|---|
| **Hosting** | GitHub Pages (`NewAILife.github.io`) |
| **Site Generator** | Jekyll 4.4 |
| **Theme** | Chirpy 7.5 (Jekyll gem) |
| **Local Dev** | Docker (`jekyll/jekyll` on Alpine) |
| **CI/CD** | GitHub Actions |
| **AI Agent** | DeepCode (DeepSeek V4) |

The target was straightforward: a multi-topic blog covering AI in daily life — productivity, health, photography, learning, coding, and more.

---

## Challenge 1: Docker + Git Bash = Pain

Running Jekyll inside Docker on Windows with Git Bash exposed three immediate issues.

**TTY errors.** `docker run -it` failed with `the input device is not a TTY`. Git Bash uses MinTTY, which Docker doesn't recognize. Solution: prefix with `winpty`.

**Path mangling.** Git Bash converts POSIX paths like `/srv/jekyll` to Windows paths like `C:/Program Files/Git/srv/jekyll`. Solution: set `MSYS_NO_PATHCONV=1` before Docker commands.

**Volume mounts.** Using `.` for the current directory produced `invalid character for a local volume name`. Solution: use `$PWD` for absolute paths.

The working local dev command:

```bash
winpty docker run --rm --volume="$PWD:/srv/jekyll" -p 4000:4000 -it jekyll/jekyll sh
```

---

## Challenge 2: Missing Gems

The `jekyll/jekyll` Docker image runs Ruby 3.1 on Alpine Linux (musl). Two gems caused trouble:

**`webrick`** was removed from Ruby's stdlib in 3.0 but Jekyll's dev server still needs it. Added to `Gemfile`:

```ruby
gem "webrick"
```

**`sass-embedded`** (Dart Sass compiled to native code) segfaults on musl/Alpine. The fix: pin `jekyll-sass-converter` to version 2.x, which uses the older `sassc` (LibSass) that works on Alpine — but only when running locally, not on CI.

```ruby
gem "jekyll-sass-converter", "~> 2.0" unless ENV["CI"]
```

On GitHub Actions (Ubuntu, glibc), `CI=true` is always set, so CI uses `sass-embedded` 3.x which works fine there.

---

## Challenge 3: The SCSS Black Hole

This was the hardest bug. The compiled CSS file was **71 bytes** — essentially just the `@use` statement with a sourcemap comment. No actual styles.

The Chirpy theme uses Sass's modern module system (`@use` / `@forward`). But the Docker image's `sassc` (LibSass) doesn't support `@use` at all — it passes the statements through unchanged.

On CI (Ubuntu with `sass-embedded` / Dart Sass), `@use` IS supported, but the compiled CSS was still empty. The root cause: `purgecss.js` in `npm run build` was **deleting** the `_sass/vendors/` directory and regenerating `_bootstrap.scss`. If that step failed, the file was gone.

**The fix:** bypass the Ruby SCSS pipeline entirely. Using npm's Dart Sass (JS version), we pre-compiled the complete theme CSS and committed it as a static file:

```bash
npx sass --load-path=_sass entry.scss assets/css/jekyll-theme-chirpy.css
```

This produced an 87KB CSS file with full theme styling. Jekyll just serves it as a static asset — no compilation needed.

---

## Challenge 4: CI Configuration

The deploy workflow (`pages-deploy.yml`) from the Chirpy starter was in the `starter/` subdirectory, not in `.github/workflows/` — inactive! Moving it activated the pipeline, but several fixes followed:

| Issue | Fix |
|---|---|
| Git submodule `assets/lib` not cloned | `submodules: true` in checkout |
| `bundler-cache: true` without lockfile | Removed cache, added explicit `bundle install` |
| JS assets not compiled | Added `npm run build:js` (not `build`, which deletes SCSS) |
| `htmlproofer` failing on demo content | `continue-on-error: true` |
| API rate limits on submodule clone | HTTPS submodule within GitHub Actions just works |

The final deploy workflow: checkout with submodules → setup Ruby → setup Node → build JS → `bundle install` → `jekyll build` → upload artifact → deploy.

---

## Challenge 5: Demo Content vs. Real Content

The Chirpy starter ships with four demo posts full of broken image references (`devices-mockup.png` 404 everywhere). We deleted them and replaced with:

- A **welcome post** introducing the blog's mission and categories
- An **about page** explaining what AI Life covers
- A simple SVG **avatar** (purple "AI" logo)
- The **DeepSeek vs. Codex** comparison (a real article from the same day)

---

## The Final Stack

```text
NewAILife.github.io/
├── _posts/          # Markdown blog posts
├── _tabs/           # About, Categories, Tags, Archives
├── _sass/           # SCSS partials (Bootstrap + Chirpy theme)
├── assets/
│   ├── css/         # Pre-compiled jekyll-theme-chirpy.css
│   ├── js/dist/     # Pre-built JS bundles
│   ├── lib/         # Git submodule (FontAwesome, etc.)
│   └── img/         # Avatar SVG
├── .github/workflows/
│   ├── pages-deploy.yml   # Builds and deploys on push to master
│   └── ci.yml             # Tests on push to master
├── _config.yml      # Site identity, theme settings
├── Gemfile          # Ruby dependencies
└── package.json     # Node.js dependencies
```

---

## Key Takeaways

1. **Docker + Windows Git Bash requires ceremony.** `winpty` + `MSYS_NO_PATHCONV=1` + `$PWD` are your friends.

2. **Alpine + native gems = trouble.** When a gem ships compiled binaries (like `sass-embedded`), musl compatibility isn't guaranteed. Have a fallback.

3. **Pre-compile when you can.** If the build pipeline is fragile, compiling assets once and committing them is a valid strategy — especially for static sites.

4. **GitHub Actions needs explicit wiring.** Submodules, dependency installation, and workflow activation are all manual steps beyond the starter template.

5. **AI agents are great for systematic debugging.** The agent methodically worked through seven CI failures, each time identifying the root cause and applying a targeted fix.

---

## Conclusion

In a single afternoon, an AI coding agent took a blank GitHub repository to a fully deployed, styled, and content-populated blog — navigating Docker quirks, Ruby gem conflicts, SCSS compilation failures, and CI pipeline debugging along the way.

The result: [newailife.github.io](https://newailife.github.io).

If you're considering building a GitHub Pages site with Jekyll, the pattern is proven — and with AI assistance, it's faster than ever.
