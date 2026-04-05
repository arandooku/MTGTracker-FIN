# Foil Celebration & Variants Binder — Design Spec

**Date:** 2026-04-05  
**Status:** Approved

## Summary

Six related improvements to the MTG FIN tracker:

1. **Foil celebration** — when a foil/variant is added to the collection, fire a distinct rainbow-shimmer celebration (toast + particle burst) from both entry points (modal + FAB autocomplete)
2. **Main set count fix** — correct the hardcoded ~300 assumption to reflect the actual 309-card main booster set
3. **Variants binder tab** — new dedicated tab ("Variant Collection") showing all ~270 collector variant cards (CN 310–585) in a paginated binder grid, gated behind the existing `collectorVariants` scope flag
4. **Tab renames** — "Foil" → "Foil Collection"; new variants tab labelled "Variant Collection"
5. **Tab bar UI overhaul** — consistent sizing and improved look/feel across all tabs
6. **Header cleanup** — remove rarity stat-badge pills from below the progress bar; remove "ClearEdge 360" label, show only the active binder size

---

## FIN Set Card Count Reference

| Collector Number Range | Count | What they are |
|---|---|---|
| CN 1–309 | 309 | Main booster set |
| CN 315–406 | 92 | Borderless character & woodblock cards |
| CN 407–420 | 14 | Alt-art Cid variants |
| CN 421–517 | 97 | Extended-art cards |
| CN 519–550 | 32 | Surge foil character cards |
| CN 551 (a–f) | 6 | Chocobo variants (neon ink + serialized) |
| CN 552–563 | 12 | Starter Kit cards |
| CN 564–571 | 8 | Chocobo Bundle / holiday cards |
| CN 572–585 | 14 | Surge foil lands + others |
| **Total** | **~577+** | **All printings** |

---

## Feature 1: Main Set Count Fix

The binder config wizard currently defaults to `300` cards. The actual main booster set has **309** cards. 

**Change:** Update the wizard default from `300` to `309`. The binder is already card-count-agnostic (page count is derived from `allCards.length / slotsPerPage`), so no other changes are needed. Remove any hardcoded `300` references in binder math or display strings.

---

## Feature 2: Foil Add Celebration

### Entry Points

`addFoil(cn)` is the single write function for foil ownership, called from:
1. Card modal "Add Foil ✦" button (`modal-foil-toggle` onclick)
2. FAB search autocomplete quick-add foil button (event delegation in FAB click handler)

**Change:** After saving to storage, `addFoil(cn)` looks up the card via `getCardByNumber(cn)` and calls `celebrateFoil(card)`. Both entry points are covered automatically.

### Variant Label Helper

`getFoilVariantLabel(card)` — derives the display badge label. Reuses detection logic from existing `getVariantType()` in the foil tab renderer.

| Condition | Label |
|-----------|-------|
| `frame_effects` includes `'surge'` | `SURGE FOIL` |
| `promo_types` includes `'neon_ink'` | `NEON INK FOIL` |
| `name` includes `'Traveling Chocobo'` | `NEON INK FOIL` |
| `frame_effects` includes `'serialized'` | `SERIALIZED` |
| `finishes` includes `'foil'` (default) | `TRADITIONAL FOIL` |
| Collector variant fallback | `COLLECTOR FOIL` |

### `celebrateFoil(card)` Function

Parallel to `celebrateAdd(card)`. Calls:

1. **Rainbow glow ring** — `spawnGlowRing()` with `'rgba(199,125,255,0.7)'` (prismatic purple)
2. **`spawnRainbowBurst()`** — new particle function (see below)
3. **`showFoilToast(card)`** — foil-specific toast (see below)

No screen flash (foil is a shimmer reveal, not a rarity shock). No gold shower (stays mythic-exclusive).

### `spawnRainbowBurst()` — New Particle Function

- ~40 particles emitted from screen center
- Each particle colored by cycling through 5 foil palette colors: `#ff6b9d`, `#ffd93d`, `#6bcb77`, `#4d96ff`, `#c77dff`
- Fly outward at random angles with varying speeds and sizes
- Fade out over ~1–1.5s
- Same DOM-append-then-remove pattern as `spawnSparkles` / `spawnFireworks`
- `pointer-events: none`, `position: fixed`, `z-index: 9999`

### `showFoilToast(card)` — Foil Toast

Parallel to `showAddToast(card, cfg)`. Reuses `.add-toast` base class, adds `.foil` modifier.

**Structure:**
```
[ SURGE FOIL ]             ← .toast-foil-badge (rainbow shimmer gradient)
[ card image ]
[ #123 Card Name ]         ← .toast-name
[ shimmer message ]        ← .toast-foil-msg (rainbow shimmer text)
[ Page X, Slot Y · $4.20 foil ]  ← .toast-placement
```

**Badge (`.toast-foil-badge`):**
- Text: variant label from `getFoilVariantLabel(card)`
- Background: `linear-gradient(135deg, #ff6b9d, #ffd93d, #6bcb77, #4d96ff, #c77dff)`
- `background-size: 300% 300%` + `animation: foil-shimmer 3s ease infinite`
- White text, bold, uppercase

**Glow / Border:**
- `--toast-glow` set to `rgba(199,125,255,0.4)`
- Border: `1.5px solid rgba(255,255,255,0.2)`

**Message Pool (`.toast-foil-msg`)** — random pick from:
- `"Shimmer acquired."`
- `"It catches the light."`
- `"The foil is real."`
- `"Prismatic."`
- `"That one's a keeper."`
- `"Foil secured. ✦"`
- `"Ooh, shiny."`

Styled with rainbow shimmer text via `background-clip: text` on the foil gradient.

**Price Line:** Shows `price_usd_foil` if > 0, else falls back to `price_usd`. Format: `$X.XX foil` or `$X.XX`.

**Dismiss Timing:** Fixed at 3000ms.

### CSS Additions

| Selector | Purpose |
|----------|---------|
| `.toast-foil-badge` | Rainbow shimmer badge for foil toast |
| `.add-toast.foil` | Prismatic glow/border overrides for foil toast container |
| `.toast-foil-msg` | Shimmer gradient text for foil message line |

All new selectors live in the existing Toast CSS section alongside `.toast-rarity-badge`.

---

## Feature 3: Variants Binder Tab

### Gating

The `Variants` tab appears in the tab bar when `binderConfig.scope.collectorVariants` is `true` — same conditional rendering pattern as the `Foil` tab. No new scope flag needed; `collectorVariants` is the correct semantic gate.

### Data

Renders `variantCards` array (already loaded and sorted by `collectorKey()` when `collectorVariants` scope is enabled). Page count derived dynamically from `variantCards.length / slotsPerPage`.

### Grid & Slots

Identical structure to the main Binder tab:

- **Slots** — card image (or empty placeholder), collector number badge, rarity dot
- **Owned slots** — `.slot-owned` green shimmer border (same as main binder; ownership via `ownedSet`)
- **Foil-owned slots** — `.slot-foil-owned` rainbow shimmer overlay stacked on top when `hasFoil(cn)` is true
- **Unowned slots** — heavily dimmed: `filter: grayscale(1) brightness(0.25); opacity: 0.5` (increased from current `grayscale(0.7) brightness(0.6)` — clearly absent, still identifiable)
- **Foil badge** — ✦ badge on foil-owned slots
- **Variant type label** — small chip showing variant type (Surge Foil, Neon Ink, Borderless, Extended Art, etc.) via `getFoilVariantLabel()` — shared helper from Feature 2
- **Slot click** — toggles ownership via `addCard` / `removeCard`, triggers `celebrateAdd` on add

### Filter Bar

Same type/rarity/search layout as the Foil tab:

- **Type filters:** `All` | `Owned` | `Traditional` | `Surge` | `Neon Ink` | `Chocobo Track`
- **Rarity pills:** `C` | `U` | `R` | `M`
- **Name search input**

#### Corrected Filter Logic

Research confirmed that the Foil tab's existing `buildGrid()` filter logic for Surge and Neon Ink uses **wrong field names** — `frame_effects.includes('surge_foil')` and `frame_effects.includes('neon_ink')` will never match real Scryfall data. The correct detection is already used in the codebase by `getVariantType()`. The Variants tab must use the correct logic below, and the **Foil tab's `buildGrid()` must be fixed at the same time**:

| Filter | Correct Logic | Broken logic to fix in Foil tab |
|--------|---------------|--------------------------------|
| `All` | all variant cards | — |
| `Owned` | `ownedSet.has(cn)` | — |
| `Traditional` | cards where `frame_effects` does NOT include `'surge'` or `'serialized'`, AND `promo_types` does NOT include `'neon_ink'`, AND name does not include `'chocobo'` — i.e. borderless/extended art cards | replaces "show allCards" behaviour which is irrelevant for variant-only view |
| `Surge` | `frame_effects.includes('surge')` | was `finishes.includes('surge_foil') \|\| frame_effects.includes('surge_foil')` |
| `Neon Ink` | `promo_types.includes('neon_ink')` | was `frame_effects.includes('neon_ink')` |
| `Chocobo Track` | `name.toLowerCase().includes('chocobo')` | was `frame_effects.includes('chocobo_track') \|\| name.includes('chocobo')` — frame_effects check is dead code, name check is fine |

Note: `promo_types` is already stored in `variantCards` (fetched at line ~4994).

Filter state persisted on the DOM element (`el._variantFilter` / `el._variantRarity` / `el._variantSearch` — same pattern as Foil tab).

### Pagination & Grid Size

Both the Foil tab and the Variants tab replace the current "dump all cards" layout with **page-based pagination** identical to the main binder:

- Page size = `binderConfig.slotsPerPage` (defaults to 9, user-configurable) — one page fits the screen without scrolling
- Each tab tracks its own current page in a closure-local variable (`foilPage`, `variantPage`), reset to 0 when filters change
- **Prev / Next** buttons below the grid — same style as the main binder's page controls
- Page indicator: `Page X of Y` centred between the buttons
- On mobile, same single-page view; no spread view (that stays binder-only)

This replaces the existing foil tab's IntersectionObserver lazy-load-everything approach. Lazy loading of images within the current page is still used.

### Navigation

- Prev / Next page buttons with page indicator
- No two-page spread view (binder-only feature)

### Ownership Model

Variant cards use `ownedSet` (same as main set cards) — they are regular collection cards, not foil-only. Foil ownership layered on top via `foilOwned` as usual.

---

---

## Feature 7: Foil Card Visual Treatment

### Problem

The current foil shimmer runs on the *slot container's background*, which is fully covered by the card image. The card image itself looks completely flat — no foil effect is visible on the actual card surface.

### Approach — CSS `::after` Overlay

A `::after` pseudo-element is added to owned foil slots. It sits **on top of the card image** as a semi-transparent gradient overlay. Zero extra DOM nodes. CSS-only animation. GPU-accelerated via `will-change: background-position`.

Stagger delays are applied via `foil-delay-{0|1|2}` classes (assigned by `idx % 3`) so cards don't pulse in sync — natural, organic feel.

With pagination limiting the page to `slotsPerPage` cards (~9), animating 9 pseudo-elements is negligible on all devices.

### Variant CSS Classes

During slot render, a variant class is added to owned foil slots based on card data (using same detection logic as `getFoilVariantLabel`):

| Class | Condition | Visual style |
|-------|-----------|-------------|
| *(default)* | traditional foil | Rainbow diagonal shimmer, 4s |
| `.foil-variant-surge` | `frame_effects.includes('surge')` | Blue-white electric shimmer, 2.5s, more intense |
| `.foil-variant-neon` | `promo_types.includes('neon_ink')` or name includes `'chocobo'` | Bright neon glow pulse on box-shadow, no moving gradient |
| `.foil-variant-serialized` | `frame_effects.includes('serialized')` | Slow gold shimmer, 5s, premium feel |
| `.foil-variant-collector` | all other collector variants | Soft silver-white shimmer, 3.5s |

### CSS Design

```
/* Shared overlay base — sits above card image, below badge */
.foil-slot.slot-foil-owned::after {
  content: ''; position: absolute; inset: 0; border-radius: 4px;
  pointer-events: none; z-index: 1;
  will-change: background-position;
  animation: foil-card-shimmer 4s ease-in-out infinite;
  background: linear-gradient(115deg,
    transparent 25%, rgba(255,107,157,0.3) 35%,
    rgba(77,150,255,0.25) 48%, rgba(107,203,119,0.25) 58%,
    rgba(255,217,61,0.3) 68%, transparent 78%);
  background-size: 300% 300%;
  mix-blend-mode: screen;
}

/* Surge — faster, electric blue-white */
.foil-slot.foil-variant-surge.slot-foil-owned::after {
  background: linear-gradient(115deg,
    transparent 20%, rgba(160,210,255,0.5) 35%,
    rgba(255,255,255,0.55) 50%, rgba(120,175,255,0.5) 65%,
    transparent 80%);
  animation-duration: 2.5s;
}

/* Neon — pulsing glow, no gradient travel */
.foil-slot.foil-variant-neon.slot-foil-owned::after {
  background: radial-gradient(ellipse at center,
    rgba(255,80,180,0.35) 0%, rgba(80,220,255,0.25) 50%, transparent 75%);
  animation: foil-neon-pulse 2s ease-in-out infinite;
  /* foil-neon-pulse scales opacity 0.4 → 1 → 0.4 */
}

/* Serialized — slow gold sweep */
.foil-slot.foil-variant-serialized.slot-foil-owned::after {
  background: linear-gradient(115deg,
    transparent 30%, rgba(255,215,0,0.4) 42%,
    rgba(255,180,50,0.45) 52%, rgba(255,215,0,0.4) 62%,
    transparent 72%);
  animation-duration: 5s;
}

/* Collector — soft silver */
.foil-slot.foil-variant-collector.slot-foil-owned::after {
  background: linear-gradient(115deg,
    transparent 30%, rgba(200,220,255,0.3) 45%,
    rgba(255,255,255,0.35) 55%, rgba(200,220,255,0.3) 65%,
    transparent 75%);
  animation-duration: 3.5s;
}
```

```
@keyframes foil-card-shimmer {
  0%   { background-position: 0% 0%; }
  50%  { background-position: 100% 100%; }
  100% { background-position: 0% 0%; }
}
@keyframes foil-neon-pulse {
  0%, 100% { opacity: 0.4; }
  50%       { opacity: 1; }
}
.foil-delay-0::after { animation-delay: 0s; }
.foil-delay-1::after { animation-delay: 1.35s; }
.foil-delay-2::after { animation-delay: 2.7s; }
```

### Slot Border & Glow Cleanup

The existing `.slot-foil-owned` background gradient is hidden by the card image and does nothing visible. Simplify it to border + glow only, with variant-specific glow colours:

| Variant | Box-shadow colour |
|---------|------------------|
| Traditional | `rgba(255,200,100,0.3)` (existing warm gold) |
| Surge | `rgba(100,180,255,0.45)` (electric blue) |
| Neon | `rgba(255,80,200,0.5)` (neon pink) |
| Serialized | `rgba(255,215,0,0.5)` (gold) |
| Collector | `rgba(200,220,255,0.3)` (silver) |

### Unowned Slots

No `::after` overlay. Heavy dim applied directly on the image: `filter: grayscale(1) brightness(0.25); opacity: 0.5` (same as Feature 3 spec above).

---

The existing Foil tab `renderFoilBinder()` grid also gets these updates:

- **Pagination added** — replaces the current all-cards-in-one-grid layout. Same `slotsPerPage` page size, same prev/next controls
- **Dimming increased** — unowned foil slots change from `grayscale(0.7) brightness(0.6)` → `grayscale(1) brightness(0.25); opacity: 0.5`
- **Filter logic fixed** — Surge and Neon Ink filter checks corrected (see filter table above)

---

---

## Feature 4 & 5: Tab Renames & Tab Bar UI Overhaul

### Renames

| Old label | New label | `data-tab` (unchanged) |
|-----------|-----------|------------------------|
| `✦ Foil` | `✦ Foil Collection` | `foil` |
| *(new)* | `◈ Variant Collection` | `variants` |

All other tab labels remain as-is.

### Tab Bar Visual Design

**Problem:** Current tabs have uneven widths (short "Binder View" vs long "Main Dashboard"), use inconsistent icon treatment (SVG icons on some, text glyphs on foil/settings), and the active state is only a bottom-border underline — thin and easy to miss on mobile.

**New approach — pill/capsule active indicator:**

- Tab bar background: subtle `var(--surface)` container with `border-radius: 12px` and `padding: 4px` — a "pill tray" that wraps all tabs
- Each tab button: `flex: 1; min-width: 0` so all tabs share equal width on desktop
- Active tab: filled capsule background (`var(--accent)` with low opacity fill + solid accent text), replacing the underline — clearly visible on both desktop and mobile
- Inactive tabs: no border, muted text colour, hover shows slight background tint
- Remove bottom `border-bottom: 2px solid` from `.tabs` container — the pill tray provides its own visual separation
- Mobile: tabs remain horizontally scrollable (`overflow-x: auto`) with `flex-shrink: 0` so they don't compress — equal width only applies when all tabs fit

**Icon consistency:**
- All tabs use the same SVG icon size (`16×16`) already present on Dashboard/Portfolio/Timeline/Binder
- Foil Collection: replace `✦` text glyph with a small star/sparkle SVG icon
- Variant Collection: use a layers/stack SVG icon
- Settings: replace `⚙` text with an SVG gear icon (already used elsewhere in the UI)

**Font size:** `clamp(0.72rem, 1.4vw, 0.85rem)` — slightly tighter than current to accommodate longer labels without wrapping

---

## Feature 6: Header Cleanup

### Remove Rarity Stat-Badge Pills

`renderProgress()` currently populates `#stats-row` with four coloured pill badges: `mythic: X/Y`, `rare: X/Y`, `uncommon: X/Y`, `common: X/Y`.

**Change:** Remove the `stats-row` render entirely from `renderProgress()`. The `<div class="stats-row" id="stats-row">` element in the HTML can remain (or be removed) but should render nothing. The rarity breakdown is already visible in the Dashboard tab's rarity cards — no information is lost.

### Remove "ClearEdge 360"

The subtitle currently reads: `ClearEdge 360 · 9-Pocket Binder · 34 Pages`

**Change:** Remove the `<span>ClearEdge 360</span>` and its adjacent `<span class="sep">` entirely. The subtitle becomes: `9-Pocket Binder · 34 Pages` (or whatever the active config shows via `#binder-preset-display` and `#binder-page-count-display`). No other subtitle changes.

---

- `removeCard` / `removeFoil` — no celebration on removal
- Regular `addCard` / `celebrateAdd` — unchanged
- Foil tab filter / stats rendering — unchanged
- Binder config wizard steps — only the default card count value changes (300 → 309)
