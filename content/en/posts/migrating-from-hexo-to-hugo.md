---
title: "Migrating from Hexo to Hugo: A Two-Day Journey with a Custom Theme"
date: 2026-05-19T22:53:44+08:00
aliases: ["/2026/05/19/migrating-from-hexo-to-hugo/"]
draft: false
tags: ['hugo', 'hexo', 'static-site', 'github-pages', 'migration']
categories: ["Tech"]
---

## Introduction

After spending two days migrating my blog from Hexo to Hugo, I'm excited to share my journey. The move wasn't just about switching tools—it was about creating something tailored to my needs. With AI assistance, I designed a custom Hugo theme called [Signal](https://github.com/woojar/hugo-theme-signal) that reflects my style and requirements.

This post covers the migration process, highlights the key differences between Hexo and Hugo, and introduces the Signal theme along with the GitHub Actions deployment workflow.

<!--more-->

## The Signal Theme

The **Signal** theme is a modern Hugo blog theme built specifically for technical writers and engineers. It features:

- **Dark mode** with system-preference detection and manual toggle
- **Inline search** with keyboard shortcut support
- **Reading progress bar** and **back-to-top** button
- **Giscus comments** integration (fully configurable)
- **Social sharing** buttons for Twitter, LinkedIn, Facebook, and copy-link
- **Author bio** section with avatar and social links
- **Related posts** based on tag matching
- **Responsive design** with mobile-first approach
- **Schema.org structured data** for SEO
- **Print styles** and **reduced-motion** support

The theme is published as an open-source project, available via Hugo modules, submodules, or direct clone.

## GitHub Actions Deployment Workflow

The deployment uses GitHub Actions to automatically build and deploy the site on every push to the main branch:

```yaml
name: Deploy Hugo site to Pages
on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: write
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.161.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v4
        with:
          path: ./public
```

The workflow handles Hugo installation, theme submodules, minification, and deployment to GitHub Pages—all automatically.

## Hexo vs Hugo: Key Differences

| Aspect | Hexo | Hugo |
|--------|------|------|
| **Language** | Node.js (JavaScript) | Go |
| **Build Speed** | Slower for large sites | Extremely fast (sub-second builds) |
| **Configuration** | YAML (`_config.yml`) | TOML (`hugo.toml`) or YAML |
| **Templating** | EJS, Swig | Go templates |
| **Plugin System** | Node.js ecosystem | Go modules, shortcodes |
| **Content Structure** | `source/_posts/` | `content/posts/` |
| **Learning Curve** | Moderate | Moderate |
| **Community Themes** | Many | Growing rapidly |

### Why I Chose Hugo

1. **Speed**: Hugo's build times are significantly faster, especially noticeable when working with many posts
2. **Single Binary**: No Node.js dependency management headaches
3. **Built-in Features**: Many features that require plugins in Hexo are built into Hugo
4. **Template Flexibility**: Go templates offer powerful capabilities for custom layouts

## Migration Process

The migration involved:

1. Exporting existing Markdown posts to the new content directory
2. Converting front matter from YAML to TOML format
3. Configuring the new Hugo site with the Signal theme
4. Setting up GitHub Actions for automatic deployment
5. Testing locally and then pushing live

The entire process took approximately two days, with most time spent fine-tuning the theme design and ensuring all features worked correctly.

## Acknowledgments

This migration wouldn't have been possible without the incredible open-source communities behind both Hexo and Hugo:

- **Hexo**: Thank you for powering the first iteration of my blog and providing a solid foundation for static site generation
- **Hugo**: Thank you for the blazing-fast builds and flexible templating system that made this migration worthwhile

Both projects demonstrate the power of open-source software in enabling creators to share their knowledge and stories with the world.

## Conclusion

Migrating from Hexo to Hugo was a rewarding experience that resulted in a faster, more customizable blogging platform. The Signal theme is now available for anyone who wants a clean, technical-focused Hugo theme. The GitHub Actions deployment pipeline ensures that new posts are automatically built and deployed with minimal friction.

If you're considering a similar migration, I highly recommend Hugo for its speed and flexibility. And if you're building something with the Signal theme, I'd love to hear about it!

## Resources

- [Signal Theme Repository](https://github.com/woojar/hugo-theme-signal)
- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hexo Documentation](https://hexo.io/docs/)
- [Live Demo](https://woojar.com)

## Hugo Writing Workflow

### Content Management

Hugo's content structure is straightforward:

```
content/
├── posts/          # Blog posts
├── about/          # Static pages
└── archives/       # List pages
```

Use `hugo new posts/my-article.md` to create new posts with proper front matter. Hugo supports both TOML and YAML front matter formats, making migration from Hexo seamless.

**Draft Management**: Keep posts as drafts during writing by setting `draft: true` in front matter. Use `hugo server -D` to preview drafts locally.

**Content Organization**: Organize posts with tags and categories in front matter. Hugo's section-based architecture makes it easy to group related content.

### Daily Writing Workflow

1. **Create draft**: `hugo new posts/article-title.md`
2. **Write locally**: Use your preferred Markdown editor
3. **Preview**: `hugo server -D` (auto-reload on file changes)
4. **Publish**: Set `draft: false` and push to trigger GitHub Actions

### Recommended Tools

**VS Code** - The most popular choice for Hugo development:
- **Markdown All in One** - Enhanced Markdown editing
- **Hugo Shortcode Syntax Highlighting** - Go template syntax support
- **Front Matter** - YAML/TOML front matter management
- **Live Server** - Preview with auto-reload

**Obsidian** - Excellent for knowledge management and linking:
- Works seamlessly with Hugo's Markdown files
- Graph view helps visualize content relationships
- Templates can standardize front matter across posts

**Terminal Tools**:
- `hugo new` - Create new content
- `hugo server -D` - Local development server with drafts
- `hugo --minify` - Production build

### Front Matter Patterns

**Basic post**:
```yaml
---
title: "My Article"
date: 2026-05-19T10:00:00+08:00
draft: true
tags: ['hugo', 'writing']
categories: ["Technical"]
summary: "Brief description for SEO"
---
```

**Post with featured image**:
```yaml
---
title: "Tutorial Post"
date: 2026-05-19T10:00:00+08:00
draft: false
tags: ['tutorial']
featured_image: "/images/hero.png"
---
```

### Setting Up Obsidian Templates

Obsidian templates streamline Hugo post creation with consistent front matter.

**Step 1: Install Templater Plugin**

1. Open Obsidian Settings → Community plugins
2. Install "Templater" plugin
3. Enable the plugin

**Step 2: Create Template Folder**

Create `templates/` folder in your Obsidian vault for storing templates.

**Step 3: Create Hugo Post Template**

Create `templates/hugo-post.md` with:

```markdown
---
title: "<% tp.file.title %>"
date: <% tp.date.now("YYYY-MM-DDTHH:mm:ss+08:00") %>
draft: true
tags: []
categories: []
summary: ""
---

## Introduction

## Main Content

## Conclusion
```

**Step 4: Configure Template Folder Location**

1. Settings → Templater
2. Set "Template folder location" to `templates`

**Step 5: Use the Template**

1. Create new note in `content/posts/` folder
2. Press `Ctrl/Cmd + P` → "Templater: Insert template"
3. Select `hugo-post.md`
4. Template populates front matter with current date and title

**Advanced Template with Prompts**:

```markdown
---
title: "<% tp.file.title %>"
date: <% tp.date.now("YYYY-MM-DDTHH:mm:ss+08:00") %>
draft: true
tags: ["<% tp.system.prompt("Enter tags (comma-separated)") %>"]
categories: ["<% tp.system.prompt("Enter category") %>"]
summary: "<% tp.system.prompt("Enter summary") %>"
---
```

### Auto-Insert Templates on New File Creation

**Method 1: Folder Templates (Built-in Obsidian)**

1. Settings → Files & Links → "Template file location"
2. Settings → Files & Links → "Folder templates"
3. Add folder rule:
   - Folder: `content/posts/`
   - Template: `hugo-post.md`
4. New files in `content/posts/` automatically get the template

**Method 2: Templater Trigger on New File**

Create `templates/folder-trigger.md` with folder template syntax:

```markdown
<%*
let title = tp.file.title;
if (title.includes("Untitled")) {
  title = await tp.system.prompt("Enter title:");
}
-%>
---
title: "<%* tR += title %>"
date: <% tp.date.now("YYYY-MM-DDTHH:mm:ss+08:00") %>
draft: true
tags: []
categories: []
summary: ""
---

## Introduction

## Main Content

## Conclusion
```

Then in Templater settings:
1. Enable "Trigger Templater on new file creation"
2. Set "User template folder" to `templates`
3. Create a user function (Settings → Templater → User functions) to handle folder detection

**Method 3: Hotkey Auto-Insert**

1. Settings → Hotkeys
2. Search "Templater: Insert template"
3. Assign shortcut (e.g., `Ctrl/Cmd + Shift + N`)
4. Type filename, then press shortcut to auto-insert template

**Pro Tip**: Combine with Obsidian's "QuickAdd" plugin for even faster workflows:
1. Install QuickAdd plugin
2. Create macro: "New Hugo Post"
3. Set command: Create file + insert template
4. Assign macro to a hotkey for one-click post creation