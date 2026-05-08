# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page personal portfolio site for Daniel Baez (CTO). Deployed as a static site to GitHub Pages with a custom domain (`CNAME` → `danielbz.com`).

## Architecture

**Everything lives in one file: `index.html`.** CSS is in a single `<style>` block at the top; JS is in a single `<script>` block at the bottom. There is no build step, no framework, no `package.json`, and no dependencies beyond the Satoshi font loaded from `fonts.cdnfonts.com`. This is deliberate — keep it this way unless the user explicitly asks to introduce tooling.

Treat the inline CSS/JS as if they were separate files: edit them in place, don't extract them.

### Layout & design tokens

- All design tokens live in `:root` (`index.html:14-26`): colors (`--bg`, `--surface`, `--border`, `--text*`, `--accent`), `--radius`, easing curves, and `--col: 660px`.
- `--col` is the canonical content width — it controls the nav inner max-width and every section's column. Change it in one place to reflow the whole page.
- Use existing tokens before introducing new colors or hard-coded hex values.

### Sections and the scroll-spy contract

The page is a vertical stack of `<section>` elements inside `<main>`. Two `IntersectionObserver`s at the bottom of the file drive the interactive behavior:

1. **`.reveal`** — fades elements in on scroll. Add `.reveal` to any element you want animated, and optionally `.reveal-delay-1` … `.reveal-delay-4` to stagger siblings.
2. **`.scroll-section`** — drives the active-nav highlight. The observer reads each section's `id` and toggles `.active` on the matching `<a href="#id">` in `.nav-links`.

**Adding a new top-level section requires three coordinated edits:**
1. `<section class="scroll-section" id="newid" aria-label="…">` in `<main>`.
2. A matching `<a href="#newid">…</a>` in `.nav-links` (around `index.html:737`) — without this, the scroll-spy has nothing to highlight.
3. `.reveal` (and delay classes) on the children you want animated.

### Assets

Images go in `assets/images/` (currently mostly empty — `.gitkeep` only). The favicon is at the repo root (`favicon.ico`), not under `assets/`, because GitHub Pages serves it from `/`.

## Running locally

No build/test/lint tooling. To preview, serve the directory with any static server, e.g.:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000`. Opening `index.html` directly via `file://` mostly works but some browser behaviors (e.g. the font CDN over relative URLs) are more reliable behind a server.

## Deployment

Pushing to `main` publishes to GitHub Pages. The `CNAME` file pins the custom domain — do not delete it when committing changes, or the custom domain mapping breaks.
