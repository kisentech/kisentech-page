# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static business website for **kisentech**, a sole proprietorship offering CMS and software development services. No build step, no package manager, no framework — plain HTML, CSS, and JS.

Production: <https://kisentech.net> (Cloudflare Workers static assets).

## Files

| File | Purpose |
|---|---|
| `index.html` | English version of the site — served at `/` |
| `ja.html` | Japanese version of the site — served at `/ja` |
| `kisentech-logo.svg` | Wordmark (dark fill `#231815`) — inlined into both HTML files |
| `kisentech-icon.svg` | Square "k." mark — favicon. Cropped tighter than the 400×400 export and theme-adaptive; see Favicon below |
| `kisentech-icon.png` | Square 400×400 mark — `og:image`/`twitter:image`, JSON-LD `logo`, and PNG favicon fallback |
| `GitHub_Invertocat_*.svg`, `InBug-White.png`, `LI-In-Bug.png` | Social icons, dark/light pairs |
| `wrangler.jsonc` | Cloudflare Workers deploy config |
| `.assetsignore` | Files excluded from the deploy |

## Running the site

```bash
npx serve .   # prints the local URL
```

`serve` does clean URLs, matching production: `/` serves `index.html`, `/ja` serves `ja.html`.

Use it rather than `open index.html` (file://) or `python3 -m http.server` — the language switcher uses absolute paths (`/ja`, `/`), which 404 or fail under both. Everything else on the page renders fine under file://.

## Deployment

Cloudflare Workers static assets, configured by `wrangler.jsonc` (`assets.directory: "./"`). `.assetsignore` keeps docs and config out of the deploy.

Three things that are easy to get wrong:

- **Clean URLs are automatic.** `/index.html` 307-redirects to `/`, `/ja.html` to `/ja`. Link to `/` and `/ja` — never the `.html` forms, or every internal navigation takes a redirect hop.
- **`robots.txt` is Cloudflare-managed and is NOT in this repo.** It is injected at the edge and blocks AI *training* crawlers (GPTBot, ClaudeBot, CCBot, Google-Extended, and others) while allowing search and AI *retrieval* bots. Adding a static `robots.txt` here would take over that path and silently delete those blocks. Change the policy in the Cloudflare dashboard instead.
- **Referenced assets must be committed** or they 404 in production while working fine locally. `kisentech-icon.png` is referenced four times per page (favicon, `og:image`, `twitter:image`, JSON-LD `logo`).

## Architecture

### Single-file approach
All CSS and JS live inside each HTML file — no external stylesheets or script files. Both pages are self-contained and intentionally duplicated rather than templated (no build system). If the same section needs updating, edit both files.

### EN and JA are not mirror translations
`index.html` and `ja.html` are **deliberately differentiated** — same person, different audiences. `ja.html` groups experience under フリーランス / 会社員 headings; `index.html` is flat and carries a contract-positioning line the JA page omits; bullet content differs per locale. Do not "fix" this divergence as though it were drift.

hreflang still applies: Google treats localized versions as alternates, not duplicates, so long as the main content is genuinely localized. Titles and descriptions should be *targeted per locale* rather than translated across.

### Metadata that must stay in sync
Both files carry a self-referencing canonical, a reciprocal hreflang cluster (`en`, `ja`, `x-default`), and a JSON-LD `@graph` (`Person` + `Organization`). When editing one file, edit the other:

- hreflang must reciprocate — one-way hreflang is ignored.
- Canonicals are self-referencing (`/` → `/`, `/ja` → `/ja`). Never cross-canonicalize one language to the other; it would drop a page from the index.
- JSON-LD `@id` (`#takuya-kisen`, `#kisentech`) and `sameAs` must be **identical** in both files so the pages consolidate into one entity. `name`, `jobTitle`, and `address` are locale-appropriate and are expected to differ.
- Add nothing to the JSON-LD that isn't visible on the page.

### CSS custom properties (design tokens)
All colors are defined as CSS variables on `:root` and overridden for light mode via `[data-theme="light"]` on the `<html>` element:

```css
:root {
  --bg, --surface, --border, --accent, --accent2, --text, --muted,
  --hero-footer-bg, --experience-bg
}
```

Never use hardcoded color values for new UI — always use the variables so both themes stay in sync.

`#experience` goes full-bleed via `box-shadow: 0 0 0 100vmax` plus `clip-path: inset(0 -100vmax)`. It is fragile — check it visually after changing anything that wraps or contains it.

### Dark/light theme
- Default: `data-theme="dark"` set on `<html>`
- A FOUC-prevention inline `<script>` in `<head>` reads `localStorage.getItem('theme')` and applies it before first paint
- The toggle button (`#themeToggle`) switches the attribute and writes back to `localStorage`
- Theme persists across EN↔JA navigation because both pages share the same `localStorage` key (`theme`)

### Page structure
Fixed `.lang-switch` and `#themeToggle` chrome, then `<main>` wrapping the hero and `#experience`, then `<footer>`. Put new content inside `<main>`; the fixed chrome and the footer stay outside it.

### Logo
The `kisentech-logo.svg` wordmark paths are **inlined** (not `<img src>`) so the fill can be driven by CSS. It appears once per page, in the hero's "Doing business as" line (`.hero-business-name svg`), sized `height: 1em` and filled with `var(--muted)` so it tracks the theme.

The wordmark is 282.43 × 43.78 (~6.45:1) and is not usable as a square icon — the `kisentech-icon.*` files are the square "k." mark.

### Favicon
`kisentech-icon.svg` is a favicon-specific derivative of the Illustrator export, not a copy of it. Two deliberate differences:

- **Cropped tighter.** `viewBox="75 75 250 250"` frames the mark (bounds x 109.9–290.1, y 100–300) with a ~10% margin instead of the 400×400 export frame. The mark fills ~80% of the height rather than ~50%, which is what keeps it legible at 16px. The path data is untouched — only the viewBox differs, so it stays easy to diff against the source.
- **No background rect; the fill follows the theme.** A `prefers-color-scheme` media query swaps the mark between `#231815` and `#ffffff`. This tracks the *browser chrome's* theme, not the site's `data-theme` toggle — correct, since the favicon lives in the tab strip.

Re-exporting from Illustrator will clobber both. Re-apply them, or edit the viewBox and media query back in by hand.

`kisentech-icon.png` stays as the PNG fallback and as the social image — `og:image` cannot be SVG, since the platforms reject it.

Note the business name in that hero line exists only as the SVG's `aria-label`, so it is invisible to text extractors; the brand survives in the `<title>` and the footer only.

### Language switching
- EN page links to `/ja` via the `.lang-switch` pill (top-right); JA page links to `/`
- No client-side routing — two separate files served at clean URLs by Cloudflare

## Key contact info
- Owner email: `takuya@kisentech.net` (used in the mailto CTA on both pages)
- Copyright year: 2026
