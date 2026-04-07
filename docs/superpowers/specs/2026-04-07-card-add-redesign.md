# Card Add & Pack Opening Redesign

**Date:** 2026-04-07
**Status:** Approved

## Problem

The floating action button (FAB) obstructs card content and combines two distinct workflows (quick-add and pack opening) into one cramped panel. Both flows are used equally and need to be easily accessible from any tab.

## Solution: Hybrid — Inline Header Search + Pack Opening Modal

Replace the FAB with two header toolbar actions. Each uses the interaction pattern best suited to its purpose.

## Removal Scope

Delete entirely:
- FAB HTML (`#fab-search`, `#fab-btn`, `#fab-panel`, all children)
- FAB CSS (`.fab-search`, `.fab-btn`, `.fab-panel`, `.fab-close`, `.fab-quick-actions`, `.fab-qa-btn`, `.fab-recent`, `.fab-recent-item`, `.fab-empty`, `.fab-panel-header`, `.search-wrap` inside FAB)
- FAB JS (`openFab()`, `closeFab()`, `toggleFab()`, `renderFabRecent()`, FAB event listeners, `fabMode` state variable)
- Pack-mode-inside-FAB JS (`renderPackStep1()`, `renderPackStep2()`, pack autocomplete wiring inside FAB)

Preserve:
- `addCard()`, `removeCard()`, `addFoil()`, `celebrateAdd()`, `celebrateFoil()` — unchanged
- `openModal()` / `closeModal()` — extended with pack context
- `finalizePack()`, `showPackSummary()` — moved into new pack modal
- `packSession`, `currentPack`, `boosterPacks` state — unchanged

---

## Part 1: Header Search Bar

### Resting State
Two new icon buttons in the header alongside the existing sync icon:
- 🔍 Search icon
- 📦 Pack icon

### Expanded State
Tapping 🔍 replaces the header content (title + icons) with:
- 🔍 icon (left)
- Text input field with focus (flex-1, auto-focused, placeholder "Search cards...")
- ESC badge/button (right)

Animated transition: header content slides/fades out, search bar slides/fades in.

### Autocomplete Dropdown
Appears below the header when input has 1+ characters. Same search logic as current FAB:
- Searches `allCards` + `variantCards` by name (case-insensitive) or collector number (exact match)
- Max 15 results

Each result row shows:
- Card thumbnail (small image)
- Card name + variant badge (Extended Art, Showcase, etc.)
- Collector number, rarity (color-coded), type line
- Price (USD + foil price if applicable)
- Owned badge (green border + "✓ Owned") if already in collection
- Chevron (›) indicating it's tappable

**No action buttons in autocomplete.** Rows are purely for selection.

### Result Interaction
- **Tap a row** → opens the existing card modal with full add/remove/foil controls
- **Close modal** → returns to search with query preserved, ready for next card
- **Keyboard nav** → Arrow Up/Down to highlight, Enter to open highlighted card's modal
- **Escape** → closes search, restores header to resting state, clears input

### Dismissal
- Tap ESC badge
- Press Escape key
- Tap outside the autocomplete dropdown (but not the search input)

---

## Part 2: Pack Opening Modal

### Trigger
Tap 📦 icon in header → opens a centered modal overlay (not the card modal — a separate pack-specific modal).

### Step 1 — Price Entry
- Modal title: "📦 Open Booster Pack"
- Close button (✕) top-right
- Price input: "RM" label + number input (step 0.10, default 0)
- "Start Opening" primary button
- Helper text: "Price is optional — leave at 0 to skip"
- Enter key also triggers Start Opening

### Step 2 — Card Scanning
- Modal title: "📦 Pack #N" (where N = boosterPacks.length + 1)
- Price shown in header area: "RM X.XX"
- Close button (✕) with cancel confirmation if cards added

**Search bar** inside the modal:
- Same autocomplete logic as header search
- Results limited to 15, same row format (thumbnail, name, variant, rarity, price)
- Tapping a result opens the card modal **stacked on top** of the pack modal

**Card list** below search:
- Shows cards added to this pack in order (numbered 1, 2, 3...)
- Each entry: sequential number, thumbnail, card name, rarity badge, foil indicator (✦)
- Auto-scrolls to newest card
- Staggered entry animation

**Footer buttons:**
- "Done — Save Pack" (primary, disabled if 0 cards)
- "Cancel" (secondary)

**Card counter:**
- "X cards in this pack" with bump animation on new card

### Step 2b — Card Modal (Stacked)
When a search result is tapped in the pack modal:
- Pack modal dims and scales down slightly (opacity 0.2, scale 0.95)
- Card modal opens on top with modified context:
  - Toggle button reads "Add to Pack" instead of "Add to Binder"
  - Foil button reads "Add as Foil ✦"
  - Helper text: "Also adds to your binder"
  - After adding, modal closes automatically and returns to pack modal
  - Pack modal's card list updates with the new card

### Step 3 — Pack Summary
After "Done — Save Pack":
- `finalizePack()` saves pack to `boosterPacks` array and localStorage
- Summary overlay appears:
  - "Pack #N Opened!" title
  - Card count and price
  - Grid of card thumbnails with rarity-colored borders
  - Best pull highlighted with name and rarity
  - Celebration effects fire (same rarity-tiered system: sparkles, fireworks, screen flash)
- "Close" button dismisses summary
- Auto-dismiss after 3.5 seconds (same as current)

### Cancel Behavior
If the user cancels or closes while cards have been added:
- Confirmation dialog: "Discard this pack? X cards will be removed from your binder."
- Confirm → removes only the cards in `currentPack.cards` from `ownedSet` (not unrelated quick-adds), clears `currentPack`
- Cancel → returns to pack modal

---

## State Changes

| Variable | Change |
|----------|--------|
| `fabMode` | **Remove** — no longer needed |
| `_packModeAdding` | **Keep** — still used to detect card modal context |
| `currentPack` | **Keep** — same `{ price, cards: [] }` structure |
| `packSession` | **Keep** — tracks recent adds for session |
| `acHighlight` | **Keep** — reused for both search contexts |

## Mobile Considerations

- Header search: full-width input, keyboard opens automatically
- Pack modal: full-screen on mobile (same bottom-sheet style as card modal)
- Card modal stacking: card modal slides up as bottom sheet over dimmed pack modal
- ESC badge in header search is a tappable button (not keyboard-only)

## Future Consideration (Shelved)

OCR-based bulk card scanning via Tesseract.js + camera. Could be added as a camera icon in the pack modal search bar. Not in scope for this redesign.
