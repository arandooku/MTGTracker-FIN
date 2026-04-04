# Binder Setup & Configuration — Design Spec

**Date:** 2026-04-04
**Status:** Approved

---

## Overview

Add a configurable binder setup system so users can define their physical binder's grid size, page count, and what they want to collect (main set, collector variants, foil binder). A first-run wizard guides new users through setup; a persistent Settings tab allows reconfiguration at any time.

---

## Data Model

### localStorage key: `fin-binder-config`

```json
{
  "gridRows": 3,
  "gridCols": 3,
  "slotsPerPage": 9,
  "pageCount": 34,
  "presetName": "ClearEdge 360 (9-pocket)",
  "scope": {
    "mainSet": true,
    "collectorVariants": false,
    "foilBinder": false
  },
  "configured": true
}
```

- `slotsPerPage` = `gridRows × gridCols` (derived, stored for convenience)
- `pageCount` auto-calculates as `ceil(totalCards / slotsPerPage)`, user-overridable
- `scope.mainSet` is always `true` — cannot be disabled
- `configured: false` (or key missing) triggers auto-navigation to Settings tab on load

### Preset Binder Sizes

| Label | Rows | Cols | Slots/page |
|-------|------|------|------------|
| 4-pocket (2×2) | 2 | 2 | 4 |
| 9-pocket (3×3) | 3 | 3 | 9 |
| 12-pocket (3×4) | 3 | 4 | 12 |
| 16-pocket (4×4) | 4 | 4 | 16 |
| 18-pocket (3×6) | 3 | 6 | 18 |
| Custom… | user | user | user |

### Additional localStorage Keys

| Key | Purpose |
|-----|---------|
| `fin-foil-collection` | `{ "47": ["foil"], "322": ["foil"] }` — foil ownership by collector number |
| `fin-variant-data-v1` | Cached collector booster variant cards (7-day TTL, same pattern as `fin-card-data-v4`) |

### Existing key changes
- `fin-card-data-v4` → bump to **`fin-card-data-v5`** to add `finishes` and `oracle_id` fields
- Gist sync payload gains `binderConfig` and `foilCollection` fields

---

## First-Run Wizard

Renders inside the Settings tab. Shown automatically when `configured` is falsy. Three steps with a progress indicator; Back/Next navigation; no skip.

### Step 1 — Binder Type
- Preset dropdown
- Selecting "Custom…" reveals Rows × Columns number inputs
- Live preview: *"X slots per page"*
- Hint: *"For the main set (300 cards) this means Y pages"* (updates as scope changes in Step 2)

### Step 2 — Collection Scope
Three independent toggle switches:

| Toggle | Always-on? | Description |
|--------|-----------|-------------|
| Main Set | Yes (locked) | ~300 booster-pack cards |
| Collector Variants | No | Borderless, showcase, surge foil, neon ink, etc. Triggers extra Scryfall fetch. |
| Foil Binder | No | Adds a separate Foil tab for tracking foil ownership |

Live summary below toggles: *"Your binder will track Z cards across Y pages."*

### Step 3 — Review & Apply
- Summary card of all choices
- **"Set Up My Binder"** button: saves config, navigates to Binder tab

---

## Settings Tab (Returning Users)

When `configured: true`, the Settings tab shows a compact form instead of the wizard.

### Binder Setup Section
- Preset dropdown + custom inputs (same as wizard)
- Page count field (auto-calculated, editable)
- Live summary: *"X slots per page · Y pages · Z total slots"*

### Collection Scope Section
- Same three toggles (Main Set locked on)
- Toggling Collector Variants on for the first time triggers Scryfall fetch on Save

### Save Changes Button
- Applies config to localStorage
- Recalculates all binder slot positions
- Re-renders active tab
- Shows success toast: *"Binder updated."*

### Danger Zone (collapsible, red-tinted)

| Action | Effect | Confirmation required |
|--------|--------|-----------------------|
| Reset Collection | Clears `fin-collection` | Yes |
| Reset Foil Collection | Clears `fin-foil-collection` | Yes |
| Reset Everything | Wipes all localStorage, returns to first-run wizard | Yes |

---

## Foil Binder Tab

- Visible only when `scope.foilBinder: true`
- Positioned between Portfolio and Timeline in the tab bar
- Same grid layout as Binder tab, driven by same `fin-binder-config`
- Separate ownership store: `fin-foil-collection`
- Hiding via Settings toggle: tab disappears, data preserved in localStorage
- Card slots: owned foil = rainbow shimmer CSS animation; unowned = greyed out with subtle foil-border hint
- Clicking a slot opens existing `openModal()` with an additional **"Add Foil ✦" / "Remove Foil"** button

---

## Impact on Existing Tabs

### Binder Tab
- Grid renders `gridRows × gridCols` slots per page (was hardcoded 3×3)
- `collectorKey()` sort and slot placement use `slotsPerPage` from config (was hardcoded `9`)
- Spread view unchanged (2 pages side-by-side)
- Collector variant cards shown if `scope.collectorVariants: true`

### Dashboard Tab
- Progress stats reflect scoped card pool total (300 base, +N if collector variants enabled)

### Portfolio Tab
- Value calculations scoped to active card pool
- Collector variant prices included when that scope is on

### Pack Simulator Tab
- Unchanged — always simulates from main booster pool

### Cloud Sync
- `fin-binder-config` added to Gist sync payload
- `foilCollection` added to Gist sync payload

---

## Scryfall Data Changes

### Main card fetch (updated)
Query: `set:fin+is:booster+game:paper` (unchanged)
New fields captured per card: `finishes`, `oracle_id`, `foil_only`
Cache key bumped: `fin-card-data-v4` → `fin-card-data-v5`

### Collector variant fetch (new, conditional)
Query: `set:fin+game:paper+-is:booster`
Triggered when: user enables Collector Variants scope
Cache key: `fin-variant-data-v1` (7-day TTL)
Post-processing: each variant linked to its main card via `oracle_id` match

---

## Binder Math

Current hardcoded values to replace with config-driven equivalents:

| Was hardcoded | Becomes |
|---------------|---------|
| `9` (slots per page) | `config.slotsPerPage` |
| `34` (page count) | `config.pageCount` |
| `300` (total cards) | `allCards.length` (dynamic) |

---

## UI Design Language & Animations

All new UI must feel like a natural extension of the existing Catppuccin-themed, dark-first aesthetic. The guiding principle: **purposeful motion, no decorative noise**. Every animation communicates state change.

### Design Tokens (reuse existing CSS variables)
- Surfaces: `--surface`, `--surface2`, `--slot-bg`
- Accent: `--accent` (#f38ba8 rose / #d20f39 light), `--accent2` (#cba6f7 lavender / #8839ef light)
- Spring easing: `cubic-bezier(0.34, 1.4, 0.64, 1)` — matches existing modal-in, card-enter
- Strong spring: `cubic-bezier(0.34, 1.56, 0.64, 1)` — matches missing-inspect hover
- Text: `--text`, `--text-muted`

### Wizard Step Transitions
- Steps slide horizontally: entering step slides in from the right (`translateX(32px) → 0`), exiting step slides out to the left (`0 → translateX(-32px)`), both with `opacity 0→1 / 1→0`, duration `0.3s`, easing `cubic-bezier(0.4,0,0.2,1)`
- Going Back reverses the direction
- Progress pill bar fills smoothly: `width` transition `0.4s ease`

### Toggle Switches
- Custom CSS toggle (not native checkbox): pill track + circle thumb
- Thumb slides with `transform: translateX` — `0.22s cubic-bezier(0.34,1.4,0.64,1)` (slight overshoot)
- Track color transitions: `--surface2` → `--accent` over `0.2s ease`
- Locked toggles (Main Set): visually distinct — muted opacity, lock icon, `cursor: not-allowed`
- Minimum tap target: 44×28px

### Preset Dropdown & Custom Inputs
- Custom-styled `<select>` with chevron icon, matches existing input aesthetic
- Custom inputs (rows/cols) have a focus ring: `box-shadow: 0 0 0 2px var(--accent)` with `0.15s ease`
- Live summary line beneath animates value changes: number counts up/down with a subtle `scale(1.08) → scale(1)` pop, `0.2s spring`

### Settings Form Save Button
- Default state: standard `--accent` fill
- On click: brief `scale(0.96)` press → release spring back `cubic-bezier(0.34,1.56,0.64,1)`
- On success: button briefly shows a checkmark icon with a `scale(0→1)` entrance, then label returns — `0.4s` total
- Toast notification slides in from bottom: `translateY(100%) → translateY(0)`, fades out after 2s

### Section Headers in Settings Form
- Each section (`Binder Setup`, `Collection Scope`, `Danger Zone`) has a thin `--accent` left-border rule and a label in `--text-muted`
- Danger Zone uses `--accent` in red tint (`#f38ba8` dark / `#d20f39` light) for its border and action buttons

### First-Run Wizard Container
- Wizard card: `background: var(--surface)`, `border-radius: 16px`, `box-shadow: 0 8px 32px var(--shadow)`
- Entrance: `card-enter` keyframe (already defined — `translateY(12px) opacity:0 → translateY(0) opacity:1`), `0.4s cubic-bezier(0.34,1.4,0.64,1)`
- "Set Up My Binder" CTA button: full-width, `--accent` background, subtle shimmer sweep on hover (same pattern as `.missing-inspect::after`)

### Foil Binder Tab Entry Animation
- When Foil tab becomes visible after enabling in Settings: tab button fades + scales in `(scale(0.85) opacity:0 → scale(1) opacity:1)`, `0.3s spring`
- Grid slots enter staggered: `animation-delay: index * 0.015s` (same pattern as existing `card-enter` stagger)

### Bottom Sheet (Mobile Danger Zone Dialogs)
- Slides up from bottom: `translateY(100%) → translateY(0)`, `0.35s cubic-bezier(0.34,1.4,0.64,1)`
- Backdrop fades in: `opacity 0→0.6`, `0.25s ease`
- Dismiss: reverse slide + fade, `0.25s ease`
- Drag handle indicator (2px × 32px rounded pill) at top of sheet

### General Principles
- No animation exceeds `0.5s` — keep it snappy
- `prefers-reduced-motion: reduce` media query wraps all transitions/animations with `duration: 0.01ms` fallback
- Interactive elements have `:hover` and `:active` states — no bare, unresponsive buttons
- Focus-visible outlines for keyboard/accessibility: `outline: 2px solid var(--accent); outline-offset: 2px`

---

## Mobile Considerations

The existing app is built mobile-first (viewport meta, `100dvh`, `safe-area-inset`, `touch-action: manipulation`, `-webkit-tap-highlight-color: transparent`). All new UI must match these existing patterns.

### Wizard (Settings Tab — first run)
- Steps render full-width, single-column — no sidebars
- **Back / Next** buttons are full-width, minimum 48px tall (touch target)
- Preset dropdown uses native `<select>` (iOS/Android keyboard-friendly)
- Custom row/col inputs use `inputmode="numeric"` to trigger numeric keyboard on mobile
- Progress indicator is a simple pill bar at the top, not a horizontal stepper (avoids overflow on narrow screens)
- Step content scrolls vertically if content is tall; buttons are sticky at bottom of viewport

### Settings Form (returning users)
- All sections stack vertically, full-width
- Toggle switches minimum 44px tap target (follow existing `.btn` sizing)
- Page count input uses `inputmode="numeric"`
- Save Changes button full-width on mobile
- Danger Zone section collapses behind a tap-to-expand disclosure; destructive buttons are large and clearly separated

### Foil Binder Grid
- Uses same responsive grid as the existing Binder tab — scales card slot size based on viewport width
- On narrow screens (< 480px), `slotsPerPage` above 9 may produce very small thumbnails; implementation should enforce a minimum slot size of ~60px and allow horizontal scroll on the spread view if needed

### Tab Bar
- Adding Settings tab (and conditionally Foil tab) means up to 7 tabs total — tab bar must remain scrollable horizontally on small screens (existing `overflow-x: auto` on `.tabs` handles this, verify it still works with additional tabs)

### Confirmation Dialogs (Danger Zone)
- Rendered as a bottom sheet on mobile (slides up from bottom), not a centered modal — easier to reach with thumbs
- On desktop, centered modal is fine

---

## Verification

1. Fresh load (no localStorage) → auto-navigates to Settings tab, wizard shown
2. Complete wizard with 12-pocket preset + Foil Binder enabled → Binder tab shows 4×3 grid, Foil tab appears
3. Open a card modal → "Add Foil ✦" button present
4. Add foil card → Foil tab slot gets shimmer, stats update
5. Go to Settings → compact form shown (not wizard), change to 16-pocket → Binder reflows to 4×4
6. Danger Zone: Reset Collection → confirmation dialog → collection cleared, config preserved
7. Reload → all settings persist via localStorage
8. Enable Collector Variants → save → Scryfall fetch fires, variant cards appear in Binder
9. Gist sync → config + foil data included in payload
10. On a 390px-wide viewport (iPhone): wizard steps are single-column, buttons are full-width and thumb-reachable, tab bar scrolls horizontally when 7 tabs are present
11. Danger Zone confirmation appears as a bottom sheet on mobile, centered modal on desktop
