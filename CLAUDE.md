# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static business website for **kisentech**, a sole proprietorship offering CMS and software development services. No build step, no package manager, no framework — open HTML files directly in a browser.

## Files

| File | Purpose |
|---|---|
| `index.html` | English version of the site |
| `ja.html` | Japanese version of the site |
| `kisentech-logo.svg` | Original logo (dark fill `#231815`) — used inlined in both HTML files |

## Running the site

```bash
open index.html   # English
open ja.html      # Japanese
```

Or serve locally to avoid any file:// quirks:

```bash
npx serve .
python3 -m http.server 8000
```

## Architecture

### Single-file approach
All CSS and JS live inside each HTML file — no external stylesheets or script files. Both pages are self-contained and intentionally duplicated rather than templated (no build system). If the same section needs updating, edit both files.

### CSS custom properties (design tokens)
All colors are defined as CSS variables on `:root` and overridden for light mode via `[data-theme="light"]` on the `<html>` element:

```css
:root {
  --bg, --surface, --border, --accent, --accent2, --text, --muted
}
```

Never use hardcoded color values for new UI — always use the variables so both themes stay in sync.

### Dark/light theme
- Default: `data-theme="dark"` set on `<html>`
- A FOUC-prevention inline `<script>` in `<head>` reads `localStorage.getItem('theme')` and applies it before first paint
- The toggle button (`#themeToggle`) switches the attribute and writes back to `localStorage`
- Theme persists across EN↔JA navigation because both pages share the same `localStorage` key (`theme`)

### Logo
The SVG logo paths are **inlined** (not `<img src>`) so fill color can be controlled via CSS:
- Dark mode: `fill: #ffffff`
- Light mode: `fill: #231815` (original ink color)

Logo appears twice per page: small in the nav (`.logo svg`, 22px) and large in the hero (`.hero-logo svg`, 40px).

### Language switching
- EN page links to `ja.html` via `.lang-switch` pill in the nav
- JA page links to `index.html`
- No URL-based routing — just two separate files

## Key contact info
- Owner email: `takuya@kisentech.net` (used in the mailto CTA on both pages)
- Copyright year: 2026
