# Woojar Blog (woojar/woojar.github.io)

Hugo blog project, CNAME domain: `woojar.com`. What follows should save context on future sessions.

---

## Repository Structure

| Path | Purpose |
|---|---|
| `hugo.toml` | Root Hugo config. Base URL: `https://woojar.com/`, theme: `telecom-tech`. |
| `themes/telecom-tech/` | Custom theme (symlinked/cloned subtree). Theme source lives here; never commit directly upstream. |
| `layouts/` | Project-level layout overrides (`archives/list.html`, `shortcodes/img.html`). |
| `content/posts/` | All blog post markdown files. |
| `static/` | Static assets (images, `CNAME`). |
| `.github/workflows/hugo.yml` | GitHub Actions build-and-deploy workflow (see below). |

### Config highlights (`hugo.toml`)

- Theme: **`telecom-tech`** (custom, under `themes/telecom-tech/`)
- Output formats: `HTML`, `RSS`, `JSON`
- `HUGO_ENVIRONMENT: production` is set in CI; use the same locally when verifying production output.
- Comments: **Giscus** (repo: `woojar/woojar-blog`, category: `General`)
- Parameters used by the theme's partials: `analytics`, `share`, `comments.giscus`, `authorBio`, `newsletter`, `footerTitle`, `footerDescription`, `social`.
- Site menus: Home / Posts / Tags / Categories / Archive / About.

---

## Post Summary Rendering — Important

Hugo generates `.Summary` automatically from the first paragraph(s) of each post's Markdown content.

**All summary templates must use `{{ .Summary | safeHTML }}`**, never `| plainify` or `| markdownify`.

| Template | File | Filter used |
|---|---|---|
| Homepage | `themes/telecom-tech/layouts/index.html:22` | `\| safeHTML` |
| Posts listing / section | `themes/telecom-tech/layouts/_default/list.html:31` | `\| safeHTML` ✅ |
| Tags / categories listing | `themes/telecom-tech/layouts/_default/taxonomy.html:27` | `\| safeHTML` |
| Related posts (single) | `themes/telecom-tech/layouts/_default/single.html:52,75` | bare `{{ .Summary }}` |
| JSON feed | `themes/telecom-tech/layouts/index.json:9` | `\| plainify` (OK — JSON context, not HTML) |

`| plainify` strips all `<p>`, `<ul>`, `<li>` tags from summaries, causing the CSS rules targeting those elements (lines 524–560 of `main.css`) to have nothing to style — summaries collapse into flat plain text. This was the original bug.

`| markdownify` re-processes the already-HTML summary as Markdown again — double-escaping and broken nesting.

---

## Git History — Summary-Related Lessons

| Commit | What |
|---|---|
| `db622ca` | First corrected `list.html` to `safeHTML`. |
| `ef552f7` | Changed to `markdownify` (later found to be wrong). |
| `1992427` | Reverted to `safeHTML` on both `list.html` and `taxonomy.html`. |
| `8c02311` | **Bug introduced**: Changed `list.html` back to `plainify` + added 7-space indentation. |
| `1d1c621` | Reverted `white-space: pre-line` CSS (safe but separate concern). |
| `63dce56` | Added spacing CSS + `div.post-summary-content` block. |
| `7626349` | Added `white-space: pre-line` (later reverted). |

The last "clean" state before the `plainify` bug was at `1992427`.

---

## GitHub Actions Workflow (`.github/workflows/hugo.yml`)

**Current configuration:**

```yaml
on:  push: branches: ["main"]
    workflow_dispatch:

env:
  HUGO_VERSION: 0.161.1          # aligned with local dev version

jobs:
  build:
    steps:
      - Install Hugo CLI (deb from GitHub releases)
      - actions/checkout@v4 (submodules: recursive)
      - actions/configure-pages@v5
      - hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - actions/upload-pages-artifact@v4  # path: ./public (no publish_branch)

  deploy:
    needs: build
    steps:
      - actions/deploy-pages@v5          # pushes artifact to gh-pages
```

**Why v4 + v5 (not v3 + v4):** The `upload-pages-artifact@v3` + `deploy-pages@v5` pair is an unsupported combination. Version 4 of the upload action changed artifact semantics so `deploy-pages@v5` only consumes it correctly when both are at v4/v5 respectively. The `publish_branch: gh-pages` parameter on the old v3 upload action is also no longer needed — the deploy action handles branch routing via the Pages infrastructure configuration in the repo settings.

**Removed steps (unnecessary for this project):**
- `snap install dart-sass` — the theme has zero `.scss`/`.sass` sources; only plain CSS.
- `npm ci` — no `package.json` exists; the step always ran a no-op fallback.

---

## Local Development

| Task | Command |
|---|---|
| Start dev server | `hugo server --buildDrafts` |
| Build production | `HUGO_ENVIRONMENT=production hugo --minify && hugo --gc --minify` |
| Current local Hugo | `v0.161.1+extended` |

### Testing summary rendering locally

Because `hugo server` runs with `--buildDrafts` by default, make sure any draft posts you're testing with are set to `draft: false` in frontmatter, or pass `--buildDrafts` explicitly.

---

## Known Post Patterns

- Short posts (1-3 sentences with no `<p>` tags in summary) — `plainify` and `safeHTML` behave identically; bug is invisible.
- Posts with **bulleted lists, blockquotes, or multi-paragraph** summaries break under `plainify`. Structured content in summaries was the root cause of the discrepancy between local and CI rendering.
- Summary delimiter: Hugo uses the `<!--more-->` tag; if absent, auto-generates from the first 70 words. Both approaches return HTML — must use `safeHTML`.

---

## Theme Customization Checklist

When editing theme files under `themes/telecom-tech/`:
1. **Layouts** — ensure `list.html` uses `\| safeHTML` for `.Summary`.
2. **CSS** — `main.css` lines 477+ target `.post-summary`, `.post-preview`, `.post-summary-content`, `.post-preview-content`. The nested `* + *` sibling selector depends on actual HTML child elements being present.
3. **Partial: `head.html`** — references `.Site.BaseURL`; if baseURL changes, update both `hugo.toml` and the template.
4. **Partial: `comments.html`** — Giscus config; repo ID and category ID are hardcoded.
