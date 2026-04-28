# CLAUDE.md

Notes for future Claude sessions working on this repo.

## What this is

Personal site of Uğur Kılıç. Jekyll + [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy), deployed to GitHub Pages via the `.github/workflows/pages-deploy.yml` workflow on every push to `master`. Custom domain via `CNAME`.

## Local dev

System Ruby (2.6) is too old for Chirpy 7.x. Use Homebrew Ruby:

```bash
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
bundle install
bundle exec jekyll serve
```

Open <http://127.0.0.1:4000>.

GA only injects when `JEKYLL_ENV=production`. To test the production HTML locally:

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

## Posts

- Filenames: `_posts/YYYY-MM-DD-<slug>.md` (zero-padded date).
- URL is derived from the slug under the global `permalink: /:title/`. Older posts intentionally use original capitalization/spaces in filenames to preserve URLs — don't rename them.
- Frontmatter: `layout: post`, `title:`, `categories: [English]` (or `[Türkçe]` for archived TR posts), `author: Uğur Kılıç`.
- Prefer fenced code blocks with a language tag (` ```swift `, etc.) over indented blocks — they get Rouge syntax highlighting.

## Sidebar tabs

Tabs live in `_tabs/`. Categories/Tags are intentionally hidden from the sidebar (the `categories.md` page exists at the repo root, not in `_tabs/`, so `/categories/` resolves but no sidebar entry is generated).

To rename a sidebar entry, override the locale string in `_data/locales/en.yml` (that's how "Archives" is shown as "Posts").

## Deploy

`gh run list --branch master --limit 5` to see recent deploys.

GitHub Pages is configured to deploy via GitHub Actions (not "from a branch"). Don't push directly to a `gh-pages` branch — the workflow handles it.
