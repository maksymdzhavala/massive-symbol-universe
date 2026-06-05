# Massive — Symbol Universe Heatmap

Single-file animated landing page. Visualises Massive's coverage of US-market tickers across 5 asset classes (stocks, options, indices, futures, currency pairs) as a colored mosaic where each region is sized proportionally to its catalog count.

## Files

- `index.html` — everything (HTML + CSS + JS, no build step)
- `assets/indices/{1..5}.svg` — logos for indices region (mapped to SPX / NDX / DJI / RUT / VIX)
- `assets/currencies/{1..3}.svg` — logos for currencies region (logo-only, no price card)
- `exp.jpg`, `og.png`, `og-share.png` — hero image + social share images

## Running

Just open `index.html` in a browser. No server / build / install needed.

```
open index.html
```

---

## What's on screen

**Top bar (right):** live-mode toggle · mobile/desktop preview toggle · style toggle (glass/solid) · theme toggle (light/dark)

**Hero section:** "One API. Every US symbol." headline + ticker search bar (replaces the older "Get API key / Read the docs" CTA buttons)

**Stage:** 5 colored mosaic regions. Each tile has randomised opacity (0.45-0.95), ~5% are "hot" tiles that pulse.

**Two animation layers on the stage:**

1. **Mini flips** — frequent, decorative
   - 4×4 tile chips that flip in place to reveal a company logo
   - Biased to region corners (75%), enforced 8-tile separation between active minis
   - Up to 3 concurrent, ~420-900ms spawn gap, ~1000ms hold

2. **Hero flips** — rarer, informative, clickable
   - 24×10 tile blocks that flip to a stock card (logo + ticker + price + change% + corner sparkline)
   - Click any hero card → opens code panel modal with curl / JS / Python samples for that ticker
   - Up to 2 concurrent (different regions only), ~3.8-6.5s gap, ~1.7s hold

**Page-load reveal:** each region's tiles wipe in via a diagonal `mask-image` gradient sweep, cascaded by `regionIndex * 90ms`. Plays once on first load (`.mosaic-loading` class controls it); theme toggle skips it. Total time ~1.5s.

**Connection beam:** every 14-26s, a faint curved SVG line draws between two random region centers and animates a stroke-dasharray sweep (~1.6s). Subtle ambient touch suggesting "one schema unifies these regions."

**Footer:** "Five asset classes …" caption + animated count-up to `1.48 M+`

**Latency strip:** `p50 8ms · p99 24ms · uptime 99.99% · last sync 0s ago` — values jitter every 3.5-8s, last-sync counts up every second.

**Schema section (bottom):** "One schema. Every asset class." with tabbed JSON examples for stocks / options / indices / futures / currencies (syntax highlighted).

---

## Interactive features

- **Ticker search** in the hero — type AAPL / TSLA / SPX / NDX / DJI / MSFT etc., hit Enter. Opens the code panel with the matching ticker's API call, and on desktop also triggers a hero flip in that ticker's region. Miss → form shakes red briefly.
- **Click any hero card** while it's visible → opens the same code panel. Hover hint "Click for API ↗" appears in the top-right of the card.
- **Live mode toggle** (top-right, with pulsing red dot when on). Drifts sample prices on a 2.2s interval to feel like a feed. Affects both LOGOS array and the indices REGION_LOGOS entries.
- **Theme / style / mobile-preview toggles** — same as before.
- **Search bar focus** — border turns blue (`#5b86f7`) with a soft blue halo glow.

---

## Configuration knobs (all in the `<script>` block near the bottom)

### Hero card geometry
```js
const HERO_W = 24;   // tile width
const HERO_H = 10;   // tile height
```

### Hero card timing
```js
const FLIP_DUR = 620;
const FLIP_HOLD = 1700;
const HERO_MAX_ACTIVE = 2;   // simultaneous hero flips (different regions)
const HERO_GAP_MIN = 3800;
const HERO_GAP_MAX = 6500;
```

### Mini chip timing
```js
const MINI_CLUSTER = 4;
const MINI_HOLD = 1000;
const MINI_FLIP_DUR = 420;
const MINI_GAP_MIN = 420;
const MINI_GAP_MAX = 900;
const MINI_MAX_ACTIVE = 3;
const MINI_SEPARATION = 8;
```

### Hero card visual layout (CSS, `.hero-card .hc-chart`)
- Corner chart: `width: 22%; height: 26%; inset: auto 0 0 auto` (top-left explicitly nulled to override `.stage svg { inset: 0 }`)

### Colors (CSS custom properties)

Light mode:
- `--c-stocks: #0ea5e9` (sky blue)
- `--c-options: #a3a3a3` (light-medium gray)
- `--c-indices: #10b981` (emerald)
- `--c-futures: #eab308` (gold)
- `--c-currencies: #8b5cf6` (purple)

Dark mode: same hues but lighter (sky-400, gray-400, emerald-400, gold-400, violet-400).

---

## Stock data

Hardcoded sample prices in `LOGOS` array (stocks) and `REGION_LOGOS` map (per-region).

```js
const LOGOS = [
  { name: 'AAPL', price: 228.40, change: 1.84, svg: '...' },
  // TSLA, MSFT, NVDA, GOOGL, META, AMZN
];

const REGION_LOGOS = {
  '--c-indices': [
    { name: 'SPX', price: 5824.40,  change:  0.42, file: 'assets/indices/1.svg' },
    { name: 'NDX', price: 20486.10, change:  0.78, file: 'assets/indices/2.svg' },
    { name: 'DJI', price: 42634.50, change:  0.12, file: 'assets/indices/3.svg' },
    { name: 'RUT', price: 2318.66,  change: -0.34, file: 'assets/indices/4.svg' },
    { name: 'VIX', price: 14.82,    change: -2.18, file: 'assets/indices/5.svg' }
  ],
  '--c-currencies': [
    { file: 'assets/currencies/1.svg' },   // no price card — just logo on flip
    // ...
  ]
};
```

A logo with a `price` field gets the full stock card on hero flip; without it, only the logo image is shown.

---

## Sparkline algorithm

`buildSparkPaths(seed, trendUp, w, h, points)` generates both line + closed-area paths deterministically per ticker.

Linear trend (start → end based on up/down direction) + sine-faded noise that peaks in the middle and is zero at both endpoints. Every chart is an unambiguous up or down trend with mild variation — no swirly oscillations.

Path uses quadratic-bezier smoothing through midpoints. 12 points by default.

---

## Region geometry

```js
const regions = [
  { x: 24,  y: 20,  w: 472, h: 220, varName: '--c-stocks',     hotPct: 0.04 },
  { x: 504, y: 20,  w: 672, h: 320, varName: '--c-options',    hotPct: 0.05 },
  { x: 24,  y: 248, w: 472, h: 252, varName: '--c-indices',    hotPct: 0.05 },
  { x: 504, y: 348, w: 400, h: 152, varName: '--c-futures',    hotPct: 0.06 },
  { x: 912, y: 348, w: 264, h: 152, varName: '--c-currencies', hotPct: 0.08 }
];
```

Coordinates are in the stage SVG's 1200×520 viewBox. Tile constants: `TILE=5, GAP=1, STEP=6`.

Region labels (count + sub-label) are absolutely positioned divs over the stage SVG.

---

## Schema section data

The five JSON samples in the schema tabs are hardcoded in `setupSchemaTabs()`. Each follows the same unified shape: `symbol`, `asset_class`, `price`, `change`, `change_pct`, `ts`, plus asset-class-specific fields (exchange, expiry, strike, side, underlying, bid/ask, etc.).

Highlighter is a small regex tokenizer (keys, strings, numbers, booleans).

---

## Code panel data

`buildHeroCardEl(logo)` adds a click handler that calls `openCodePanel(logo)`. The panel renders curl / JS / Python snippets via `renderCodePanel()` using a tiny regex highlighter. Close with × button, backdrop click, or Esc.

---

## Known dead code (safe to delete if desired)

- `tumbleReveal` function (~80 lines)
- `.tumble-card`, `.tumble-inner`, `.tumble-face`, `.tumble-front`, `.tumble-back`, `.tumble-shade`, `tumbleFlip*`, `tumbleLift`, `tumbleShade` CSS rules
- Originally toggled by an animation-style button that was removed. Code is unused but left in case the tumble effect is wanted again.

---

## Performance notes — what we tried and removed

- **Mouse parallax (mosaic translates with cursor)** — added then removed; caused lag because layout was changing each frame even with RAF throttling.
- **Mouse spotlight (radial gradient + mix-blend-mode)** — added then removed; mix-blend-mode is expensive when composited over many tiles.
- **Region hover lift (CSS `transform: scale` on `<g class="region-group">`)** — added then removed; scaling a group containing tens of thousands of `<rect>` tiles forces a recomposite each frame.
- **Continuous mask drift (subtle alpha back-and-forth)** — tried; mask animation across the entire mosaic was visibly laggy.

Final state: only the **one-time load reveal** (5 per-region mask animations cascaded over ~1.5s) and the periodic **connection beam** (~once every 20s) are animated on the stage. No continuous mouse interaction effects.

---

## Iteration tips & gotchas

- The `.stage svg` rule (~line 362) applies `position: absolute; inset: 0; width: 100%; height: 100%` to *all* SVGs in the stage. Any new SVG positioned inside `.hero-card` must explicitly override `inset` (use `inset: auto 0 0 auto` or similar). This bit me hard when the hero card chart was rendering top-left instead of bottom-right.
- The hero flip's spot-picker checks both region label overlap and active mini cells before placing a card. If `HERO_W`/`HERO_H` are too large for a small region (futures: 66×25, currencies: 44×25), it may fail to find a clean spot and skip that round silently.
- Mini cards check `.in-logo` class to avoid hero areas; hero spot-picker iterates `miniBusyCells` to avoid active minis.
- Search uses a `{ticker → logo, region}` lookup built at startup from LOGOS + REGION_LOGOS. To add new searchable tickers, add them to one of those.
- Adding new stocks: 1-line change in `LOGOS` array.
- Live-mode drift modifies `logo.price` and `logo.change` in-place. Restart page to reset to defaults.
