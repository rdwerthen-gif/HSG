# HSG — Habits, Systems, Goals

A static, single-user personal productivity app (habits, goals, calendar, budget, fitness, spirituality, project planning, and more) that runs entirely client-side. No backend, no build step required — all data is stored in the browser's `localStorage`.

## Repository Structure

```
/
├── index.html          Semantic markup — all page sections, no inline CSS/JS (except one
│                        tiny theme-flash-prevention snippet, see "Why one inline script" below)
├── styles.css           All styling — extracted from the original single-file build
├── script.js            All application logic — extracted from the original single-file build
├── assets/
│   ├── images/          Place any custom images/screenshots here (none required — the app
│   │                    currently uses emoji and inline SVG for all icons/graphics, so there
│   │                    are zero image assets to migrate)
│   └── fonts/           Not currently used — fonts are loaded from Google Fonts via CDN
│                        (see "Fonts" below for a self-hosting option)
└── README.md            This file
```

## Deploying to GitHub Pages

1. Push these files to the root of a GitHub repository (or to a `/docs` folder, or a dedicated branch — whichever your Pages source setting points to).
2. In the repo's Settings → Pages, set the source to that location.
3. That's it — no build step, no `npm install`, no bundler. This is a static site.

## Honest Performance Notes — Please Read Before Assuming 100/100

This split gives you real, meaningful wins:
- **`script.js` is `defer`red** — the browser downloads it in parallel with parsing the HTML and doesn't block rendering, and it's also `<link rel="preload">`ed so the fetch starts as early as possible.
- **`styles.css` loads first, before third-party scripts** — critical rendering path is prioritized.
- **Google Fonts load non-blocking** (the `media="print"` → `onload="this.media='all'"` trick) and **preconnect hints** are in place for both Google Fonts and cdnjs.
- **The two CDN dependencies** (`xlsx.js` for spreadsheet export, `jspdf.js` for PDF export) are both `defer`red.
- **Zero inline styles or scripts remain in `index.html`**, except one ~500-byte snippet that reads a saved theme from `localStorage` and applies it *before* first paint — this one has to stay inline and un-deferred, or you'd get a visible flash of the wrong theme/font on every load. This is a standard, deliberate exception, not an oversight.

**What this split does *not* do, and what a genuine 100/100 would actually require:**

- **`script.js` is ~2.9MB.** That's not incidental bloat — it's the actual content of the app: 150 Gantt chart templates, 60 finance books with full breakdowns, 110+ author/framework education entries across Time Management, Project Management, and Goals, 90+ Fitness/Spirit topic deep-dives, and more, all held as in-memory JavaScript data so every page works instantly with no network requests once loaded. Deferring it means it doesn't *block* rendering, but the browser still has to download and parse ~2.9MB before the app is interactive — that alone will cost you meaningfully on Lighthouse's Total Blocking Time and Time to Interactive metrics, regardless of file organization.
- **A real fix for that is code-splitting**, not file-splitting: lazy-load each page's data only when the user actually navigates there (e.g., don't parse the 60-book Finance library's data until someone opens the Finance page), rather than loading everything upfront. That's a genuine architectural change — turning each page's dataset into its own dynamically-`import()`ed module — not something that can be done by moving code between three files. It's a substantial follow-up project in its own right if you want to pursue it.
- **`styles.css` is ~175KB unminified.** Minifying it (any CSS minifier, or a GitHub Action that runs one on push) is a safe, mechanical win worth doing before shipping. The same goes for `script.js` — minification/tree-shaking would help, though its size is dominated by data, not verbose code, so the gains are more modest than they'd be for a typical app bundle.

If your priority is genuinely hitting 100/100, the honest next step is deciding which pages' data can be deferred until needed, not further reorganizing these three files.

## Fonts

Currently loaded from Google Fonts CDN (18 font families, only the specific weights used). To self-host instead (removes the Google Fonts network dependency entirely, which helps both performance and privacy):
1. Download the `.woff2` files for each family/weight from [google-webfonts-helper](https://gwfh.mranftl.com/fonts) into `assets/fonts/`.
2. Replace the Google Fonts `<link>` in `index.html` with `@font-face` declarations at the top of `styles.css`.

## Images

The app currently has no raster image assets — all icons and graphics are emoji or inline SVG (generated at runtime by `script.js`). If you add custom images later (a logo, screenshots, etc.):
- Use **WebP** format (with a JPEG/PNG fallback via `<picture>` if you need to support very old browsers).
- Always set explicit `width` and `height` attributes on `<img>` tags to reserve layout space and prevent Cumulative Layout Shift.
- Place them in `assets/images/` and reference with relative paths (`assets/images/your-file.webp`).

## Data & Privacy

All user data lives in the browser's `localStorage` under a single key. Nothing is transmitted anywhere. Users can export/import a backup from Settings → Data & Sync.
