# Chirpy Theme Migration — Design

**Date:** 2026-04-28
**Repo:** urklc/urklc.github.io
**Status:** Approved (pending user review of this document)

## Goal

Migrate the existing Jekyll blog from `jekyll-theme-cayman` to `jekyll-theme-chirpy`, in-place in the existing repo, and expand the site from a blog into a portfolio + blog. Preserve git history, the custom domain (`urklc.github.io` via CNAME), Google Analytics, and existing post URLs.

## Decisions (locked)

- **Multilingual:** Single combined feed. No future Turkish posts. Existing Turkish posts kept in the archive.
- **Deployment:** GitHub Actions (Chirpy is not in GitHub Pages' default supported gem list). Pages Source switches from "Deploy from a branch" → "GitHub Actions".
- **URL preservation:** Keep current `permalink: /:title/`. No URL changes for existing posts; no redirects needed.
- **Sidebar tabs:** Home, Posts (Chirpy's "Archives" tab, relabelled), Apps, YouTube, About. Categories and Tags tabs are disabled.
- **Comments:** Disabled (Disqus shortname has always been empty). Easy to enable later.
- **Avatar:** Download current GitHub-hosted avatar to `assets/img/avatar.jpg` and serve locally.
- **Location:** London. Timezone `Europe/London`. Location surfaced in the sidebar tagline (`iOS Engineer · London`) and on the About page.
- **YouTube channel:** `https://www.youtube.com/@urklc`.

## Scope — what changes, what stays

### Files removed (Cayman-era leftovers)

- `_layouts/{default,home,page,post}.html`
- `_includes/{analytics,disqus,meta,site-header,social-metatags,svg-icons}.html`
- `_sass/`
- `assets/` (current Cayman assets — replaced with Chirpy starter's `assets/`)
- `404.md` (replaced by Chirpy starter's `404.html`)
- `index.md` (replaced by Chirpy starter's `index.html`)
- `turkce/_posts/` and `english/_posts/` directories (after content is moved to `_posts/` at repo root)
- `sik-sorulan-sorular/` (Cayman-era folder; verify empty/orphaned before deleting — flag if it holds something we need)

### Files added

- `Gemfile` — rewritten for Chirpy
- `_config.yml` — rewritten in Chirpy's format (see "Configuration")
- `index.html` — from chirpy-starter (one-liner with `layout: home` frontmatter)
- `404.html` — from chirpy-starter
- `_data/contact.yml` — sidebar contact links (GitHub, LinkedIn, YouTube, RSS)
- `_data/locales/en.yml` — minimal override file containing only the relabel of "Archives" → "Posts"
- `_tabs/about.md` — About page (avatar, bio, location, links)
- `_tabs/apps.md` — Apps portfolio page (see "Apps tab content")
- `_tabs/youtube.md` — YouTube portfolio page (channel link + description)
- `_tabs/archives.md` — copied from chirpy-starter so we control the sidebar order (label still resolves through `_data/locales/en.yml`)
- `_plugins/posts-lastmod-hook.rb` — chirpy-starter's auto last-modified hook
- `.github/workflows/pages-deploy.yml` — Chirpy starter's deploy workflow
- `tools/run.sh`, `tools/test.sh` — Chirpy starter helpers (optional but useful for local dev)
- `assets/` — chirpy-starter's `assets/` tree (favicons, etc.)
- `assets/img/avatar.jpg` — local copy of the avatar
- `assets/img/apps/` — local copies of the four App Store icons (one per app)

### Files retained

- `CNAME`, `.gitignore`, `LICENSE`
- `README.md` — refresh content but keep the file
- `images/` — kept at root so existing posts that reference `/images/...` continue to work. New posts may use Chirpy's convention `assets/img/` if desired, but no migration of existing image references is required.
- All published posts (migrated, see "Post migration")

## Post migration

- Move every file from `turkce/_posts/` and `english/_posts/` to `_posts/` at repo root.
- Add `categories: [Türkçe]` to the four Turkish posts and `categories: [English]` to the English post in their frontmatter, so they remain filterable even though there's no public Categories tab. (Tags/categories still work for personal organization and for archive grouping.)
- Normalize filenames to `YYYY-MM-DD-slug.md` (zero-padded). Specifically: `2023-6-11-wwdc2023-onerileri.md` → `2023-06-11-wwdc2023-onerileri.md`. Other filenames already conform.
- Each post keeps its existing slug, so URLs are unchanged under the global `permalink: /:title/`.
- No changes to image references inside posts; the `images/` folder stays where it is.

## Sidebar tabs

Final order in the sidebar:

1. **Home** — built-in (the `index.html` page with `layout: home`); no tab file.
2. **Posts** — Chirpy's archive view. We ship our own `_tabs/archives.md` (copied from chirpy-starter) so we control its sidebar order. The "Archives" label is overridden to "Posts" in `_data/locales/en.yml`:
   ```yaml
   tabs:
     archives: Posts
   ```
3. **Apps** — `_tabs/apps.md`, icon `fas fa-mobile-screen`.
4. **YouTube** — `_tabs/youtube.md`, icon `fab fa-youtube`.
5. **About** — `_tabs/about.md`, icon `fas fa-info-circle`.

`order:` values in each `_tabs/*.md` frontmatter will be set during implementation (sequential integers) to produce the order above.

**Categories and Tags are disabled** by not shipping `_tabs/categories.md` or `_tabs/tags.md`. Posts can still carry `tags:` and `categories:` in their own frontmatter for personal organization — they just don't get public listing pages.

## Apps tab content

`_tabs/apps.md` lists the four apps from developer ID `405878985`, newest first, with a 64×64 local icon and a link to the App Store page. No description line for now (will be added later). Format:

```markdown
---
icon: fas fa-mobile-screen
order: 3
title: Apps
---

## [KidStory AI](https://apps.apple.com/us/app/kidstory-ai/id6475002611)
![KidStory AI](/assets/img/apps/kidstory-ai.png){: w="64" h="64" }

## [Why Delete? – Compress Photos](https://apps.apple.com/us/app/why-delete-compress-photos/id6708242452)
![Why Delete?](/assets/img/apps/why-delete.png){: w="64" h="64" }

## [Secret Photo Vault – Pripho](https://apps.apple.com/us/app/secret-photo-vault-pripho/id1459016776)
![Secret Photo Vault – Pripho](/assets/img/apps/pripho.png){: w="64" h="64" }

## [Drop Drop Bomb](https://apps.apple.com/us/app/drop-drop-bomb/id909917663)
![Drop Drop Bomb](/assets/img/apps/drop-drop-bomb.png){: w="64" h="64" }
```

Icons are downloaded from Apple's CDN at build time (one-off, committed to the repo) at the 512×512 source URL, then resized/compressed to 128×128 PNGs and stored in `assets/img/apps/`.

## YouTube tab content

`_tabs/youtube.md` — a small page that:

- Shows a brief intro line about the channel (Swift programming, Turkish shorts/playlists).
- Links to `https://www.youtube.com/@urklc` as the primary CTA.
- Optional: 1-2 embedded featured videos. Out of scope for the initial migration; can be added later by hand.

## About tab content

`_tabs/about.md` — Chirpy renders the avatar and tagline in the sidebar from `_config.yml`. The About page itself is a longer-form bio. Initial content:

- One-paragraph bio (iOS Engineer based in London).
- Links: GitHub, LinkedIn, YouTube.
- Note about the apps and the blog.

## Configuration

`_config.yml` (key fields):

```yaml
theme: jekyll-theme-chirpy
lang: en
timezone: Europe/London

title: Uğur Kılıç
tagline: "iOS Engineer · London"
description: "iOS Engineer based in London. Notes on Swift, iOS, and indie apps."

url: "https://urklc.github.io"
baseurl: ""

permalink: /:title/
paginate: 10

avatar: /assets/img/avatar.jpg

social:
  name: Uğur Kılıç
  links:
    - https://github.com/urklc
    - https://www.linkedin.com/in/ugurkilic
    - https://www.youtube.com/@urklc

github:
  username: urklc

analytics:
  google:
    id: G-XQE7W2KV3T

comments:
  active: ""    # disabled

theme_mode:    # auto (system); user can toggle in the UI
```

`_data/contact.yml` — controls the contact icon row in the sidebar:

```yaml
- type: github
  icon: fab fa-github
- type: linkedin
  icon: fab fa-linkedin
- type: youtube
  icon: fab fa-youtube
- type: rss
  icon: fas fa-rss
```

`Gemfile`:

```ruby
source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.0"

group :test do
  gem "html-proofer", "~> 5.0"
end

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
```

Chirpy depends on `jekyll-paginate`, `jekyll-redirect-from`, `jekyll-seo-tag`, `jekyll-archives`, `jekyll-sitemap`, and `jekyll-feed` transitively — they are not listed explicitly.

## Deployment

`.github/workflows/pages-deploy.yml` is copied verbatim from `cotes2020/chirpy-starter`. Behavior:

- Triggers on push to `master` and on `workflow_dispatch`.
- Job `build`: checks out (with submodules: recursive, since Chirpy ships its CSS via the gem; submodules flag is harmless here), sets up Ruby, runs `bundle install`, runs `bundle exec jekyll b -d "_site"` with `JEKYLL_ENV: production`, runs `bundle exec htmlproofer _site` for link/image validation, uploads the artifact via `actions/upload-pages-artifact`.
- Job `deploy`: needs `build`; deploys via `actions/deploy-pages@v4`.

**One-time GitHub repo settings change** (manual, by user):
1. Settings → Pages → Source: **GitHub Actions**.
2. Settings → Actions → General → Workflow permissions: **Read and write**.

## Out of scope (YAGNI)

- **Courses tab / LemonSqueezy integration** — explicitly deferred. Will be added later as a new `_tabs/*.md` when the user has paid content live.
- **i18n plugin / multilingual routing** — the user is consolidating to English-only going forward; no `jekyll-polyglot` or fork.
- **Comments engine** (Disqus, Giscus, Utterances) — not enabled. Config field left blank, easy to fill in later.
- **Custom theme color / palette overrides** — using Chirpy defaults.
- **Featured posts / pinned post list customization** — using Chirpy defaults.
- **Embedded YouTube video grid on the YouTube tab** — initial version is a link-out only.
- **Migration of `images/` to `assets/img/`** — leaving image references unchanged in existing posts.
- **Refactoring of existing post content** — purely a theme/structure migration; post bodies are not edited beyond frontmatter (categories field).

## Verification checklist

After implementation, the migrated site should:

- Build successfully via `bundle exec jekyll build` locally and via the GitHub Actions workflow.
- Pass `htmlproofer` (no broken internal links/images).
- Show all 5 existing posts on the Home tab in reverse chronological order, at their original `/<slug>/` URLs.
- Show the **Posts** tab (relabeled Archives) with all posts grouped by year.
- Show **Apps** tab with the four apps, each with a 64×64 icon, name as a link to the App Store page.
- Show **YouTube** tab with the channel link.
- Show **About** tab with bio, location, and social links.
- Sidebar shows avatar, name, tagline ("iOS Engineer · London"), and contact icons (GitHub, LinkedIn, YouTube, RSS).
- Categories and Tags tabs are not present in the sidebar.
- Google Analytics tag (`G-XQE7W2KV3T`) is present in the rendered HTML.
- RSS feed and sitemap are generated.
- The custom domain (`urklc.github.io`) continues to resolve to the deployed site (CNAME preserved).

## Open items the user can resolve later

- Personal descriptions per app (one or two lines each).
- About page bio copy (will be drafted during implementation; user to refine).
- Whether to enable a comments engine (Giscus recommended if/when wanted).
- Whether to embed featured YouTube videos on the YouTube tab.
- Future Courses tab when LemonSqueezy content is ready.
