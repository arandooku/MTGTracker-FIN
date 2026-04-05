# Foil Celebration & Variants Binder Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add foil add celebrations, a Variant Collection binder tab, foil card shimmer visuals, tab bar UI overhaul, and header cleanup to the MTG FIN tracker.

**Architecture:** Everything lives in `index.html` — one file, no build step. Tasks are ordered from safest/most isolated to most complex. Each task ends with a browser verification step.

**Tech Stack:** Vanilla HTML/CSS/JS, Scryfall API, localStorage.

---

## File Map

| File | Changes |
|------|---------|
| `index.html` | All changes — CSS section, HTML structure, JS functions |

Line number landmarks (approximate, verify before editing):
- Header subtitle HTML: ~line 1894
- Stats row HTML: ~line 1908
- Tabs HTML: ~lines 1911–1918
- Tab CSS `.tab-btn`: ~lines 261–276
- Foil shimmer CSS: ~lines 779–841
- Toast CSS: ~lines 729–744, 1707–1780
- `renderProgress()`: ~line 3709
- `addFoil()`: ~line 2630
- `getCardByNumber()`: ~line 3059
- `spawnGlowRing()`: ~line 3208
- `showAddToast()`: ~line 3216
- `celebrateAdd()`: ~line 3244
- `getVariantLabel()` (modal): ~line 4673
- `applyConfigToTabs()`: ~line 2400
- `fetchVariants()`: ~line 4950
- `renderFoilBinder()`: ~line 5022
- `buildGrid()` inside foil binder: ~line 5116

---

## Task 1: Header Cleanup

**Files:**
- Modify: `index.html` — HTML subtitle block ~line 1894, `renderProgress()` ~line 3709

- [ ] **Step 1: Remove ClearEdge 360 from subtitle HTML**

Find this block (~line 1897):
```html
<p class="subtitle">
  <span>ClearEdge 360</span>
  <span class="sep"></span>
  <span id="binder-preset-display">9-Pocket Binder</span>
  <span class="sep"></span>
  <span id="binder-page-count-display">34 Pages</span>
</p>
```
Replace with:
```html
<p class="subtitle">
  <span id="binder-preset-display">9-Pocket Binder</span>
  <span class="sep"></span>
  <span id="binder-page-count-display">34 Pages</span>
</p>
```

- [ ] **Step 2: Remove rarity stat-badge pills from `renderProgress()`**

Find this block inside `renderProgress()` (~line 3716):
```js
const rarities = ['mythic', 'rare', 'uncommon', 'common'];
document.getElementById('stats-row').innerHTML = rarities.map(r => {
  const tot = allCards.filter(c => c.rarity === r).length;
  const own = allCards.filter(c => c.rarity === r && ownedSet.has(c.collector_number)).length;
  return `<span class="stat-badge ${r}">${r}: ${own}/${tot}</span>`;
}).join('');
```
Replace with:
```js
document.getElementById('stats-row').innerHTML = '';
```

- [ ] **Step 3: Verify in browser**

Open `index.html`. The header subtitle should show only e.g. `9-Pocket Binder · 34 Pages`. Below the progress bar there should be no coloured pill badges.

- [ ] **Step 4: Commit**
```bash
git add index.html
git commit -m "feat: remove ClearEdge 360 label and rarity stat pills from header"
```

---

## Task 2: Main Set Count Fix

**Files:**
- Modify: `index.html` — wizard default and brand display string

- [ ] **Step 1: Update brand-set display string (~line 1894)**

Find:
```html
<div class="brand-set">Main Set &mdash; 300 Cards via Play Boosters</div>
```
Replace with:
```html
<div class="brand-set">Main Set &mdash; 309 Cards via Play Boosters</div>
```

- [ ] **Step 2: Update wizard default card count**

Search for `300` in the context of binder wizard defaults. Find any `var base = allCards.length || 300` (~line 2171) and any wizard HTML that pre-fills `300`. Update `|| 300` → `|| 309`.

Also search for any other hardcoded `300` used in binder math or display:
```bash
grep -n "300" index.html
```
Update any that refer to the main set card count (skip unrelated ones like `300%` in gradients, `300px` in CSS, etc.).

- [ ] **Step 3: Verify in browser**

Open Settings → run the binder wizard. The default card count should reflect 309. The header brand line should read "309 Cards".

- [ ] **Step 4: Commit**
```bash
git add index.html
git commit -m "fix: update main set card count from 300 to 309"
```

---

## Task 3: `getFoilVariantLabel()` and `getFoilSlotClass()` Helpers

**Files:**
- Modify: `index.html` — add two functions near `getCardByNumber()` (~line 3059)

These helpers are shared across foil celebration toast, foil slot rendering, and variant binder rendering. Add them just after `getCardByNumber()`.

- [ ] **Step 1: Add helpers after `getCardByNumber()`**

Find:
```js
function getCardByNumber(cn) {
  return allCards.find(c => c.collector_number === cn);
}
```
Replace with:
```js
function getCardByNumber(cn) {
  return allCards.find(c => c.collector_number === cn)
      || variantCards.find(c => c.collector_number === cn);
}

// Returns uppercase badge label for foil toast and variant chips
function getFoilVariantLabel(card) {
  if (!card) return 'FOIL';
  const fe = card.frame_effects || [];
  const pt = card.promo_types || [];
  if (fe.includes('surge')) return 'SURGE FOIL';
  if (pt.includes('neon_ink')) return 'NEON INK FOIL';
  if ((card.name || '').includes('Traveling Chocobo')) return 'NEON INK FOIL';
  if (fe.includes('serialized')) return 'SERIALIZED';
  if ((card.finishes || []).includes('foil') && !(card.finishes || []).includes('nonfoil')) return 'COLLECTOR FOIL';
  return 'TRADITIONAL FOIL';
}

// Returns CSS variant class for foil slot shimmer overlay
function getFoilSlotClass(card) {
  if (!card) return '';
  const fe = card.frame_effects || [];
  const pt = card.promo_types || [];
  if (fe.includes('surge')) return 'foil-variant-surge';
  if (pt.includes('neon_ink') || (card.name || '').toLowerCase().includes('chocobo')) return 'foil-variant-neon';
  if (fe.includes('serialized')) return 'foil-variant-serialized';
  if ((card.finishes || []).every(f => f !== 'nonfoil')) return 'foil-variant-collector';
  return '';
}
```

Note: `getCardByNumber` is extended to also search `variantCards` — this ensures `celebrateFoil` works when a foil is added via the foil tab for a variant card.

- [ ] **Step 2: Verify in browser console**

Open browser devtools console. After page loads:
```js
getFoilVariantLabel({ frame_effects: ['surge'], finishes: ['foil'] })
// → "SURGE FOIL"
getFoilVariantLabel({ finishes: ['foil', 'nonfoil'] })
// → "TRADITIONAL FOIL"
getFoilSlotClass({ frame_effects: ['surge'] })
// → "foil-variant-surge"
```

- [ ] **Step 3: Commit**
```bash
git add index.html
git commit -m "feat: add getFoilVariantLabel and getFoilSlotClass helpers, extend getCardByNumber to search variantCards"
```

---

## Task 4: Foil Card Visual CSS

**Files:**
- Modify: `index.html` — CSS section, foil binder styles block (~line 779)

- [ ] **Step 1: Add new keyframes and foil overlay CSS**

Find the existing foil CSS block that starts with `/* ── Foil Binder ── */` (~line 778). Add the following **after** the existing `.slot-foil-missing` and `.foil-badge` rules (after ~line 801):

```css
/* ── Foil card shimmer overlay (::after on owned slots) ── */
@keyframes foil-card-shimmer {
  0%   { background-position: 0% 0%; }
  50%  { background-position: 100% 100%; }
  100% { background-position: 0% 0%; }
}
@keyframes foil-neon-pulse {
  0%, 100% { opacity: 0.4; }
  50%       { opacity: 1; }
}

/* Base overlay — traditional foil rainbow */
.foil-slot.slot-foil-owned::after {
  content: ''; position: absolute; inset: 0; border-radius: 4px;
  pointer-events: none; z-index: 1;
  will-change: background-position;
  background: linear-gradient(115deg,
    transparent 25%, rgba(255,107,157,0.3) 35%,
    rgba(77,150,255,0.25) 48%, rgba(107,203,119,0.25) 58%,
    rgba(255,217,61,0.3) 68%, transparent 78%);
  background-size: 300% 300%;
  mix-blend-mode: screen;
  animation: foil-card-shimmer 4s ease-in-out infinite;
}
/* Surge — electric blue-white, faster */
.foil-slot.foil-variant-surge.slot-foil-owned::after {
  background: linear-gradient(115deg,
    transparent 20%, rgba(160,210,255,0.5) 35%,
    rgba(255,255,255,0.55) 50%, rgba(120,175,255,0.5) 65%,
    transparent 80%);
  animation-duration: 2.5s;
}
/* Neon ink — pulsing radial glow */
.foil-slot.foil-variant-neon.slot-foil-owned::after {
  background: radial-gradient(ellipse at center,
    rgba(255,80,180,0.35) 0%, rgba(80,220,255,0.25) 50%, transparent 75%);
  background-size: 100% 100%;
  animation: foil-neon-pulse 2s ease-in-out infinite;
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
/* Stagger delays so cards don't pulse in sync */
.foil-delay-1::after { animation-delay: 1.35s; }
.foil-delay-2::after { animation-delay: 2.7s; }

/* Variant-specific border glow overrides */
.foil-slot.foil-variant-surge.slot-foil-owned {
  box-shadow: 0 0 14px rgba(100,180,255,0.45) !important;
}
.foil-slot.foil-variant-neon.slot-foil-owned {
  box-shadow: 0 0 14px rgba(255,80,200,0.5) !important;
}
.foil-slot.foil-variant-serialized.slot-foil-owned {
  box-shadow: 0 0 14px rgba(255,215,0,0.5) !important;
}
.foil-slot.foil-variant-collector.slot-foil-owned {
  box-shadow: 0 0 14px rgba(200,220,255,0.3) !important;
}
```

- [ ] **Step 2: Simplify `.slot-foil-owned` — remove hidden background gradient**

Find `.slot-foil-owned` (~line 784):
```css
.slot-foil-owned {
  background: linear-gradient(135deg,
    rgba(255,107,157,0.25),rgba(255,217,61,0.25),
    rgba(107,203,119,0.25),rgba(77,150,255,0.25),rgba(199,125,255,0.25)) !important;
  background-size: 300% 300% !important;
  animation: foil-shimmer 3s ease infinite;
  border: 1.5px solid rgba(255,255,255,0.3) !important;
  box-shadow: 0 0 14px rgba(255,200,100,0.25) !important;
}
```
Replace with (keep border and glow, remove hidden background — the `::after` overlay does the visual work):
```css
.slot-foil-owned {
  border: 1.5px solid rgba(255,255,255,0.3) !important;
  box-shadow: 0 0 14px rgba(255,200,100,0.3) !important;
}
```

- [ ] **Step 3: Verify in browser**

Open `index.html`. Navigate to the Foil Collection tab. Any foil-owned cards should now show the shimmer overlay directly on the card image. Owned cards with no foil set won't show the `::after` yet (that comes when slot rendering adds the variant class — this task just sets up the CSS).

To preview now: open devtools, find a `.foil-slot` element, manually add class `slot-foil-owned` and `foil-delay-1` in the Elements panel. You should see the rainbow shimmer on the card.

- [ ] **Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add foil card ::after shimmer overlay CSS with per-variant animations"
```

---

## Task 5: Foil Celebration — Toast CSS, Functions, and `addFoil()` Wiring

**Files:**
- Modify: `index.html` — Toast CSS block, JS celebration section (~line 3216)

### Part A: CSS

- [ ] **Step 1: Add foil toast CSS**

Find the `/* ── Toast ── */` CSS section (~line 729). After the existing `.toast.show` rule block, add:

```css
/* Foil add toast */
.add-toast.foil {
  --toast-glow: rgba(199,125,255,0.4);
  border: 1.5px solid rgba(255,255,255,0.2);
}
.toast-foil-badge {
  font-weight: 900; text-transform: uppercase;
  letter-spacing: 2px; padding: 5px 18px;
  border-radius: 20px; margin-bottom: 14px;
  font-size: 0.85rem; color: #fff;
  background: linear-gradient(135deg, #ff6b9d, #ffd93d, #6bcb77, #4d96ff, #c77dff);
  background-size: 300% 300%;
  animation: foil-shimmer 3s ease infinite, badge-pop 0.4s 0.2s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}
.toast-foil-msg {
  font-size: 1.05rem; font-weight: 700;
  background: linear-gradient(135deg, #ff6b9d, #ffd93d, #6bcb77, #4d96ff, #c77dff);
  background-size: 300% 300%;
  animation: foil-shimmer 3s ease infinite;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

### Part B: JS Functions

- [ ] **Step 2: Add `spawnRainbowBurst()` after `spawnGlowRing()`**

Find `function spawnGlowRing(color)` (~line 3208). After its closing `}`, add:

```js
function spawnRainbowBurst() {
  const colors = ['#ff6b9d', '#ffd93d', '#6bcb77', '#4d96ff', '#c77dff'];
  const container = document.createElement('div');
  container.className = 'fx-container';
  document.body.appendChild(container);
  const cx = window.innerWidth / 2;
  const cy = window.innerHeight / 2;

  for (let i = 0; i < 40; i++) {
    const p = document.createElement('div');
    const color = colors[i % colors.length];
    const angle = (Math.random() * Math.PI * 2);
    const dist = 80 + Math.random() * 220;
    const size = 4 + Math.random() * 6;
    p.style.cssText = `
      position: fixed;
      left: ${cx}px; top: ${cy}px;
      width: ${size}px; height: ${size}px;
      border-radius: 50%;
      background: ${color};
      pointer-events: none;
      z-index: 9999;
      box-shadow: 0 0 ${size * 2}px ${color};
    `;
    p.animate([
      { transform: 'translate(-50%, -50%) scale(1)', opacity: 1 },
      { transform: `translate(calc(-50% + ${Math.cos(angle) * dist}px), calc(-50% + ${Math.sin(angle) * dist}px)) scale(0.2)`, opacity: 0 },
    ], {
      duration: 900 + Math.random() * 600,
      delay: Math.random() * 200,
      easing: 'cubic-bezier(0.25, 0, 0.3, 1)',
      fill: 'forwards',
    });
    container.appendChild(p);
  }
  setTimeout(() => container.remove(), 1800);
}
```

- [ ] **Step 3: Add `showFoilToast()` after `showAddToast()`**

Find `function showAddToast(card, cfg)` (~line 3216). After its closing `}` (before `function celebrateAdd`), add:

```js
function showFoilToast(card) {
  const msgs = [
    'Shimmer acquired.', 'It catches the light.', 'The foil is real.',
    'Prismatic.', "That one's a keeper.", 'Foil secured. \u2726', 'Ooh, shiny.'
  ];
  const msg = msgs[Math.floor(Math.random() * msgs.length)];
  const label = getFoilVariantLabel(card);
  const p = getPlacement(card);
  const price = card.price_usd_foil > 0 ? card.price_usd_foil : card.price_usd;
  const priceStr = price > 0 ? ` \u00B7 $${price.toFixed(2)}${card.price_usd_foil > 0 ? ' foil' : ''}` : '';

  const container = document.getElementById('toast-container');
  container.querySelectorAll('.add-toast').forEach(t => t.remove());

  const toast = document.createElement('div');
  toast.className = 'add-toast foil';
  toast.style.setProperty('--toast-glow', 'rgba(199,125,255,0.4)');
  toast.innerHTML = `
    <div class="toast-foil-badge">${label}</div>
    <img src="${card.image_normal || card.image_small}" alt="${card.name}">
    <div class="toast-name">#${card.collector_number} ${card.name}</div>
    <div class="toast-foil-msg">${msg}</div>
    <div class="toast-placement">Page ${p.page}, Slot ${p.slot}${priceStr}</div>
  `;
  container.appendChild(toast);

  const dismissTime = 3000;
  setTimeout(() => {
    toast.classList.add('removing');
    setTimeout(() => toast.remove(), 400);
  }, dismissTime);
}
```

- [ ] **Step 4: Add `celebrateFoil()` after `celebrateAdd()`**

Find `function celebrateAdd(card)` (~line 3244). After its closing `}`, add:

```js
function celebrateFoil(card) {
  if (!card) return;
  spawnGlowRing('rgba(199,125,255,0.7)');
  spawnRainbowBurst();
  showFoilToast(card);
}
```

- [ ] **Step 5: Wire `celebrateFoil` into `addFoil()`**

Find `function addFoil(cn)` (~line 2630):
```js
function addFoil(cn) {
  if (!foilOwned[cn]) foilOwned[cn] = [];
  if (!foilOwned[cn].includes('foil')) foilOwned[cn].push('foil');
  saveFoil();
}
```
Replace with:
```js
function addFoil(cn) {
  if (!foilOwned[cn]) foilOwned[cn] = [];
  if (!foilOwned[cn].includes('foil')) foilOwned[cn].push('foil');
  saveFoil();
  const card = getCardByNumber(cn);
  celebrateFoil(card);
}
```

- [ ] **Step 6: Verify in browser**

Open `index.html`. Open any card modal for a foil-capable card. Click "Add Foil ✦". You should see:
- Rainbow particles burst from center of screen
- A glow ring pulse
- Foil toast appears with shimmer badge (e.g. "TRADITIONAL FOIL"), rainbow shimmer message text, card image, and price

Also test via FAB search: type a card name, click the ✦ foil quick-add button in the autocomplete. Same celebration should fire.

- [ ] **Step 7: Commit**
```bash
git add index.html
git commit -m "feat: foil add celebration — rainbow burst, shimmer toast, celebrateFoil function"
```

---

## Task 6: Tab Bar UI Overhaul + Tab Renames

**Files:**
- Modify: `index.html` — `.tabs` and `.tab-btn` CSS (~line 257), tab HTML (~line 1911)

### Part A: CSS

- [ ] **Step 1: Replace tab bar CSS**

Find the `/* ── Tabs ── */` section (~line 256). Replace the entire `.tabs` and `.tab-btn` block with:

```css
/* ── Tabs ── */
.tabs {
  display: flex; gap: 4px; margin-bottom: 20px;
  background: var(--surface); border-radius: 12px;
  padding: 4px; overflow-x: auto;
  -webkit-overflow-scrolling: touch; scrollbar-width: none;
}
.tabs::-webkit-scrollbar { display: none; }
.tab-btn {
  flex: 1; min-width: 0;
  display: flex; align-items: center; justify-content: center; gap: 5px;
  padding: clamp(7px, 1.2vw, 9px) clamp(8px, 1.5vw, 14px);
  background: none; border: none; border-radius: 9px;
  color: var(--text-muted);
  font-size: clamp(0.72rem, 1.4vw, 0.85rem);
  font-weight: 500; cursor: pointer; white-space: nowrap;
  transition: background 0.15s, color 0.15s;
  flex-shrink: 0;
}
.tab-btn svg {
  width: clamp(12px, 1.8vw, 15px); height: clamp(12px, 1.8vw, 15px);
  flex-shrink: 0;
}
.tab-btn:hover { background: var(--surface2); color: var(--text); }
.tab-btn.active {
  background: var(--accent);
  color: #fff;
  font-weight: 700;
  box-shadow: 0 2px 8px rgba(233,69,96,0.35);
}
```

Remove the old `.tab-btn[data-tab="foil"]` entrance animation rule (~line 804) — it referenced the old tab by name and is no longer needed.

- [ ] **Step 2: Update mobile tab CSS**

Find the `@media (max-width: 600px)` block that has `.tabs` overrides (~line 1162). Remove any `.tabs` border-bottom overrides that are now obsolete. The mobile tabs already have `overflow-x: auto` from the new rule above.

### Part B: HTML

- [ ] **Step 3: Replace tab buttons HTML**

Find the `<div class="tabs">` block (~line 1911). Replace the entire block:

```html
<div class="tabs">
  <button class="tab-btn active" data-tab="dashboard">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>
    Dashboard
  </button>
  <button class="tab-btn" data-tab="portfolio">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
    Portfolio
  </button>
  <button class="tab-btn" data-tab="timeline">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
    Timeline
  </button>
  <button class="tab-btn" data-tab="binder">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/></svg>
    Binder
  </button>
  <button class="tab-btn" data-tab="foil" id="tab-btn-foil" style="display:none">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>
    Foil Collection
  </button>
  <button class="tab-btn" data-tab="variants" id="tab-btn-variants" style="display:none">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="7" width="20" height="14" rx="2"/><path d="M16 7V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v2"/></svg>
    Variant Collection
  </button>
  <button class="tab-btn" data-tab="settings">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/></svg>
    Settings
  </button>
</div>
```

- [ ] **Step 4: Update `applyConfigToTabs()` to also gate the variants tab**

Find `function applyConfigToTabs()` (~line 2400):
```js
function applyConfigToTabs() {
  var foilBtn = document.getElementById('tab-btn-foil');
  if (foilBtn) foilBtn.style.display = binderConfig.scope.foilBinder ? '' : 'none';
  ...
}
```
Add the variants tab line inside the function:
```js
var variantBtn = document.getElementById('tab-btn-variants');
if (variantBtn) variantBtn.style.display = binderConfig.scope.collectorVariants ? '' : 'none';
```

- [ ] **Step 5: Verify in browser**

Open `index.html`. The tab bar should show a rounded pill tray background. The active tab should have a filled accent-coloured capsule (red/pink background, white text). All tabs should have consistent sizes. Foil/Settings tabs should have SVG icons instead of text glyphs. "Main Dashboard" is now just "Dashboard", "Binder View" is "Binder".

- [ ] **Step 6: Commit**
```bash
git add index.html
git commit -m "feat: tab bar pill tray redesign, rename Foil→Foil Collection, add Variant Collection tab button"
```

---

## Task 7: Foil Tab — Pagination, Filter Fix, and Shimmer Integration

**Files:**
- Modify: `index.html` — `renderFoilBinder()` function (~line 5022), specifically `buildGrid()` inside it

- [ ] **Step 1: Add foil page tracking and reset on filter change**

Inside `renderFoilBinder()`, the filter state is stored on the element. Add `foilPage` tracking. Find the filter state block (~line 5026):
```js
let activeFilter = el._foilFilter || 'All';
let activeRarity = el._foilRarity || '';
let searchQuery  = el._foilSearch || '';

function persist() {
  el._foilFilter = activeFilter;
  el._foilRarity = activeRarity;
  el._foilSearch = searchQuery;
}
```
Replace with:
```js
let activeFilter = el._foilFilter || 'All';
let activeRarity = el._foilRarity || '';
let searchQuery  = el._foilSearch || '';
let foilPage     = el._foilPage  || 0;

function persist() {
  el._foilFilter = activeFilter;
  el._foilRarity = activeRarity;
  el._foilSearch = searchQuery;
  el._foilPage   = foilPage;
}
function resetPage() { foilPage = 0; el._foilPage = 0; }
```

- [ ] **Step 2: Fix `buildGrid()` — corrected filters, pagination, shimmer classes, dimming**

Find the `function buildGrid()` block inside `renderFoilBinder()` (~line 5116). Replace the entire function:

```js
function buildGrid() {
  const spp = binderConfig.slotsPerPage;
  let cards;

  if (activeFilter === 'Owned') {
    cards = [...allCards, ...variantCards]
      .filter(c => hasFoil(c.collector_number))
      .sort((a, b) => compareCollector(a.collector_number, b.collector_number));
  } else if (activeFilter === 'Surge') {
    cards = variantCards.filter(c => (c.frame_effects || []).includes('surge'));
  } else if (activeFilter === 'Neon Ink') {
    cards = variantCards.filter(c => (c.promo_types || []).includes('neon_ink') || (c.name || '').toLowerCase().includes('chocobo'));
  } else if (activeFilter === 'Chocobo Track') {
    cards = variantCards.filter(c => (c.name || '').toLowerCase().includes('chocobo'));
  } else if (activeFilter === 'Traditional') {
    cards = allCards.slice().sort((a, b) => compareCollector(a.collector_number, b.collector_number));
  } else {
    // All
    cards = [...allCards].sort((a, b) => compareCollector(a.collector_number, b.collector_number));
  }

  if (activeRarity) {
    const rarityMap = { C: 'common', U: 'uncommon', R: 'rare', M: 'mythic' };
    cards = cards.filter(c => c.rarity === rarityMap[activeRarity]);
  }
  if (searchQuery.trim()) {
    const q = searchQuery.trim().toLowerCase();
    cards = cards.filter(c => c.name.toLowerCase().includes(q));
  }

  if (cards.length === 0) {
    return `<div style="padding:32px;text-align:center;color:var(--text-muted)">No cards match the current filters.</div>`;
  }

  const totalPages = Math.ceil(cards.length / spp);
  if (foilPage >= totalPages) foilPage = totalPages - 1;
  const pageCards = cards.slice(foilPage * spp, (foilPage + 1) * spp);

  const slotsHTML = pageCards.map((card, idx) => {
    const isFoilOwned = hasFoil(card.collector_number);
    const variantClass = isFoilOwned ? getFoilSlotClass(card) : '';
    const delayClass = isFoilOwned ? `foil-delay-${idx % 3}` : '';
    const slotCls = isFoilOwned ? 'slot-foil-owned' : 'slot-foil-missing';
    const imgStyle = isFoilOwned
      ? 'width:100%;height:100%;object-fit:cover;border-radius:4px'
      : 'width:100%;height:100%;object-fit:cover;border-radius:4px;filter:grayscale(1) brightness(0.25);opacity:0.5';
    const imgSrc = card.image_normal || card.image_small;
    return `<div class="card-slot foil-slot ${slotCls} ${variantClass} ${delayClass}" data-cn="${card.collector_number}" style="position:relative">
      <img data-src="${imgSrc}" alt="${card.name}" style="${imgStyle}">
      ${isFoilOwned ? '<span class="foil-badge">\u2726</span>' : ''}
    </div>`;
  }).join('');

  const navHTML = `
    <div style="display:flex;align-items:center;justify-content:center;gap:12px;margin-top:14px">
      <button class="btn btn-secondary foil-prev" style="padding:6px 16px" ${foilPage === 0 ? 'disabled' : ''}>&#8592; Prev</button>
      <span style="font-size:0.85rem;color:var(--text-muted)">Page ${foilPage + 1} of ${totalPages}</span>
      <button class="btn btn-secondary foil-next" style="padding:6px 16px" ${foilPage >= totalPages - 1 ? 'disabled' : ''}>Next &#8594;</button>
    </div>`;

  return `<div class="page-grid foil-card-grid" style="grid-template-columns:repeat(${binderConfig.gridCols},1fr);gap:6px">${slotsHTML}</div>${navHTML}`;
}
```

- [ ] **Step 3: Wire up pagination buttons in `renderFoilBinder()`**

After the `el.innerHTML = ...` assignment (where the full foil tab HTML is set), find where the filter button click handlers are wired up. In the same event delegation block, add handlers for the new prev/next buttons. Look for the existing click handler on `el` (it handles `.foil-filter-type`, `.foil-filter-rarity`, `.foil-search-input`). Add inside the same handler:

```js
if (e.target.classList.contains('foil-prev')) {
  foilPage = Math.max(0, foilPage - 1);
  persist();
  const gridEl = el.querySelector('.foil-card-grid')?.parentElement || el.querySelector('[class*="foil"]');
  // Re-render just the grid section
  renderFoilBinder();
  return;
}
if (e.target.classList.contains('foil-next')) {
  foilPage++;
  persist();
  renderFoilBinder();
  return;
}
```

- [ ] **Step 4: Trigger lazy image loading after grid renders**

The existing `lazyLoadFoilImages()` function should already be called after the grid renders. Verify this call still happens after `buildGrid()` output is inserted into the DOM. If the function is called at the end of `renderFoilBinder()`, no change is needed. If not, add `lazyLoadFoilImages()` after the grid HTML is inserted.

- [ ] **Step 5: Verify in browser**

Open `index.html`. Enable "Foil Binder" in Settings → save. Go to the "Foil Collection" tab:
- Should show a grid of max 9 cards (or whatever `slotsPerPage` is set to)
- Prev/Next buttons and "Page 1 of N" should appear below the grid
- Owned foil cards should show shimmer overlay animation on the card image
- Unowned cards should be heavily darkened (grayscale, dim)
- Surge filter should work (if any surge variant cards are loaded)
- Neon Ink filter should work (if variant cards loaded)

- [ ] **Step 6: Commit**
```bash
git add index.html
git commit -m "feat: foil tab pagination, corrected Surge/Neon Ink filters, foil shimmer on card images, stronger unowned dimming"
```

---

## Task 8: Variant Collection Tab

**Files:**
- Modify: `index.html` — add tab content HTML, `renderVariantBinder()` function, update `fetchVariants()` and tab click wiring

### Part A: HTML

- [ ] **Step 1: Add Variant Collection tab content div**

Find the existing `<div class="tab-content" id="tab-foil">` element. After its closing `</div>`, add:

```html
<div class="tab-content" id="tab-variants"></div>
```

### Part B: `renderVariantBinder()` Function

- [ ] **Step 2: Add `renderVariantBinder()` after `renderFoilBinder()`**

Find the end of `renderFoilBinder()` (its closing `}`). After it, add the full function:

```js
function renderVariantBinder() {
  const el = document.getElementById('tab-variants');
  if (!el) return;

  // ── Filter state ──
  let activeFilter = el._variantFilter || 'All';
  let activeRarity = el._variantRarity || '';
  let searchQuery  = el._variantSearch || '';
  let variantPage  = el._variantPage  || 0;

  function persist() {
    el._variantFilter = activeFilter;
    el._variantRarity = activeRarity;
    el._variantSearch = searchQuery;
    el._variantPage   = variantPage;
  }
  function resetPage() { variantPage = 0; el._variantPage = 0; }

  // ── Stats ──
  const ownedVariantCount = variantCards.filter(c => ownedSet.has(c.collector_number)).length;
  const variantTotal = variantCards.length;
  const ownedPct = variantTotal > 0 ? (ownedVariantCount / variantTotal * 100) : 0;
  const ownedFoilCount = variantCards.filter(c => hasFoil(c.collector_number)).length;
  const variantValue = variantCards
    .filter(c => ownedSet.has(c.collector_number))
    .reduce((s, c) => s + Math.max(c.price_usd || 0, c.price_usd_foil || 0), 0);

  const statsHTML = `
    <div class="dash-grid" style="margin-bottom:18px">
      <div class="dash-card">
        <h3>Variants Owned</h3>
        <div class="dash-big-num" style="font-size:1.4rem">${ownedVariantCount} / ${variantTotal}</div>
        <div class="dash-sub">${ownedPct.toFixed(1)}% complete</div>
        <div class="rarity-track" style="margin:8px 0 4px">
          <div class="rarity-fill rare" style="width:${ownedPct.toFixed(1)}%"></div>
        </div>
      </div>
      <div class="dash-card">
        <h3>✦ Foils Owned</h3>
        <div class="dash-big-num" style="font-size:1.4rem">${ownedFoilCount}</div>
        <div class="dash-sub">of ${variantTotal} variant cards</div>
      </div>
      <div class="dash-card">
        <h3>Variant Value</h3>
        <div class="dash-big-num" style="font-size:1.4rem">$${variantValue.toFixed(2)}</div>
        <div class="dash-sub">RM ${(variantValue * USD_TO_MYR).toFixed(2)} (est.)</div>
      </div>
    </div>`;

  if (variantCards.length === 0) {
    el.innerHTML = `<div style="padding:48px;text-align:center;color:var(--text-muted)">
      <div style="font-size:2rem;margin-bottom:12px">◈</div>
      <div>No variant cards loaded.</div>
      <div style="margin-top:8px;font-size:0.85rem">Collector Variants must be enabled in Settings.</div>
    </div>`;
    return;
  }

  // ── Filter bar ──
  const filterTypes = ['All', 'Owned', 'Traditional', 'Surge', 'Neon Ink', 'Chocobo Track'];
  const rarities = ['C', 'U', 'R', 'M'];
  const rarityMap = { C: 'common', U: 'uncommon', R: 'rare', M: 'mythic' };

  const filterBarHTML = `
    <div style="display:flex;flex-wrap:wrap;gap:8px;align-items:center;margin-bottom:14px">
      <div style="display:flex;gap:6px;flex-wrap:wrap">
        ${filterTypes.map(f => `<button class="btn ${activeFilter === f ? 'btn-primary' : 'btn-secondary'} variant-filter-type" data-type="${f}" style="padding:4px 12px;font-size:0.82rem">${f}</button>`).join('')}
      </div>
      <div style="display:flex;gap:5px">
        ${rarities.map(r => `<button class="btn ${activeRarity === r ? 'btn-primary' : 'btn-secondary'} variant-filter-rarity" data-rarity="${r}" style="padding:4px 10px;font-size:0.8rem;font-weight:700">${r}</button>`).join('')}
      </div>
      <input type="text" class="variant-search-input" placeholder="Search card name\u2026" value="${searchQuery.replace(/"/g, '&quot;')}"
        style="padding:5px 10px;border-radius:8px;border:1px solid var(--surface3);background:var(--surface2);color:var(--text);font-size:0.85rem;min-width:160px">
    </div>`;

  // ── Grid builder ──
  function buildVariantGrid() {
    const spp = binderConfig.slotsPerPage;
    let cards;

    if (activeFilter === 'Owned') {
      cards = variantCards.filter(c => ownedSet.has(c.collector_number));
    } else if (activeFilter === 'Surge') {
      cards = variantCards.filter(c => (c.frame_effects || []).includes('surge'));
    } else if (activeFilter === 'Neon Ink') {
      cards = variantCards.filter(c => (c.promo_types || []).includes('neon_ink') || (c.name || '').toLowerCase().includes('chocobo'));
    } else if (activeFilter === 'Chocobo Track') {
      cards = variantCards.filter(c => (c.name || '').toLowerCase().includes('chocobo'));
    } else if (activeFilter === 'Traditional') {
      cards = variantCards.filter(c => {
        const fe = c.frame_effects || [];
        const pt = c.promo_types || [];
        return !fe.includes('surge') && !fe.includes('serialized')
            && !pt.includes('neon_ink')
            && !(c.name || '').toLowerCase().includes('chocobo');
      });
    } else {
      cards = variantCards.slice();
    }

    if (activeRarity) {
      cards = cards.filter(c => c.rarity === rarityMap[activeRarity]);
    }
    if (searchQuery.trim()) {
      const q = searchQuery.trim().toLowerCase();
      cards = cards.filter(c => c.name.toLowerCase().includes(q));
    }

    if (cards.length === 0) {
      return `<div style="padding:32px;text-align:center;color:var(--text-muted)">No cards match the current filters.</div>`;
    }

    const totalPages = Math.ceil(cards.length / spp);
    if (variantPage >= totalPages) variantPage = totalPages - 1;
    const pageCards = cards.slice(variantPage * spp, (variantPage + 1) * spp);

    const slotsHTML = pageCards.map((card, idx) => {
      const isOwned = ownedSet.has(card.collector_number);
      const isFoilOwned = hasFoil(card.collector_number);
      const variantClass = isFoilOwned ? getFoilSlotClass(card) : '';
      const delayClass = isFoilOwned ? `foil-delay-${idx % 3}` : '';
      const foilSlotCls = isFoilOwned ? 'slot-foil-owned' : '';
      const ownedSlotCls = isOwned ? 'slot-owned' : '';
      const imgStyle = isOwned
        ? 'width:100%;height:100%;object-fit:cover;border-radius:4px'
        : 'width:100%;height:100%;object-fit:cover;border-radius:4px;filter:grayscale(1) brightness(0.25);opacity:0.5';
      const label = getFoilVariantLabel(card);
      const imgSrc = card.image_normal || card.image_small;
      return `<div class="card-slot foil-slot ${ownedSlotCls} ${foilSlotCls} ${variantClass} ${delayClass}" data-cn="${card.collector_number}" style="position:relative;cursor:pointer">
        <img data-src="${imgSrc}" alt="${card.name}" style="${imgStyle}">
        ${isFoilOwned ? '<span class="foil-badge">\u2726</span>' : ''}
        <span style="position:absolute;bottom:2px;left:2px;right:2px;font-size:0.5rem;color:rgba(255,255,255,0.7);text-align:center;pointer-events:none;line-height:1.2;background:rgba(0,0,0,0.45);border-radius:0 0 4px 4px;padding:1px 2px">${label}</span>
      </div>`;
    }).join('');

    const navHTML = `
      <div style="display:flex;align-items:center;justify-content:center;gap:12px;margin-top:14px">
        <button class="btn btn-secondary variant-prev" style="padding:6px 16px" ${variantPage === 0 ? 'disabled' : ''}>&#8592; Prev</button>
        <span style="font-size:0.85rem;color:var(--text-muted)">Page ${variantPage + 1} of ${totalPages}</span>
        <button class="btn btn-secondary variant-next" style="padding:6px 16px" ${variantPage >= totalPages - 1 ? 'disabled' : ''}>Next &#8594;</button>
      </div>`;

    return `<div class="page-grid foil-card-grid" style="grid-template-columns:repeat(${binderConfig.gridCols},1fr);gap:6px">${slotsHTML}</div>${navHTML}`;
  }

  // ── Render ──
  el.innerHTML = statsHTML + filterBarHTML + `<div id="variant-grid-wrap">${buildVariantGrid()}</div>`;
  el.dataset.variantRendered = '1';

  // ── Lazy load images ──
  el.querySelectorAll('img[data-src]').forEach(img => {
    const obs = new IntersectionObserver(entries => {
      entries.forEach(e => {
        if (e.isIntersecting) { e.target.src = e.target.dataset.src; obs.unobserve(e.target); }
      });
    });
    obs.observe(img);
  });

  // ── Event delegation ──
  el.addEventListener('click', function handleVariantClick(e) {
    // Pagination
    if (e.target.classList.contains('variant-prev')) {
      variantPage = Math.max(0, variantPage - 1); persist(); renderVariantBinder(); return;
    }
    if (e.target.classList.contains('variant-next')) {
      variantPage++; persist(); renderVariantBinder(); return;
    }
    // Filter type buttons
    const typeBtn = e.target.closest('.variant-filter-type');
    if (typeBtn) {
      activeFilter = typeBtn.dataset.type; resetPage(); persist(); renderVariantBinder(); return;
    }
    // Rarity pills
    const rarBtn = e.target.closest('.variant-filter-rarity');
    if (rarBtn) {
      activeRarity = activeRarity === rarBtn.dataset.rarity ? '' : rarBtn.dataset.rarity;
      resetPage(); persist(); renderVariantBinder(); return;
    }
    // Slot click — toggle ownership
    const slot = e.target.closest('.card-slot[data-cn]');
    if (slot) {
      const cn = slot.dataset.cn;
      if (ownedSet.has(cn)) removeCard(cn);
      else addCard(cn);
      renderVariantBinder();
      return;
    }
  }, { once: true });

  // Search input
  const searchEl = el.querySelector('.variant-search-input');
  if (searchEl) {
    searchEl.addEventListener('input', function() {
      searchQuery = this.value; resetPage(); persist(); renderVariantBinder();
    });
  }
}
```

### Part C: Wire Up Tab Click and Data Fetch

- [ ] **Step 3: Add variant tab click handler**

Find the existing tab click handler setup. It typically looks like:
```js
document.querySelectorAll('.tab-btn').forEach(btn => {
  btn.addEventListener('click', function() {
    ...
    if (this.dataset.tab === 'foil') renderFoilBinder();
    ...
  });
});
```
Add inside the same handler:
```js
if (this.dataset.tab === 'variants') renderVariantBinder();
```

- [ ] **Step 4: Trigger variant re-render from `addCard` / `removeCard`**

Find `function addCard(cn)` (~line 3268). At the end of the function (after the other `if active tab` blocks), add:
```js
if (document.getElementById('tab-variants')?.classList.contains('active')) renderVariantBinder();
```

Find `function removeCard(cn)` (~line 3292). Similarly add:
```js
if (document.getElementById('tab-variants')?.classList.contains('active')) renderVariantBinder();
```

- [ ] **Step 5: Trigger variant binder render from `fetchVariants()`**

Find the end of `fetchVariants()` (~line 5014):
```js
if (document.getElementById('tab-foil')?.classList.contains('active')) {
  renderFoilBinder();
}
```
Add after it:
```js
if (document.getElementById('tab-variants')?.classList.contains('active')) {
  renderVariantBinder();
}
```

- [ ] **Step 6: Verify in browser**

Open `index.html`. In Settings, enable "Collector Variants" and save. The "Variant Collection" tab should appear in the tab bar. Click it. You should see:
- Stats cards (Variants Owned, Foils Owned, Variant Value)
- Filter bar: All / Owned / Traditional / Surge / Neon Ink / Chocobo Track + rarity pills + search
- Paginated card grid — unowned cards heavily dimmed
- Owned cards with normal brightness; foil-owned cards with shimmer overlay
- Each slot has a small variant type label (e.g. "SURGE FOIL") at the bottom
- Clicking a slot toggles ownership and re-renders

- [ ] **Step 7: Commit**
```bash
git add index.html
git commit -m "feat: Variant Collection tab with paginated grid, filter bar, foil shimmer, slot click ownership toggle"
```

---

## Self-Review

### Spec Coverage Check

| Spec requirement | Task covering it |
|-----------------|-----------------|
| Foil add celebration from modal + FAB | Task 5 (`addFoil` wiring) |
| `getFoilVariantLabel()` helper | Task 3 |
| `spawnRainbowBurst()` | Task 5 |
| `showFoilToast()` with shimmer badge + messages | Task 5 |
| `celebrateFoil()` | Task 5 |
| Main set count 300 → 309 | Task 2 |
| Variant Collection tab (gated on `collectorVariants`) | Task 8 |
| Variant tab slot click → addCard/removeCard | Task 8 |
| Variant tab filter bar (corrected logic) | Task 8 |
| Foil tab filter logic fix (Surge, Neon Ink, Chocobo) | Task 7 |
| Foil tab pagination | Task 7 |
| Foil tab unowned dimming | Task 7 |
| Foil card `::after` shimmer overlay CSS | Task 4 |
| Per-variant shimmer classes (surge, neon, serialized, collector) | Task 4 |
| Stagger delay classes | Task 4 |
| Slot border/glow cleanup | Task 4 |
| Tab bar pill tray redesign | Task 6 |
| Tab renames (Foil Collection, Variant Collection) | Task 6 |
| SVG icon consistency (foil, settings) | Task 6 |
| Remove ClearEdge 360 | Task 1 |
| Remove rarity stat-badge pills | Task 1 |
| Variant tab stats cards | Task 8 |
| Variant tab foil shimmer on owned slots | Task 8 (uses Task 4 CSS + Task 3 helpers) |

All spec requirements are covered.

### Type / Name Consistency

- `getFoilVariantLabel(card)` — defined Task 3, used Tasks 5 and 8 ✓
- `getFoilSlotClass(card)` — defined Task 3, used Tasks 7 and 8 ✓
- `getCardByNumber(cn)` — extended Task 3, used Task 5 ✓
- `celebrateFoil(card)` — defined Task 5, called from `addFoil` Task 5 ✓
- `foil-variant-surge` / `foil-variant-neon` / `foil-variant-serialized` / `foil-variant-collector` — CSS defined Task 4, classes applied Tasks 7 and 8 ✓
- `foil-delay-0/1/2` — CSS defined Task 4, applied Tasks 7 and 8 ✓
- `tab-btn-variants` / `tab-variants` — HTML added Task 6 (button) + Task 8 (content div), gated Task 6 (`applyConfigToTabs`) ✓
- `renderVariantBinder()` — defined Task 8, wired Task 8 ✓
