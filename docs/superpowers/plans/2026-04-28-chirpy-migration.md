# Chirpy Theme Migration — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrate the Jekyll site from `jekyll-theme-cayman` to `jekyll-theme-chirpy`, in-place, without losing post URLs, the custom domain, GA, or git history. Add Apps and YouTube portfolio tabs. Deploy via GitHub Actions.

**Architecture:** The current repo has Cayman-era custom layouts, includes, SCSS, and split post folders (`turkce/_posts/`, `english/_posts/`, `sik-sorulan-sorular/_posts/`). We delete all Cayman scaffolding, copy Chirpy starter scaffolding (`Gemfile`, `_config.yml`, `index.html`, `404.html`, `_plugins/`, `tools/`, `assets/`, `.github/workflows/pages-deploy.yml`), customize the config and tabs, consolidate posts into `_posts/` at the repo root, then ship via a GitHub Actions workflow.

**Tech Stack:** Jekyll 4.3, jekyll-theme-chirpy ~> 7.2, Ruby (system), GitHub Pages with GitHub Actions deploy.

**Reference spec:** `docs/superpowers/specs/2026-04-28-chirpy-migration-design.md`

---

## Task 1: Create migration branch and verify baseline

**Files:** none modified

- [ ] **Step 1: Verify clean working tree**

Run:
```bash
cd /Users/ugurkilic/Projects/urklc.github.io
git status
```

Expected: `nothing to commit, working tree clean` on `master`. If anything is dirty, stop and surface it to the user.

- [ ] **Step 2: Create migration branch**

Run:
```bash
git checkout -b chirpy-migration
```

Expected: `Switched to a new branch 'chirpy-migration'`

- [ ] **Step 3: Confirm Ruby and Bundler are available**

Run:
```bash
ruby --version && bundle --version
```

Expected: Ruby 3.x or higher, Bundler 2.x. If Ruby is older than 3.0, surface to the user before proceeding (Chirpy 7.x requires Ruby >= 3.0).

---

## Task 2: Remove Cayman leftovers

Wipe the theme-specific files so we start with a clean slate. Posts in `turkce/_posts/`, `english/_posts/`, `sik-sorulan-sorular/_posts/`, the `images/` folder, and `CNAME` are kept (posts will be moved in Task 11).

**Files:**
- Delete: `_layouts/` (entire dir)
- Delete: `_includes/` (entire dir)
- Delete: `_sass/` (entire dir)
- Delete: `assets/css/` (Cayman CSS)
- Delete: `404.md`
- Delete: `index.md`

- [ ] **Step 1: Delete Cayman directories and files**

Run:
```bash
rm -rf _layouts _includes _sass assets/css 404.md index.md
```

- [ ] **Step 2: Verify deletions**

Run:
```bash
ls _layouts _includes _sass assets/css 2>&1; ls 404.md index.md 2>&1
```

Expected: `No such file or directory` for every path.

- [ ] **Step 3: Confirm posts and CNAME still exist**

Run:
```bash
ls turkce/_posts english/_posts sik-sorulan-sorular/_posts CNAME images
```

Expected: posts listed in each `_posts` dir; `CNAME` present; `images/` listed.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "chore: remove Cayman theme scaffolding"
```

---

## Task 3: Add Chirpy starter scaffolding (Gemfile, index, 404, plugins, tools, assets, workflow)

We copy a small set of files from `cotes2020/chirpy-starter` (the official starter template) into the repo. This gives us the bones Chirpy expects.

**Files:**
- Create: `Gemfile`
- Create: `index.html`
- Create: `404.html`
- Create: `_plugins/posts-lastmod-hook.rb`
- Create: `tools/run.sh`
- Create: `tools/test.sh`
- Create: `.github/workflows/pages-deploy.yml`
- Create: `assets/img/favicons/` (and the favicons inside it)
- Create: `assets/lib/` (Chirpy includes vendored libs in starter, but the gem provides them — skip if not present in the starter snapshot)

- [ ] **Step 1: Clone chirpy-starter to a scratch location**

Run:
```bash
git clone --depth 1 https://github.com/cotes2020/chirpy-starter.git /tmp/chirpy-starter
ls /tmp/chirpy-starter
```

Expected: directory contains `Gemfile`, `_config.yml`, `_data/`, `_plugins/`, `_tabs/`, `assets/`, `index.html`, `404.html`, `tools/`, `.github/workflows/`.

- [ ] **Step 2: Overwrite the existing Gemfile**

Replace `Gemfile` with the chirpy-starter version, edited to lock to ~> 7.2.

Write `Gemfile`:
```ruby
# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.2"

group :test do
  gem "html-proofer", "~> 5.0"
end

# Windows and JRuby do not include zoneinfo files
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
```

- [ ] **Step 3: Copy index.html and 404.html**

Run:
```bash
cp /tmp/chirpy-starter/index.html ./index.html
cp /tmp/chirpy-starter/404.html ./404.html
```

- [ ] **Step 4: Copy `_plugins/`**

Run:
```bash
cp -r /tmp/chirpy-starter/_plugins ./_plugins
ls _plugins
```

Expected: `posts-lastmod-hook.rb` listed.

- [ ] **Step 5: Copy `tools/`**

Run:
```bash
cp -r /tmp/chirpy-starter/tools ./tools
chmod +x tools/*.sh
ls tools
```

Expected: `run.sh`, `test.sh` listed and executable.

- [ ] **Step 6: Copy `assets/` from chirpy-starter into our `assets/`**

Run:
```bash
cp -r /tmp/chirpy-starter/assets/. ./assets/
ls assets
```

Expected: at minimum `img/` directory present (for favicons). Some Chirpy starter versions ship vendored libs under `assets/lib/`; copy whatever is there.

- [ ] **Step 7: Copy GitHub Actions workflow**

Run:
```bash
mkdir -p .github/workflows
cp /tmp/chirpy-starter/.github/workflows/pages-deploy.yml .github/workflows/pages-deploy.yml
```

- [ ] **Step 8: Edit `pages-deploy.yml` so it triggers on `master` (chirpy-starter defaults to `main`)**

Open `.github/workflows/pages-deploy.yml`. Find the line:
```yaml
    branches: [main]
```
Replace with:
```yaml
    branches: [master]
```

If the workflow uses `branches:` under `push:` only, just change that one. If it has additional refs, leave them alone.

- [ ] **Step 9: Verify scaffolding is in place**

Run:
```bash
ls Gemfile index.html 404.html _plugins/ tools/ .github/workflows/pages-deploy.yml
```

Expected: all listed without errors.

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "chore: add Chirpy starter scaffolding (Gemfile, index, plugins, tools, workflow)"
```

---

## Task 4: Write the customized `_config.yml`

Replace the Cayman `_config.yml` entirely with a Chirpy-style config.

**Files:**
- Modify: `_config.yml` (overwrite)

- [ ] **Step 1: Overwrite `_config.yml`**

Write `_config.yml` with this exact content:

```yaml
# The Site Configuration

theme: jekyll-theme-chirpy

lang: en
timezone: Europe/London

title: Uğur Kılıç
tagline: "iOS Engineer · London"
description: >-
  iOS Engineer based in London. Notes on Swift, iOS, and indie apps.

url: "https://urklc.github.io"
baseurl: ""

github:
  username: urklc

social:
  name: Uğur Kılıç
  email: ""
  links:
    - https://github.com/urklc
    - https://www.linkedin.com/in/ugurkilic
    - https://www.youtube.com/@urklc

# Site avatar (served from local assets)
avatar: /assets/img/avatar.jpg

# Google Analytics
analytics:
  google:
    id: G-XQE7W2KV3T

# Comments engine — disabled
comments:
  active: ""

# Post and page settings
permalink: /:title/
paginate: 10

# Site theme: 'light', 'dark', or '' (auto)
theme_mode:

# The CDN endpoint for media resources — use local serving
cdn: ""

# the avatar on sidebar, support local file or CORS resources
img_cdn: ""

# Default values for posts
defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: post
      comments: false
      toc: true
  - scope:
      path: ""
      type: tabs
    values:
      layout: page
      permalink: /:title/

sass:
  style: compressed

compress_html:
  clippings: all
  comments: all
  endings: all
  profile: false
  blanklines: false
  ignore:
    envs: [development]

exclude:
  - "*.gem"
  - "*.gemspec"
  - tools
  - README.md
  - LICENSE
  - "*.config.js"
  - package*.json
  - docs

jekyll-archives:
  enabled: [categories, tags]
  layouts:
    category: category
    tag: tag
  permalinks:
    tag: /tags/:name/
    category: /categories/:name/
```

- [ ] **Step 2: Sanity-check the YAML parses**

Run:
```bash
ruby -ryaml -e "YAML.load_file('_config.yml'); puts 'ok'"
```

Expected: `ok`. If YAML errors, fix the file before continuing.

- [ ] **Step 3: Commit**

```bash
git add _config.yml
git commit -m "chore: rewrite _config.yml for Chirpy theme"
```

---

## Task 5: Add `_data/locales/en.yml` (rename "Archives" → "Posts") and `_data/contact.yml`

**Files:**
- Create: `_data/locales/en.yml`
- Create: `_data/contact.yml`

- [ ] **Step 1: Create the locale override**

Run:
```bash
mkdir -p _data/locales
```

Write `_data/locales/en.yml`:
```yaml
# Override the default Chirpy English strings.
# Only keys present here override the gem's defaults.

tabs:
  archives: Posts
```

- [ ] **Step 2: Create the contact data**

Write `_data/contact.yml`:
```yaml
- type: github
  icon: "fab fa-github"

- type: linkedin
  icon: "fab fa-linkedin"

- type: youtube
  icon: "fab fa-youtube"

- type: rss
  icon: "fas fa-rss"
  noblank: true
```

- [ ] **Step 3: Commit**

```bash
git add _data
git commit -m "chore: add Chirpy locale override (Archives -> Posts) and contact links"
```

---

## Task 6: Download avatar to local assets

The current avatar is hosted on GitHub raw. We make a local copy at `assets/img/avatar.jpg`.

**Files:**
- Create: `assets/img/avatar.jpg`

- [ ] **Step 1: Download the avatar**

Run:
```bash
mkdir -p assets/img
curl -sSL "https://raw.githubusercontent.com/urklc/urklc.github.io/master/images/urklc.jpg" -o assets/img/avatar.jpg
file assets/img/avatar.jpg
```

Expected: `assets/img/avatar.jpg: JPEG image data, ...`. If the file isn't a valid JPEG (e.g. an HTML 404 page got saved), stop and surface to the user — the source URL may have changed.

- [ ] **Step 2: Verify size**

Run:
```bash
ls -lh assets/img/avatar.jpg
```

Expected: file size in tens of KB (not zero, not megabytes).

- [ ] **Step 3: Commit**

```bash
git add assets/img/avatar.jpg
git commit -m "chore: add local avatar to assets"
```

---

## Task 7: Create `_tabs/about.md`

Replace the gem-default About page with a customized version.

**Files:**
- Create: `_tabs/about.md`

- [ ] **Step 1: Create the tabs directory and About page**

Run:
```bash
mkdir -p _tabs
```

Write `_tabs/about.md`:
```markdown
---
# the default layout is 'page'
icon: fas fa-info-circle
order: 5
title: About
---

I'm Uğur, an iOS Engineer based in London.

I write Swift, ship indie apps to the App Store, and post (mostly in English, occasionally in Turkish) about things I learn along the way. I also run a [YouTube channel](https://www.youtube.com/@urklc) where I cover Swift programming in Turkish.

You can find me here:

- GitHub — [urklc](https://github.com/urklc)
- LinkedIn — [ugurkilic](https://www.linkedin.com/in/ugurkilic)
- YouTube — [@urklc](https://www.youtube.com/@urklc)
```

- [ ] **Step 2: Commit**

```bash
git add _tabs/about.md
git commit -m "feat(tabs): add custom About page"
```

---

## Task 8: Create `_tabs/youtube.md`

**Files:**
- Create: `_tabs/youtube.md`

- [ ] **Step 1: Write the YouTube tab**

Write `_tabs/youtube.md`:
```markdown
---
icon: fab fa-youtube
order: 4
title: YouTube
---

I publish short videos and playlists about Swift programming on YouTube — mostly in Turkish.

[**youtube.com/@urklc**](https://www.youtube.com/@urklc){: target="_blank" }
```

- [ ] **Step 2: Commit**

```bash
git add _tabs/youtube.md
git commit -m "feat(tabs): add YouTube tab"
```

---

## Task 9: Download App Store icons and create `_tabs/apps.md`

**Files:**
- Create: `assets/img/apps/kidstory-ai.png`
- Create: `assets/img/apps/why-delete.png`
- Create: `assets/img/apps/pripho.png`
- Create: `assets/img/apps/drop-drop-bomb.png`
- Create: `_tabs/apps.md`

- [ ] **Step 1: Make the icons directory**

Run:
```bash
mkdir -p assets/img/apps
```

- [ ] **Step 2: Download each icon at 256×256 (Apple CDN supports size token in URL)**

Run:
```bash
curl -sSL "https://is1-ssl.mzstatic.com/image/thumb/Purple211/v4/df/51/22/df5122b8-c945-0406-5486-765f5df1c3e7/AppIcon-0-0-1x_U007ephone-0-85-220.png/256x256bb.png" -o assets/img/apps/kidstory-ai.png
curl -sSL "https://is1-ssl.mzstatic.com/image/thumb/Purple221/v4/e6/c9/51/e6c951f4-da72-feac-8735-ffda3e483abf/AppIcon-0-0-1x_U007ephone-0-1-85-220.png/256x256bb.png" -o assets/img/apps/why-delete.png
curl -sSL "https://is1-ssl.mzstatic.com/image/thumb/Purple211/v4/52/04/8a/52048aa3-77ae-b60f-5228-3ed0aafc4a9d/AppIcon-0-0-1x_U007emarketing-0-3-0-85-220.png/256x256bb.png" -o assets/img/apps/pripho.png
curl -sSL "https://is1-ssl.mzstatic.com/image/thumb/Purple113/v4/40/02/60/4002606b-470e-3c9f-5988-59ea7e7e1d18/AppIcon-0-1x_U007emarketing-0-0-GLES2_U002c0-512MB-sRGB-0-0-0-85-220-0-0-0-2.png/256x256bb.png" -o assets/img/apps/drop-drop-bomb.png
```

- [ ] **Step 3: Verify all four are valid PNGs**

Run:
```bash
file assets/img/apps/*.png
```

Expected: each line ends with `PNG image data, 256 x 256, ...`. If any is not a PNG, stop and surface.

- [ ] **Step 4: Write `_tabs/apps.md`**

Write `_tabs/apps.md`:
```markdown
---
icon: fas fa-mobile-screen
order: 3
title: Apps
---

A small collection of apps I've shipped on the App Store.

## [KidStory AI](https://apps.apple.com/us/app/kidstory-ai/id6475002611)

![KidStory AI icon](/assets/img/apps/kidstory-ai.png){: w="64" h="64" }

## [Why Delete? — Compress Photos](https://apps.apple.com/us/app/why-delete-compress-photos/id6708242452)

![Why Delete? icon](/assets/img/apps/why-delete.png){: w="64" h="64" }

## [Secret Photo Vault — Pripho](https://apps.apple.com/us/app/secret-photo-vault-pripho/id1459016776)

![Pripho icon](/assets/img/apps/pripho.png){: w="64" h="64" }

## [Drop Drop Bomb](https://apps.apple.com/us/app/drop-drop-bomb/id909917663)

![Drop Drop Bomb icon](/assets/img/apps/drop-drop-bomb.png){: w="64" h="64" }
```

- [ ] **Step 5: Commit**

```bash
git add assets/img/apps _tabs/apps.md
git commit -m "feat(tabs): add Apps page with App Store icons"
```

---

## Task 10: Add Archives tab so we control sidebar order

Chirpy's gem ships an Archives tab; we copy chirpy-starter's `_tabs/archives.md` into our repo so we can place it at `order: 2` and so its label is overridden to "Posts" via our locale file.

**Files:**
- Create: `_tabs/archives.md`

- [ ] **Step 1: Copy starter's Archives tab**

Run:
```bash
cp /tmp/chirpy-starter/_tabs/archives.md _tabs/archives.md
cat _tabs/archives.md
```

Expected: a small file with frontmatter that includes `icon:`, `order:`, and `layout: archives`. (The display title of "Archives" is provided by the layout, not the frontmatter, and our `_data/locales/en.yml` override changes it to "Posts".)

- [ ] **Step 2: Set the order to 2**

Open `_tabs/archives.md`. Find the `order:` line and set it to `2`. (If the chirpy-starter file uses a different order, overwrite it with `2`.)

- [ ] **Step 3: Commit**

```bash
git add _tabs/archives.md
git commit -m "feat(tabs): pin Archives tab at order 2"
```

---

## Task 11: Migrate posts to `_posts/`

Move every post from `turkce/_posts/`, `english/_posts/`, and `sik-sorulan-sorular/_posts/` into `_posts/` at the repo root. Keep filenames byte-for-byte identical except zero-padding single-digit month/day in the date prefix (so URLs are preserved exactly under `permalink: /:title/`).

**Files (move + frontmatter edit):**

Turkish (4 posts), add `categories: [Türkçe]`:
- Move: `turkce/_posts/2023-6-11-wwdc2023-onerileri.md` → `_posts/2023-06-11-wwdc2023-onerileri.md`
- Move: `turkce/_posts/2025-5-23-cursor-izlenimler.md` → `_posts/2025-05-23-cursor-izlenimler.md`
- Move: `turkce/_posts/2026-4-27-claude-izlenimler.md` → `_posts/2026-04-27-claude-izlenimler.md`
- Move: `sik-sorulan-sorular/_posts/2023-05-09-ib-vs-programmatic-ui.md` → `_posts/2023-05-09-ib-vs-programmatic-ui.md`

English (6 posts), add `categories: [English]`:
- Move: `english/_posts/2018-8-20-Storing objects with JSON representation using Swift.md` → `_posts/2018-08-20-Storing objects with JSON representation using Swift.md`
- Move: `english/_posts/2019-11-27-Things to Consider When Developing Push (Remote) Notifications on iOS.md` → `_posts/2019-11-27-Things to Consider When Developing Push (Remote) Notifications on iOS.md`
- Move: `english/_posts/2019-12-26-Using protocols that have associated type requirements as a generic constraint.md` → `_posts/2019-12-26-Using protocols that have associated type requirements as a generic constraint.md`
- Move: `english/_posts/2020-08-11-type erasure.md` → `_posts/2020-08-11-type erasure.md`
- Move: `english/_posts/2020-8-1-opaque-return-types.md` → `_posts/2020-08-01-opaque-return-types.md`
- Move: `english/_posts/2025-02-08-step-by-step-guide-to-sendable-in-swift-6.md` → `_posts/2025-02-08-step-by-step-guide-to-sendable-in-swift-6.md`

> **Important:** Spaces and capitals in filenames are preserved on purpose. Jekyll's `:title` permalink uses the filename slug verbatim, so changing capitalization or replacing spaces would change the live URL. Don't kebab-case these names.

- [ ] **Step 1: Make `_posts/` and move the Turkish posts (zero-padding dates)**

Run:
```bash
mkdir -p _posts
git mv "turkce/_posts/2023-6-11-wwdc2023-onerileri.md"  "_posts/2023-06-11-wwdc2023-onerileri.md"
git mv "turkce/_posts/2025-5-23-cursor-izlenimler.md"   "_posts/2025-05-23-cursor-izlenimler.md"
git mv "turkce/_posts/2026-4-27-claude-izlenimler.md"   "_posts/2026-04-27-claude-izlenimler.md"
git mv "sik-sorulan-sorular/_posts/2023-05-09-ib-vs-programmatic-ui.md" "_posts/2023-05-09-ib-vs-programmatic-ui.md"
```

- [ ] **Step 2: Move the English posts (zero-padding dates only; keep spaces/caps)**

Run:
```bash
git mv "english/_posts/2018-8-20-Storing objects with JSON representation using Swift.md" "_posts/2018-08-20-Storing objects with JSON representation using Swift.md"
git mv "english/_posts/2019-11-27-Things to Consider When Developing Push (Remote) Notifications on iOS.md" "_posts/2019-11-27-Things to Consider When Developing Push (Remote) Notifications on iOS.md"
git mv "english/_posts/2019-12-26-Using protocols that have associated type requirements as a generic constraint.md" "_posts/2019-12-26-Using protocols that have associated type requirements as a generic constraint.md"
git mv "english/_posts/2020-08-11-type erasure.md" "_posts/2020-08-11-type erasure.md"
git mv "english/_posts/2020-8-1-opaque-return-types.md" "_posts/2020-08-01-opaque-return-types.md"
git mv "english/_posts/2025-02-08-step-by-step-guide-to-sendable-in-swift-6.md" "_posts/2025-02-08-step-by-step-guide-to-sendable-in-swift-6.md"
```

- [ ] **Step 3: Verify all 10 posts are in `_posts/`**

Run:
```bash
ls -1 _posts | wc -l
ls _posts
```

Expected: `10`. All filenames listed.

- [ ] **Step 4: Add `categories:` frontmatter to the 4 Turkish posts**

For each Turkish post in `_posts/`, open the file. Insert a `categories: [Türkçe]` line into the existing YAML frontmatter (between the `---` lines), placed after `title:`. If the post already has a `categories:` field, replace it with `[Türkçe]`.

Files to edit:
- `_posts/2023-06-11-wwdc2023-onerileri.md`
- `_posts/2025-05-23-cursor-izlenimler.md`
- `_posts/2026-04-27-claude-izlenimler.md`
- `_posts/2023-05-09-ib-vs-programmatic-ui.md`

Example — frontmatter should look like:
```yaml
---
layout: post
title: "Claude + Design izlenimlerim"
author: Uğur Kılıç
categories: [Türkçe]
---
```

- [ ] **Step 5: Add `categories:` frontmatter to the 6 English posts**

For each English post in `_posts/`, add `categories: [English]` to the frontmatter (same placement rule).

Files to edit:
- `_posts/2018-08-20-Storing objects with JSON representation using Swift.md`
- `_posts/2019-11-27-Things to Consider When Developing Push (Remote) Notifications on iOS.md`
- `_posts/2019-12-26-Using protocols that have associated type requirements as a generic constraint.md`
- `_posts/2020-08-11-type erasure.md`
- `_posts/2020-08-01-opaque-return-types.md`
- `_posts/2025-02-08-step-by-step-guide-to-sendable-in-swift-6.md`

- [ ] **Step 6: Verify each post has frontmatter with `categories:`**

Run:
```bash
for f in _posts/*.md; do echo "=== $f"; head -10 "$f"; done | grep -E "^(===|categories:)"
```

Expected: every `===` line is followed by a `categories:` line within 10 lines. Eyeball the output to confirm.

- [ ] **Step 7: Remove now-empty source directories**

Run:
```bash
rmdir turkce/_posts turkce english/_posts english sik-sorulan-sorular/_posts sik-sorulan-sorular
ls turkce english sik-sorulan-sorular 2>&1
```

Expected: `No such file or directory` for each.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "refactor(posts): consolidate posts under _posts/ with language categories"
```

---

## Task 12: Refresh README

The current `README.md` is the Jekyll Now boilerplate. Replace with a short site README.

**Files:**
- Modify: `README.md` (overwrite)

- [ ] **Step 1: Overwrite `README.md`**

Write `README.md`:
```markdown
# urklc.github.io

Personal site of Uğur Kılıç — iOS Engineer based in London.

Built with [Jekyll](https://jekyllrb.com) and the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme. Deployed to GitHub Pages via GitHub Actions on every push to `master`.

## Local development

```bash
bundle install
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000>.

## License

Site content: © Uğur Kılıç, all rights reserved.
Theme: MIT (see Chirpy upstream).
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: refresh README for Chirpy"
```

---

## Task 13: Local build and visual smoke test

**Files:** none modified (build + verify)

- [ ] **Step 1: Install dependencies**

Run:
```bash
bundle install
```

Expected: success, no errors. If `bundle install` fails to resolve `jekyll-theme-chirpy ~> 7.2`, surface the resolved error to the user — Chirpy may have moved to a newer major.

- [ ] **Step 2: Build the site**

Run:
```bash
bundle exec jekyll build
```

Expected: build succeeds; output written to `_site/`. There may be Liquid warnings about deprecated tags inside posts — note any but don't treat as failure unless the build itself errors.

- [ ] **Step 3: Verify all 10 posts rendered to their original-style URLs**

Run:
```bash
ls _site | grep -iE "(wwdc2023-onerileri|cursor-izlenimler|claude-izlenimler|ib-vs-programmatic-ui|opaque-return-types|type erasure|step-by-step-guide-to-sendable|Storing objects|Things to Consider|Using protocols)" | sort
```

Expected: 10 directory entries, one per post slug. Filenames containing spaces will be visible (Jekyll preserves them in directory names). If any post is missing, inspect its frontmatter and filename for issues.

- [ ] **Step 4: Verify the sidebar tab structure rendered correctly**

Run:
```bash
ls _site/categories _site/tags 2>&1
ls _site
```

Expected: `_site/categories` and `_site/tags` exist (Chirpy still generates them as content for category/tag-tagged posts). `_site` should contain `index.html`, `404.html`, `posts/` (or similar), `assets/`, and folders for each tab page (`about/`, `apps/`, `youtube/`).

- [ ] **Step 5: Start the dev server**

Run (background):
```bash
bundle exec jekyll serve --host 127.0.0.1 --port 4000
```

- [ ] **Step 6: Open the site in a browser and walk every tab**

Open <http://127.0.0.1:4000>. Verify by eye:
- Home tab lists all 10 posts.
- Sidebar shows: avatar, "Uğur Kılıç", tagline "iOS Engineer · London", and tabs in this order: **Home, Posts, Apps, YouTube, About**. **No** Categories or Tags entries in the sidebar.
- Click each tab; each renders without errors.
- Click into one post (e.g. `/cursor-izlenimler/`); URL matches the slug; content renders.
- Click into a post with spaces in its filename (e.g. the "type erasure" or "Storing objects..." post); URL is correct (spaces encoded as `%20`); content renders.
- Apps tab shows 4 apps with icons; each link goes to the App Store.
- About tab content is correct.
- View page source on the home page; confirm `gtag/js?id=G-XQE7W2KV3T` is present.

- [ ] **Step 7: Stop the dev server**

In the terminal running `jekyll serve`, press `Ctrl+C`.

If anything in Step 6 fails, stop and surface the issue. Do not commit a known-broken state.

---

## Task 14: Run htmlproofer

**Files:** none modified

- [ ] **Step 1: Run htmlproofer against the built site**

Run:
```bash
bundle exec htmlproofer _site \
  --disable-external \
  --ignore-urls "/^http:\/\/127\.0\.0\.1/,/^https:\/\/apps\.apple\.com/" \
  --allow-hash-href
```

Expected: passes with no broken internal links or missing images. (External links to App Store are skipped to avoid rate-limit flakiness.)

- [ ] **Step 2: If errors, fix and re-run**

Common fixes:
- Missing image: confirm the `images/` folder has the referenced asset, or update the post's reference to the new `assets/img/...` path.
- Broken internal link from a Turkish post (`../cursor-izlenimler` etc.): verify the target post URL still resolves.

Re-run htmlproofer until it passes. Do not commit while it's failing.

---

## Task 15: Push branch and request user-side GitHub repo settings change

**Files:** none modified

- [ ] **Step 1: Push the migration branch to origin**

Run:
```bash
git push -u origin chirpy-migration
```

Expected: branch pushed; GitHub URL printed.

- [ ] **Step 2: Pause and ask the user to perform the manual GitHub Pages settings change**

Surface this exact text to the user:

> Two manual changes are needed in GitHub before the site will deploy correctly:
>
> 1. **Settings → Pages → Build and deployment → Source:** change to **GitHub Actions**.
> 2. **Settings → Actions → General → Workflow permissions:** set to **Read and write permissions**, then **Save**.
>
> Confirm both are done, and let me know to proceed.

Wait for user confirmation before continuing.

---

## Task 16: Merge to master and verify live deploy

**Files:** none modified

- [ ] **Step 1: Merge `chirpy-migration` into `master`**

Run:
```bash
git checkout master
git merge --no-ff chirpy-migration -m "feat: migrate site to Chirpy theme"
git push origin master
```

- [ ] **Step 2: Watch the GitHub Actions deploy**

Run:
```bash
gh run watch
```

Or open the repo's **Actions** tab in the browser. Wait for the `pages-deploy.yml` workflow to complete.

Expected: workflow finishes with green status. If it fails, read the failed step's log; common causes are missing repo permission settings (Task 15 not completed) or a `bundle install` resolution error.

- [ ] **Step 3: Verify the live site**

Open <https://urklc.github.io> in a browser. Walk through the same checks as Task 13 Step 6, but on the live URL.

Spot-check at least three pre-existing post URLs that you know external sites might link to. Suggested:
- `https://urklc.github.io/cursor-izlenimler/`
- `https://urklc.github.io/opaque-return-types/`
- `https://urklc.github.io/step-by-step-guide-to-sendable-in-swift-6/`

Each should load with the new Chirpy styling and the original content.

- [ ] **Step 4: Delete the migration branch**

Run:
```bash
git branch -d chirpy-migration
git push origin --delete chirpy-migration
```

- [ ] **Step 5: Summarize the migration to the user**

Tell the user:
- Site is live on Chirpy.
- Existing post URLs are unchanged.
- Apps and YouTube tabs are populated; About is updated; Posts tab replaces Archives.
- Comments engine is disabled; can be enabled later by setting `comments.active` in `_config.yml`.
- Future posts go into `_posts/` at the repo root, named `YYYY-MM-DD-kebab-case-slug.md`, with `categories: [English]` (or other) frontmatter.

---

## Out of scope (not in this plan, by design)

- Courses/LemonSqueezy tab (deferred per spec)
- Comments engine (Disqus/Giscus) (deferred)
- Embedding featured YouTube videos in the YouTube tab
- Migrating `/images/...` references in old posts to `/assets/img/...`
- Theme color/palette overrides

These can be added in follow-up plans.
