# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## SafeGuard — Security Hardening Guides

Searchable, bilingual (EN/ES) security hardening guide library at [safeguard.neorgon.com](https://safeguard.neorgon.com/). Modular ES module app, no build step.

**Run:** `python3 -m http.server 8825` from `safeguard-site/` (ES modules require a server, not `file://`). Or `make serve`.

**Live:** safeguard.neorgon.com

## Architecture

Standard modular layout: `index.html` shell + `css/style.css` + `js/*.js`. Entry point is `js/app.js` which calls `loadSaved(state)` → `render()` → `bindEvents()`.

**Two views:** `list` (grid of guide cards with search + category filter pills) and `detail` (TOC sidebar + guide body). View is toggled via `state.view`; navigation uses `history.pushState` with `#guide-id` as the URL hash. Deep links and `popstate` are handled in `events.js`.

**State** (`js/state.js`): `{ lang, view, activeGuide, search, category }`. Only `lang` is persisted to `localStorage` under key `safeguard-state`. The active guide is read from `location.hash` on load.

**All guide content** lives in `js/guides.js` as a static JS array — no external API or data files. Each guide has: `id`, `category`, `icon`, `updated`, `title {en,es}`, `subtitle {en,es}`, and `sections[]`. Each section has `heading`, optional `paragraphs[]`, `steps[]`, `tip`, `warning` — all bilingual `{en, es}` objects.

**i18n** (`js/i18n.js`): UI chrome strings live in `ui` export. The `t(obj, lang)` helper resolves `{en, es}` objects; falls back to `.en` if the language key is missing. Guide content uses the same `t()` helper in `render.js`.

**Rendering** (`js/render.js`): All HTML is built as template literal strings and written via `innerHTML`. The `md()` utility in `utils.js` handles inline markdown (bold `**`, links `[text](url)`) and escapes HTML before parsing. Category accent colors are defined in the `CAT_COLORS` map in `render.js`; icons are inline SVG strings in the `ICONS` map.

## Adding a New Guide

1. Add an entry to the `guides` array in `js/guides.js` with a unique `id`, one of the existing `categories`, and an `icon` key that matches a key in the `ICONS` map in `render.js`.
2. If a new icon is needed, add it to the `ICONS` map in `render.js`.
3. If a new category is needed, add it to `categories` in `guides.js`, add a color entry to `CAT_COLORS` in `render.js`, and add label strings to `ui.categoryLabels` in `i18n.js`.

## Section Schema

```js
{
  heading: { en: '...', es: '...' },
  paragraphs: [{ en: '...', es: '...' }],  // inline md supported
  steps: [{ en: '...', es: '...' }],        // inline md supported
  tip: { en: '...', es: '...' },            // optional
  warning: { en: '...', es: '...' },        // optional
}
```

Steps and paragraphs support `**bold**` and `[link text](url)` markdown via the `md()` utility.
