# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Static marketing website for **神码印材 / Shenma Printing Materials** (Guangzhou Summa New Material Technology Co., Ltd.) — digital-printing coatings, digital materials, and flexo accessories. Pure HTML/CSS/JS with no build step, no framework, no package manager, and no tests.

## Viewing the site

There is no build, lint, or test command. The pages are self-contained and open directly from disk:

```bash
# English page (primary deliverable)
start index.html          # or open in a browser

# Chinese page
start home.html

# Optional local server (for testing relative paths / assets)
python -m http.server 8080
```

`index.html` and `home.html` are **independent, standalone pages** — they do not link to each other and there is no language switcher. Treat them as two separate deliverables.

## Architecture

Two single-file pages, each with all CSS and JS inlined in one `<style>` and one `<script>` block (no external JS/CSS except the CDN fonts and Three.js listed below):

| File | Language | Design | Sections |
|------|----------|--------|----------|
| `index.html` | English (`lang="en"`) | "Ink & Galaxy" — CMYK-on-coated-paper theme, dark hero with WebGL galaxy field | hero, strategies, coating, materials, gallery, accessories, advantages, about, contact |
| `home.html` | Chinese (`lang="zh-CN"`) | Older blue corporate theme (`--primary:#2C5F9E`) | hero, about, business, products (tabbed), advantages, contact |

`index.html` is the current/most-polished page and the one most work targets. It loads:

- **Google Fonts** — Bricolage Grotesque (display), Archivo (body), Space Mono (mono/labels).
- **Three.js r128** from cdnjs (`https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`) — drives the hero `<canvas id="heroCanvas">` galaxy/starfield in `startGalaxy()`. If `THREE` is undefined or WebGL fails, the canvas silently falls back to nothing, so the page must not depend on it.

The inline JS in `index.html` is a single IIFE handling: mobile nav toggle, `IntersectionObserver` scroll-reveal (`.reveal` → `.is-in`), count-up stats (`.num .val[data-count]`, respects `prefers-reduced-motion`), a `mailto:`-based contact form (`#contactForm` → `info@whatsuma.com`, no backend), and the Three.js galaxy field.

## Design tokens & conventions

`index.html` defines all palette/type/spacing as CSS custom properties in `:root` (lines ~18–49): CMYK primaries (`--cyan/--magenta/--yellow`), a violet/orange secondary, `--title-grad` gradient, `--font-display/--font-body/--font-mono`, `--gutter`, `--radius:4px`, and custom easing curves (`--ease-out`, `--ease-ink`). Match existing tokens rather than introducing raw values.

The signature glyph is the **four-colour CMYK registration mark** (`.regmark`, inline SVG with overlapping C/M/Y circles and a cross) — reused in the hero, nav brand, and footer.

## Image localization convention

Images are split by locale:

- `images/` — Chinese/shared assets (PNG/JPG), used by `home.html` and for shared items like `album-*.jpg`, `desk.png`, `logo.png`.
- `images/en/` — English-localized equivalents, named `*-en.*`, kept in **both** large PNG and compressed `.webp` variants. `index.html` references the `.webp` files.

When localizing a graphic, mirror the source `images/<name>.png` as `images/en/<name>-en.webp` and keep a matching `-en.png` alongside it. There is no image pipeline — WebP conversion is manual (see the Python/PIL commands in `.claude/settings.local.json`).

## Visual verification workflow

`.claude/settings.local.json` pre-approves the established way of checking rendered output: headless Chrome screenshots of `index.html` plus PIL image inspection. Example (already allow-listed):

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu \
  --hide-scrollbars --window-size=1440,900 \
  --screenshot="D:/项目开发/神码印材/神码印材网站/_shot_hero.png" \
  --virtual-time-budget=7000 \
  --user-data-dir="D:/项目开发/神码印材/神码印材网站/_chrometmp" \
  "file:///D:/项目开发/神码印材/神码印材网站/index.html"
```

`python -c "from PIL import Image; ..."` is then used to resize/convert the screenshot to JPG for review. `npx playwright *` is also allow-listed for browser automation. Temporary artifacts (`_shot_*.png`, `_shot_*.jpg`, `_chrometmp/`) are disposable.

## Source content

`docs/神码印材 公司介绍2025（中文版）(3).pptx` is the source company-introduction deck (Chinese). Company facts, product categories (coatings / digital materials / flexo parts), and contact details in the pages should stay consistent with it. Python-pptx is available for reading it.
