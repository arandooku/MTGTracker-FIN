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
