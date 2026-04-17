# GitHub Pages Starter — Cayman + Floating TOC

A ready-to-copy template for GitHub Pages sites that adds a **sticky, floating Table of Contents sidebar** to every page. Built on the [Cayman theme](https://github.com/pages-themes/cayman) and [jekyll-toc](https://github.com/allejo/jekyll-toc).

---

## Background

[GitHub Pages](https://pages.github.com) lets you host static websites directly from a GitHub repository — no server required. Jekyll is built in, so Markdown files are automatically converted to HTML.

**Official quickstart:** https://docs.github.com/en/pages/quickstart

### What this template adds on top of plain GitHub Pages

| Layer | What it does |
|---|---|
| **Cayman theme** | Clean, responsive layout with a colour banner header |
| **jekyll-toc** | Pure-Liquid snippet that generates a TOC from page headings — no gem or plugin needed |
| **Custom layout + CSS** | Places the TOC in a right-hand sidebar that sticks to the viewport as you scroll |

---

## Quick Start

### 1 — Create a new repository and enable GitHub Pages

1. Create a new GitHub repository.
2. Go to **Settings → Pages**.
3. Under **Source**, choose **Deploy from a branch**.
4. Set the branch (e.g. `main`) and folder (`/docs` or `/root`), then click **Save**.

### 2 — Copy this template into your repo

Place the five files below inside your publishing folder (e.g. `docs/`), keeping the directory structure intact:

```
docs/
├── _config.yml
├── _layouts/
│   └── default.html
├── _includes/
│   ├── toc.html
│   └── head-custom.html
└── assets/css/
    └── style.scss
```

### 3 — Edit `_config.yml`

Open `_config.yml` and fill in the three required fields:

```yaml
title: Your Site Title
description: A short description.
theme: jekyll-theme-cayman
```

If your site lives at `https://USERNAME.github.io/REPO` (a project site, not a user site), also add:

```yaml
baseurl: "/REPO"
url: "https://USERNAME.github.io"
```

Leave `baseurl` blank for a user/org site (`USERNAME.github.io`).

### 4 — Write pages in Markdown

Create `.md` files anywhere in your publishing folder. Each file needs this front matter:

```yaml
---
layout: default
title: My Page Title
description: Optional subtitle shown in the banner.
---

# Heading shown in TOC

Your content here...
```

- The TOC is generated automatically from `#` and `######` headings.
- To adjust heading depth, edit `h_min` / `h_max` in `_layouts/default.html` (line ~45).

### 5 — Push and wait

Push your changes. GitHub Pages rebuilds in about 30–60 seconds. Visit your Pages URL to see the result.

---

## Changing the Theme

Only one line needs to change — update `theme:` in `_config.yml`:

```yaml
theme: jekyll-theme-minimal   # or jekyll-theme-slate, jekyll-theme-hacker, etc.
```

The stylesheet (`assets/css/style.scss`) uses `@import "{{ site.theme }}"`, so it picks up the new theme's base CSS automatically. The sidebar CSS added in that file continues to work regardless of which supported theme you choose.

Supported GitHub Pages themes: https://pages.github.com/themes/

---

## File Reference

| File | Purpose |
|---|---|
| `_config.yml` | Site-wide settings: title, URL, theme |
| `_layouts/default.html` | Page template: banner header + flex layout with main content and TOC sidebar |
| `_includes/toc.html` | jekyll-toc v1.2.1 — generates the TOC list from compiled HTML |
| `_includes/head-custom.html` | Empty placeholder required by the layout |
| `assets/css/style.scss` | Imports the active theme, then adds sticky-sidebar and TOC link styles |

---

## References

- GitHub Pages quickstart: https://docs.github.com/en/pages/quickstart
- Cayman theme: https://github.com/pages-themes/cayman
- jekyll-toc: https://github.com/allejo/jekyll-toc
