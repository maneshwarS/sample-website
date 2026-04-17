# sample-website

A single-page landing site recreating [AnyVan](https://www.anyvan.com)'s visual brand, built around a **141-frame scroll-driven animation** as the hero interaction. Vanilla HTML / CSS / JS — no build step, no dependencies.

**Live locally:** `python3 -m http.server 8000` and open `http://localhost:8000`.

## What's on the page

| Section | Notes |
|---|---|
| **Navbar** | Blue AnyVan bar, logo, nav links, yellow "Get Instant Prices" CTA |
| **Hero** | Headline, five-card asymmetric service grid (Home removals, Furniture, Cars, Storage, Other), Trustpilot badge |
| **Scroll animation** | 350vh sticky section driving a canvas through 141 JPG frames, with four annotation cards that fade in at scroll progress 0.12 / 0.38 / 0.60 / 0.82 |
| **Recent moves** | 6-card grid of real example moves with prices and star ratings |
| **Why AnyVan** | Media-text with the three key benefits (teams, 48-hour cancellation, £50k cover) |
| **Ask Angus** | FAQ accordion using native `<details>`, founder photo |
| **Footer** | Dark grey, contact number, nav columns, legal links |

## Structure

```
.
├── index.html          # Everything: markup, styles, scripts
├── frames/             # 141 JPG frames (frame_0001.jpg … frame_0141.jpg, ~23 MB)
└── .gitignore
```

Frames are committed so the scroll animation works out of the box on any static host — Vercel, Netlify, GitHub Pages, S3. No extra setup.

## The scroll animation

On page load, all 141 frames preload in parallel behind a loader with a progress bar. Once loaded, the scroll handler picks a frame based on how far the user has scrolled into the sticky section and paints it to a canvas (`alpha: false`, DPR capped at 1.5). Annotation cards toggle a `.visible` class based on scroll progress — CSS handles the fade.

**Perf decisions** (kept honest after a real trace showed the first pass was janky):

- **No scroll hijack.** An earlier version set `document.body.style.overflow = 'hidden'` for 580 ms each time a card entered, which froze scrolling up to 2.3 s across four cards. Pure CSS opacity transitions now do the reveal.
- **No `backdrop-filter` on scrolling elements.** Cards and navbar use solid backgrounds with borders/shadows instead — the compositor doesn't have to re-blur a region on every scroll frame.
- **Capped DPR at 1.5.** On Retina screens, drawing the full viewport at 3× adds no perceptible quality but costs a lot per frame.
- **Single rAF-throttled scroll handler** drives both the frame canvas and card visibility.

Trace on a 6700 px scroll: locked 60 FPS.

## Deploying

Drop the repo on any static host. For GitHub Pages:

```
Settings → Pages → Source: Deploy from branch → main / (root)
```

For Vercel or Netlify: point at the repo, no build command needed, output directory is the repo root.

## Credits

Imagery and brand references from [anyvan.com](https://www.anyvan.com) — this is a visual recreation for learning / portfolio purposes, not an affiliated project.
