# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page personal portfolio site for Daniel Baez (CTO). Deployed as a static site to GitHub Pages with a custom domain (`CNAME` → `danielbz.com`).

## Architecture

**Everything lives in one file: `index.html`.** CSS is in a single `<style>` block at the top; JS is in a single `<script>` block at the bottom. There is no build step, no framework, no `package.json`, and no dependencies beyond the Satoshi font loaded from `fonts.cdnfonts.com`. This is deliberate — keep it this way unless the user explicitly asks to introduce tooling.

Treat the inline CSS/JS as if they were separate files: edit them in place, don't extract them.

The one exception to "one file" is the small inline `<script>` in `<head>` (`index.html:31`) that adds `.js` to `<html>`. It must stay in `<head>` and stay synchronous — see the reveal contract below.

### Layout & design tokens

- All design tokens live in `:root` (`index.html:87`): colors, `--radius`, easing curves, and `--col: 660px`.
- `--col` is the canonical content width — it controls the nav inner max-width and every section's column. Change it in one place to reflow the whole page.
- Use existing tokens before introducing new colors or hard-coded hex values.

### Dark mode contract

Dark mode follows the OS via `@media (prefers-color-scheme: dark)` (`index.html:122`). There is no toggle and no JS involved.

**The rule: that media block redefines tokens and nothing else.** No rule elsewhere in the stylesheet should need a scheme-specific override. When adding a color, add a token — never a literal hex or `rgba()` in a rule body. Tokens that exist specifically so this holds:

- `--surface-hover` (card hover) and `--surface-2` (ghost/icon button hover) — two distinct hover greys.
- `--border-2` — the stronger border used on hover.
- `--btn-bg` / `--btn-bg-hover` / `--btn-fg` — the solid button inverts between schemes, so it cannot reuse `--text`.
- `--nav-bg`, `--pill-hover`, `--pill-active` — translucent, so they can't derive from an opaque token.
- `--hero-glow-1` / `--hero-glow-2` — the two hero radial gradients.
- `--accent-rgb` — a bare `R, G, B` triplet. Every accent tint is written `rgba(var(--accent-rgb), <alpha>)` so tints track the accent in both schemes. Keep it in sync with `--accent`.

Also keep the two `theme-color` meta tags (`index.html:10-11`) matching `--bg` in each scheme.

### Sections, reveal, and the scroll-spy contract

The page is a vertical stack of `<section>` elements inside `<main>`. Two `IntersectionObserver`s at the bottom of the file drive the interactive behavior:

1. **`.reveal`** — fades elements in on scroll. Add `.reveal` to any element you want animated, and optionally `.reveal-delay-1` … `.reveal-delay-4` to stagger siblings.

   The hidden state is `.js .reveal`, not `.reveal`. Without the `.js` class (JS disabled or blocked) every element is visible by default. **Never move the hidden state onto bare `.reveal`** — that makes the whole page depend on script to be readable. The observer also bails out and reveals everything if `IntersectionObserver` is missing or the user prefers reduced motion.

2. **`.scroll-section`** — marks a section as an anchor target (it supplies `scroll-margin-top` for the fixed nav) and makes it *eligible* for the active-nav highlight.

   The spy observes only sections whose `id` has a matching `<a href="#id">` in `.nav-links`; unlinked ones (`hero`, `casestudy`) are filtered out. It tracks visible sections in a set and highlights the topmost one, and when no spied section is in the trigger band it keeps the last highlight rather than clearing it. Both behaviors exist to stop the nav from blanking out — don't "simplify" the observer back into toggling every link off per entry.

**Adding a new top-level section requires three coordinated edits:**
1. `<section class="scroll-section" id="newid" aria-label="…">` in `<main>`.
2. A matching `<a href="#newid">…</a>` in `.nav-links` (`index.html:832`) — without this the section still scrolls correctly, but it will not be spied and will never highlight.
3. `.reveal` (and delay classes) on the children you want animated.

### Reduced motion

`@media (prefers-reduced-motion: reduce)` (`index.html:193`) flattens transitions/animations, disables smooth scrolling, and forces `.reveal` elements visible. Any new animation is covered by the blanket rule there; new *transform* effects on `:active` may need an explicit reset like the existing button rule.

### Metadata

`<head>` carries Open Graph tags and a `Person` JSON-LD block (`index.html:13-81`). They are hand-maintained duplicates of on-page content: **when the headline, role, employer, education, or certifications change on the page, update the JSON-LD and `og:description` to match.** There are intentionally no Twitter card tags.

### Assets

- `assets/images/og-card.jpg` — the 1200×630 social share card referenced by `og:image` and the JSON-LD `image`. It is a static, hand-built asset (Satoshi, site background and accent, headline + name + domain). If the hero headline changes materially, rebuild the card at the same dimensions and filename so the URL stays stable.
- The favicon is at the repo root (`favicon.ico`), not under `assets/`, because GitHub Pages serves it from `/`.

## Running locally

No build/test/lint tooling. To preview, serve the directory with any static server, e.g.:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000`. Opening `index.html` directly via `file://` mostly works but some browser behaviors (e.g. the font CDN over relative URLs) are more reliable behind a server.

## Deployment

Pushing to `main` publishes to GitHub Pages. The `CNAME` file pins the custom domain — do not delete it when committing changes, or the custom domain mapping breaks.
