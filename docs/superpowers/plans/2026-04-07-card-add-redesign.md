# Card Add & Pack Opening Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the floating action button (FAB) with header toolbar icons — an inline search bar for quick-adding cards and a dedicated modal for booster pack opening.

**Architecture:** All changes are in `index.html`. The FAB HTML/CSS/JS is removed entirely. Two new header icons (🔍 and 📦) are added. Search expands inline in the header with autocomplete that opens the existing card modal. Pack opening uses a new standalone modal with its own search, card list, and summary flow. The existing `addCard()`, `removeCard()`, `openModal()`, `closeModal()`, and celebration functions are preserved and extended.

**Tech Stack:** Plain HTML/CSS/JS (no build step, no dependencies)

---

### Task 1: Add Search and Pack Icons to Header

**Files:**
- Modify: `index.html:2938-2944` (header-actions HTML)
- Modify: `index.html:120-124` (header-actions CSS)

- [ ] **Step 1: Add the two new icon buttons to the header HTML**

Find the `header-actions` div at line 2938 and add search and pack buttons before the existing theme and sync buttons:

```html
<!-- old -->
<div class="header-actions" id="header-actions">
  <button class="fab-util-btn" id="theme-toggle" title="Toggle theme"></button>
  <button class="fab-util-btn" id="sync-btn" title="Cloud Sync">
```

```html
<!-- new -->
<div class="header-actions" id="header-actions">
  <button class="fab-util-btn" id="header-search-btn" title="Search Cards">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
  </button>
  <button class="fab-util-btn" id="header-pack-btn" title="Open Booster Pack">
    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
  </button>
  <button class="fab-util-btn" id="theme-toggle" title="Toggle theme"></button>
  <button class="fab-util-btn" id="sync-btn" title="Cloud Sync">
```

- [ ] **Step 2: Verify the icons render correctly**

Open `index.html` in a browser. Confirm the search (magnifying glass) and pack (box) icons appear in the header alongside the existing theme and sync buttons.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add search and pack icons to header toolbar"
```

---

### Task 2: Add Header Search Bar HTML and Expand/Collapse CSS

**Files:**
- Modify: `index.html` — add search bar HTML inside `<header>` after `header-actions`
- Modify: `index.html` — add CSS for `.header-search-bar` and expand/collapse transitions

- [ ] **Step 1: Add the expandable search bar HTML**

Add this new div inside `<header>`, right after the closing `</div>` of `.header-actions` (after line 2944):

```html
<!-- Inline header search -->
<div class="header-search-bar" id="header-search-bar">
  <svg class="header-search-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
  <input type="text" id="header-search-input" placeholder="Search cards by name or #..." autocomplete="off">
  <button class="header-search-esc" id="header-search-esc">ESC</button>
</div>
```

- [ ] **Step 2: Add the autocomplete container HTML**

Add the autocomplete dropdown right after the `header-search-bar` div, still inside `<header>`:

```html
<div class="header-autocomplete" id="header-autocomplete"></div>
```

- [ ] **Step 3: Add CSS for the search bar and autocomplete**

Add this CSS in the `<style>` block, after the existing header CSS (after line ~124):

```css
/* ── Header Inline Search ── */
.header-search-bar {
  display: none; /* hidden by default */
  position: absolute; top: 0; left: 0; right: 0; bottom: 0;
  align-items: center; gap: 8px;
  padding: 0 16px;
  background: var(--bg);
  z-index: 50;
  animation: searchBarIn 0.2s ease;
}
@keyframes searchBarIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}
.header-search-bar.open {
  display: flex;
}
.header-search-icon {
  flex-shrink: 0; color: var(--text-muted);
}
.header-search-bar input {
  flex: 1; padding: 10px 12px; font-size: 0.95rem;
  background: var(--slot-bg); border: 2px solid var(--accent);
  border-radius: 8px; color: var(--text); outline: none;
  font-family: 'Rajdhani', sans-serif;
}
.header-search-bar input::placeholder { color: var(--text-muted); }
.header-search-esc {
  background: var(--surface2); border: none; border-radius: 6px;
  color: var(--text-muted); font-size: 0.75rem; font-weight: 700;
  padding: 5px 10px; cursor: pointer; flex-shrink: 0;
  font-family: 'Rajdhani', sans-serif;
}
.header-search-esc:hover { color: var(--text); }

/* Header autocomplete dropdown */
.header-autocomplete {
  display: none;
  position: absolute; top: 100%; left: 0; right: 0;
  background: var(--surface); border: 1px solid var(--surface2);
  border-top: none; border-radius: 0 0 12px 12px;
  max-height: 400px; overflow-y: auto;
  z-index: 49;
  box-shadow: 0 8px 24px var(--shadow);
}
.header-autocomplete.show { display: block; }

/* Autocomplete result rows — reuse .ac-card-row styles already defined */
```

- [ ] **Step 4: Verify the search bar is hidden by default**

Open `index.html` in a browser. The header should look normal — search bar is hidden, icons visible.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add header search bar HTML and CSS"
```

---

### Task 3: Wire Header Search Expand/Collapse JS

**Files:**
- Modify: `index.html` — add JS near the existing FAB section (will be placed after FAB code, which gets removed later)

- [ ] **Step 1: Add the search expand/collapse logic**

Add this JS in the script section, near the top of the FAB section (around line 4938). This code can coexist with the FAB code temporarily:

```javascript
// ── Header Search ──
const headerSearchBtn = document.getElementById('header-search-btn');
const headerSearchBar = document.getElementById('header-search-bar');
const headerSearchInput = document.getElementById('header-search-input');
const headerSearchEsc = document.getElementById('header-search-esc');
const headerAcList = document.getElementById('header-autocomplete');
let headerAcHighlight = -1;

function openHeaderSearch() {
  headerSearchBar.classList.add('open');
  setTimeout(() => headerSearchInput.focus(), 50);
}

function closeHeaderSearch() {
  headerSearchBar.classList.remove('open');
  headerSearchInput.value = '';
  headerAcList.innerHTML = '';
  headerAcList.classList.remove('show');
  headerAcHighlight = -1;
}

headerSearchBtn.addEventListener('click', openHeaderSearch);
headerSearchEsc.addEventListener('click', closeHeaderSearch);

document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && headerSearchBar.classList.contains('open')) {
    closeHeaderSearch();
  }
});

// Close search when clicking outside
document.addEventListener('click', (e) => {
  if (headerSearchBar.classList.contains('open') &&
      !e.target.closest('.header-search-bar') &&
      !e.target.closest('.header-autocomplete') &&
      !e.target.closest('#header-search-btn')) {
    closeHeaderSearch();
  }
});
```

- [ ] **Step 2: Verify expand/collapse works**

Open browser, click the search icon. The search bar should expand over the header. Click ESC or press Escape to collapse. Clicking outside should also close it.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: wire header search expand/collapse behavior"
```

---

### Task 4: Wire Header Search Autocomplete

**Files:**
- Modify: `index.html` — add autocomplete logic to the header search JS section

- [ ] **Step 1: Add autocomplete input handler**

Add this code right after the `closeHeaderSearch` event listeners from Task 3:

```javascript
// ── Header Search Autocomplete ──
headerSearchInput.addEventListener('input', () => {
  const q = headerSearchInput.value.trim().toLowerCase();
  if (q.length < 1) { headerAcList.classList.remove('show'); headerAcList.innerHTML = ''; return; }

  const allSearchable = [...allCards, ...variantCards];
  const matches = allSearchable.filter(c =>
    c.name.toLowerCase().includes(q) || c.collector_number === q
  ).slice(0, 15);

  headerAcHighlight = -1;
  if (matches.length === 0) { headerAcList.classList.remove('show'); headerAcList.innerHTML = ''; return; }

  headerAcList.innerHTML = matches.map((c, i) => {
    const isOwned = ownedSet.has(c.collector_number);
    const isFoilOwned = hasFoil(c.collector_number);
    const vInfo = getCardVariantInfo(c);
    const variantBadge = vInfo.label
      ? `<span class="ac-variant-badge ${vInfo.cssClass}">${vInfo.label}</span>` : '';
    const ownedBadge = isOwned
      ? `<span class="ac-owned-badge" style="font-size:0.65rem">\u2713 Owned</span>`
      : '';
    const foilBadge = isFoilOwned
      ? `<span style="color:var(--gold);font-size:0.65rem;font-weight:700">\u2726 Foil</span>` : '';
    return `<div class="ac-card-row" data-cn="${esc(c.collector_number)}" data-index="${i}">
      <img src="${esc(c.image_small)}" alt="${displayNameText(c)}" loading="lazy"${isOwned ? ' style="border:2px solid var(--owned-border);border-radius:6px"' : ''}>
      <div class="ac-info">
        <div class="ac-name">${displayName(c)} ${variantBadge}</div>
        <div class="ac-detail">#${esc(c.collector_number)} \u00B7 ${esc(c.rarity)} \u00B7 ${esc(c.type_line || '')}</div>
        <div class="ac-prices">
          ${c.price_usd > 0 ? `<span style="color:var(--owned-border)">$${c.price_usd.toFixed(2)}</span>` : ''}
          ${c.price_usd_foil > 0 ? `<span style="color:var(--gold)">\u2726 $${c.price_usd_foil.toFixed(2)}</span>` : ''}
        </div>
      </div>
      <div class="ac-actions">${ownedBadge}${foilBadge}<span style="color:var(--text-muted);font-size:1rem">\u203A</span></div>
    </div>`;
  }).join('');
  headerAcList.classList.add('show');
});
```

- [ ] **Step 2: Add keyboard navigation**

Add right after the input handler:

```javascript
headerSearchInput.addEventListener('keydown', (e) => {
  const items = headerAcList.querySelectorAll('.ac-card-row');
  if (!items.length) return;
  if (e.key === 'ArrowDown') {
    e.preventDefault();
    headerAcHighlight = Math.min(headerAcHighlight + 1, items.length - 1);
    updateHeaderAcHighlight(items);
  } else if (e.key === 'ArrowUp') {
    e.preventDefault();
    headerAcHighlight = Math.max(headerAcHighlight - 1, 0);
    updateHeaderAcHighlight(items);
  } else if (e.key === 'Enter' && headerAcHighlight >= 0) {
    e.preventDefault();
    items[headerAcHighlight].click();
  }
});

function updateHeaderAcHighlight(items) {
  items.forEach((el, i) => el.classList.toggle('hl', i === headerAcHighlight));
  if (items[headerAcHighlight]) items[headerAcHighlight].scrollIntoView({ block: 'nearest' });
}
```

- [ ] **Step 3: Add click handler to open card modal**

Add right after the keyboard navigation:

```javascript
headerAcList.addEventListener('click', (e) => {
  const row = e.target.closest('.ac-card-row');
  if (!row) return;
  const cn = row.dataset.cn;
  const card = getCardByNumber(cn);
  headerAcList.classList.remove('show');
  if (card) openModal(card);
});
```

- [ ] **Step 4: Preserve search after modal close**

Modify `closeModal()` (at line ~6953) to not close the header search. The current `closeModal` already doesn't touch the header search, so this should work automatically. Verify by:
1. Opening search, selecting a card (modal opens)
2. Adding it and closing modal
3. Search bar should still be open with the previous query

If the query gets cleared, add this to `closeModal()` to reopen search:
```javascript
// After overlay animation (inside the setTimeout at line ~6963):
if (headerSearchBar.classList.contains('open')) {
  headerSearchInput.focus();
}
```

- [ ] **Step 5: Verify full search flow**

1. Click search icon → bar expands
2. Type "cloud" → results appear with thumbnails, names, rarity, prices
3. Arrow down/up to highlight → Enter to select
4. Modal opens → add card → close modal → search still open
5. ESC to close search

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: wire header search autocomplete with modal integration"
```

---

### Task 5: Add Pack Modal HTML and CSS

**Files:**
- Modify: `index.html` — add pack modal HTML (near the existing modal overlay, around line 3107)
- Modify: `index.html` — add CSS for `.pack-modal-overlay`

- [ ] **Step 1: Add the pack modal HTML**

Add this right before the existing `<!-- Toast container -->` comment (around line 3073, where the FAB HTML currently ends — it will be placed after the FAB which gets removed later):

```html
<!-- Pack Opening Modal -->
<div class="pack-modal-overlay" id="pack-modal-overlay">
  <div class="pack-modal" id="pack-modal">
    <div class="pack-modal-header">
      <div class="pack-modal-title" id="pack-modal-title">📦 Open Booster Pack</div>
      <div style="display:flex;align-items:center;gap:8px;">
        <span class="pack-modal-price" id="pack-modal-price"></span>
        <button class="pack-modal-close" id="pack-modal-close">&times;</button>
      </div>
    </div>
    <div class="pack-modal-body" id="pack-modal-body"></div>
    <div class="pack-modal-footer" id="pack-modal-footer"></div>
  </div>
</div>
```

- [ ] **Step 2: Add CSS for the pack modal**

Add this CSS after the existing pack mode styles (after line ~2556):

```css
/* ── Pack Opening Modal ── */
.pack-modal-overlay {
  display: none; position: fixed; inset: 0; z-index: 850;
  background: rgba(0,0,0,0.6); backdrop-filter: blur(4px);
  justify-content: center; align-items: center;
  padding: 20px;
}
.pack-modal-overlay.visible {
  display: flex;
  animation: fadeIn 0.2s ease;
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
.pack-modal-overlay.dimmed {
  /* When card modal is stacked on top */
  z-index: 840;
}
.pack-modal-overlay.dimmed .pack-modal {
  opacity: 0.2; transform: scale(0.95);
  pointer-events: none;
}
.pack-modal {
  background: var(--surface); border-radius: 16px;
  width: 100%; max-width: 400px; max-height: 80vh;
  display: flex; flex-direction: column;
  box-shadow: 0 16px 48px var(--shadow);
  transition: opacity 0.3s, transform 0.3s;
  animation: modalSlideUp 0.3s ease;
}
@keyframes modalSlideUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
.pack-modal-header {
  padding: 16px 20px; display: flex;
  justify-content: space-between; align-items: center;
  border-bottom: 1px solid var(--surface2);
  flex-shrink: 0;
}
.pack-modal-title {
  font-size: 1.05rem; font-weight: 700; color: var(--text);
}
.pack-modal-price {
  font-size: 0.8rem; color: var(--text-muted);
}
.pack-modal-close {
  background: none; border: none; color: var(--text-muted);
  font-size: 1.4rem; cursor: pointer; padding: 4px 8px;
  line-height: 1;
}
.pack-modal-close:hover { color: var(--text); }
.pack-modal-body {
  flex: 1; overflow-y: auto; min-height: 0;
}
.pack-modal-footer {
  padding: 12px 16px; border-top: 1px solid var(--surface2);
  display: flex; gap: 8px; flex-shrink: 0;
}
.pack-modal-footer:empty { display: none; }
```

- [ ] **Step 3: Add mobile responsive CSS for the pack modal**

Add within the existing `@media (max-width: 700px)` block:

```css
.pack-modal-overlay { padding: 0; align-items: flex-end; }
.pack-modal {
  max-width: 100%; max-height: 90vh;
  border-radius: 20px 20px 0 0;
  animation: modalSlideUp 0.3s ease;
}
```

- [ ] **Step 4: Verify the modal HTML is in the DOM (hidden)**

Open browser. The pack modal overlay should be invisible (display: none). Inspect the DOM to confirm `#pack-modal-overlay` exists.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add pack opening modal HTML and CSS"
```

---

### Task 6: Wire Pack Modal Step 1 (Price Entry)

**Files:**
- Modify: `index.html` — add JS for pack modal open/close and step 1 rendering

- [ ] **Step 1: Add pack modal JS**

Add this JS in the script section, after the header search code from Task 3-4:

```javascript
// ── Pack Opening Modal ──
const packModalOverlay = document.getElementById('pack-modal-overlay');
const packModalBody = document.getElementById('pack-modal-body');
const packModalFooter = document.getElementById('pack-modal-footer');
const packModalTitle = document.getElementById('pack-modal-title');
const packModalPrice = document.getElementById('pack-modal-price');
const packModalClose = document.getElementById('pack-modal-close');

function openPackModal() {
  currentPack = null;
  renderPackModalStep1();
  packModalOverlay.classList.add('visible');
  packModalOverlay.classList.remove('dimmed');
  document.body.style.overflow = 'hidden';
}

function closePackModal(force) {
  if (!force && currentPack && currentPack.cards.length > 0) {
    // Confirm discard
    if (!confirm(`Discard this pack? ${currentPack.cards.length} card(s) will be removed from your binder.`)) return;
    // Remove cards added during this pack
    for (const cn of currentPack.cards) {
      ownedSet.delete(cn);
    }
    saveCollection();
    renderProgress();
  }
  currentPack = null;
  packModalOverlay.classList.remove('visible', 'dimmed');
  document.body.style.overflow = '';
  packModalTitle.textContent = '📦 Open Booster Pack';
  packModalPrice.textContent = '';
  packModalBody.innerHTML = '';
  packModalFooter.innerHTML = '';
}

document.getElementById('header-pack-btn').addEventListener('click', openPackModal);
packModalClose.addEventListener('click', () => closePackModal(false));
packModalOverlay.addEventListener('click', (e) => {
  if (e.target === packModalOverlay) closePackModal(false);
});
```

- [ ] **Step 2: Add Step 1 renderer (price entry)**

Add right after the above:

```javascript
function renderPackModalStep1() {
  packModalTitle.textContent = '📦 Open Booster Pack';
  packModalPrice.textContent = '';
  packModalFooter.innerHTML = '';

  packModalBody.innerHTML = `
    <div style="padding:24px 20px;text-align:center;">
      <div style="color:var(--text-muted);font-size:0.85rem;margin-bottom:16px;">How much did this pack cost?</div>
      <div class="pack-price-row" style="justify-content:center;">
        <span class="pack-price-label">RM</span>
        <input type="number" class="pack-price-input" id="pack-modal-price-input" placeholder="0.00" step="0.10" min="0" style="max-width:120px;text-align:center;font-size:1.2rem;">
      </div>
      <button class="pack-start-btn" id="pack-modal-start" style="margin-top:4px;">Start Opening</button>
      <div style="color:var(--text-muted);font-size:0.75rem;margin-top:10px;">Price is optional — leave at 0 to skip</div>
    </div>
  `;

  const startBtn = document.getElementById('pack-modal-start');
  const priceInput = document.getElementById('pack-modal-price-input');

  startBtn.addEventListener('click', () => {
    const price = parseFloat(priceInput.value) || 0;
    currentPack = { price, cards: [] };
    renderPackModalStep2();
  });
  priceInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') startBtn.click();
  });
  setTimeout(() => priceInput.focus(), 100);
}
```

- [ ] **Step 3: Verify pack modal opens and shows step 1**

Click the 📦 icon in the header. The pack modal should open with price entry. Enter a price and click "Start Opening" (it won't do anything yet — step 2 is next task).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: wire pack modal open/close and price entry step"
```

---

### Task 7: Wire Pack Modal Step 2 (Card Scanning)

**Files:**
- Modify: `index.html` — add `renderPackModalStep2()` and `renderPackModalCardList()` functions

- [ ] **Step 1: Add Step 2 renderer**

Add right after `renderPackModalStep1()`:

```javascript
function renderPackModalStep2() {
  const packNum = boosterPacks.length + 1;
  packModalTitle.textContent = `📦 Pack #${packNum}`;
  packModalPrice.textContent = currentPack.price > 0 ? `RM ${currentPack.price.toFixed(2)}` : '';

  packModalBody.innerHTML = `
    <div style="padding:12px 16px 8px;position:relative;">
      <input type="text" id="pack-modal-search" placeholder="Search cards to add to pack..." autocomplete="off"
        style="width:100%;padding:10px 14px;font-size:0.9rem;background:var(--slot-bg);border:2px solid var(--surface2);border-radius:8px;color:var(--text);outline:none;font-family:'Rajdhani',sans-serif;">
      <div class="header-autocomplete" id="pack-modal-ac" style="position:absolute;left:16px;right:16px;top:100%;border-radius:0 0 8px 8px;"></div>
    </div>
    <div class="pack-counter">
      <span id="pack-modal-count">${currentPack.cards.length}</span> cards in this pack
      ${currentPack.price > 0 ? ` \u00B7 RM ${currentPack.price.toFixed(2)}` : ''}
    </div>
    <div class="pack-card-list" id="pack-modal-card-list"></div>
  `;

  packModalFooter.innerHTML = `
    <button class="pack-done-btn" id="pack-modal-done" ${currentPack.cards.length === 0 ? 'disabled style="opacity:0.5;cursor:not-allowed;"' : ''}>Done \u2014 Save Pack</button>
    <button class="pack-cancel-btn" id="pack-modal-cancel">Cancel</button>
  `;

  renderPackModalCardList();
  wirePackModalSearch();

  document.getElementById('pack-modal-done').addEventListener('click', () => {
    if (currentPack && currentPack.cards.length > 0) finalizePackModal();
  });
  document.getElementById('pack-modal-cancel').addEventListener('click', () => closePackModal(false));

  setTimeout(() => document.getElementById('pack-modal-search')?.focus(), 100);
}
```

- [ ] **Step 2: Add the card list renderer**

```javascript
function renderPackModalCardList() {
  const container = document.getElementById('pack-modal-card-list');
  if (!container || !currentPack) return;
  if (currentPack.cards.length === 0) {
    container.innerHTML = '<div style="text-align:center;padding:20px 0;color:var(--text-muted);font-size:0.85rem;">Search and add cards from your pack</div>';
    return;
  }
  container.innerHTML = currentPack.cards.map((cn, i) => {
    const card = getCardByNumber(cn);
    if (!card) return '';
    const isFoilOwned = hasFoil(cn);
    return `<div class="pack-card-entry" style="animation-delay:${i * 0.05}s">
      <div class="pack-num">${i + 1}</div>
      <img src="${esc(card.image_small)}" alt="${displayNameText(card)}">
      <span class="pack-card-name">${displayName(card)}</span>
      ${acVariantBadgeHTML(card)}
      ${isFoilOwned ? '<span style="color:var(--foil-color);font-size:0.7rem;font-weight:700">\u2726</span>' : ''}
      <span class="pack-card-rarity ${esc(card.rarity)}">${esc(card.rarity)}</span>
    </div>`;
  }).join('');
  container.scrollTop = container.scrollHeight;

  // Update done button state
  const doneBtn = document.getElementById('pack-modal-done');
  if (doneBtn) {
    doneBtn.disabled = currentPack.cards.length === 0;
    doneBtn.style.opacity = currentPack.cards.length === 0 ? '0.5' : '';
    doneBtn.style.cursor = currentPack.cards.length === 0 ? 'not-allowed' : '';
  }
}
```

- [ ] **Step 3: Verify step 2 layout renders**

Open pack modal, enter price, click Start Opening. Should see search input, "0 cards in this pack", empty card list, Done (disabled) and Cancel buttons.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add pack modal step 2 layout and card list rendering"
```

---

### Task 8: Wire Pack Modal Search and Card Modal Stacking

**Files:**
- Modify: `index.html` — add `wirePackModalSearch()` and `addCardToPackModal()` functions
- Modify: `index.html:6953-6969` — update `closeModal()` to handle pack modal stacking

- [ ] **Step 1: Add the pack modal search wiring**

```javascript
function wirePackModalSearch() {
  const searchInput = document.getElementById('pack-modal-search');
  const acList = document.getElementById('pack-modal-ac');
  let acHl = -1;

  searchInput.addEventListener('input', () => {
    const q = searchInput.value.trim().toLowerCase();
    if (q.length < 1) { acList.classList.remove('show'); acList.innerHTML = ''; return; }
    const allSearchable = [...allCards, ...variantCards];
    const matches = allSearchable.filter(c =>
      c.name.toLowerCase().includes(q) || c.collector_number === q
    ).slice(0, 15);
    acHl = -1;
    if (matches.length === 0) { acList.classList.remove('show'); acList.innerHTML = ''; return; }
    acList.innerHTML = matches.map((c, i) => {
      const isOwned = ownedSet.has(c.collector_number);
      const isFoilOwned = hasFoil(c.collector_number);
      const inPack = currentPack.cards.includes(c.collector_number);
      return `<div class="ac-item" data-cn="${esc(c.collector_number)}" style="display:flex;align-items:center;justify-content:space-between;gap:8px">
        <span style="display:flex;align-items:center;gap:6px;min-width:0;flex:1">
          <span class="ac-rarity-dot ${esc(c.rarity)}"></span>
          <span class="card-name" style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${esc(c.name)}</span>
          <span class="card-num" style="flex-shrink:0">#${esc(c.collector_number)}</span>
        </span>
        <span style="display:flex;align-items:center;gap:4px;flex-shrink:0">
          ${acVariantBadgeHTML(c)}
          ${inPack ? '<span class="owned-tag">in pack</span>' : (isOwned || isFoilOwned) ? '<span class="owned-tag">' + (isFoilOwned ? '\u2726 ' : '') + 'owned</span>' : ''}
        </span>
      </div>`;
    }).join('');
    acList.classList.add('show');
  });

  searchInput.addEventListener('keydown', (e) => {
    const items = acList.querySelectorAll('.ac-item');
    if (!items.length) return;
    if (e.key === 'ArrowDown') { e.preventDefault(); acHl = Math.min(acHl + 1, items.length - 1); }
    else if (e.key === 'ArrowUp') { e.preventDefault(); acHl = Math.max(acHl - 1, 0); }
    else if (e.key === 'Enter' && acHl >= 0) { e.preventDefault(); items[acHl].click(); }
    items.forEach((el, i) => el.classList.toggle('hl', i === acHl));
  });

  acList.addEventListener('click', (e) => {
    const item = e.target.closest('.ac-item');
    if (!item) return;
    const cn = item.dataset.cn;
    const card = getCardByNumber(cn);
    searchInput.value = '';
    acList.classList.remove('show');
    if (card) {
      _packModeAdding = true;
      // Dim the pack modal, open card modal on top
      packModalOverlay.classList.add('dimmed');
      openModal(card);
    }
  });
}
```

- [ ] **Step 2: Add `addCardToPackModal()` function**

```javascript
function addCardToPackModal(cn, asFoil) {
  if (!currentPack) return;
  currentPack.cards.push(cn);
  addCard(cn);
  if (asFoil) addFoil(cn);
  renderPackModalCardList();
  // Bump counter animation
  const counter = document.getElementById('pack-modal-count');
  if (counter) {
    counter.textContent = currentPack.cards.length;
    counter.classList.remove('bump');
    void counter.offsetWidth;
    counter.classList.add('bump');
  }
}
```

- [ ] **Step 3: Update `closeModal()` for pack modal stacking**

In the existing `closeModal()` function (line ~6953), replace the FAB-related logic. Change the `setTimeout` callback:

Find:
```javascript
      if (document.getElementById('tab-binder').classList.contains('active')) renderSpread();
      // Return to FAB pack view if we were adding from pack mode
      if (wasPackMode && currentPack) openFab();
```

Replace with:
```javascript
      if (document.getElementById('tab-binder').classList.contains('active')) renderSpread();
      // Return to pack modal if we were adding from pack mode
      if (wasPackMode && currentPack) {
        packModalOverlay.classList.remove('dimmed');
        renderPackModalStep2();
      }
      // Re-focus header search if it was open
      if (headerSearchBar.classList.contains('open')) {
        headerSearchInput.focus();
      }
```

- [ ] **Step 4: Add "Also adds to your binder" helper text in pack mode**

In `openModal()`, after the toggle button is set to "Add to Pack" (around line 6849), add helper text below the button row. Find the toggle button setup for pack mode and add after the `toggleBtn.onclick` assignment:

```javascript
// Show helper text in pack mode
const modalBtnRow = toggleBtn.parentElement;
if (modalBtnRow && isPack && !modalBtnRow.querySelector('.pack-helper-text')) {
  const helper = document.createElement('div');
  helper.className = 'pack-helper-text';
  helper.style.cssText = 'font-size:0.72rem;color:var(--text-muted);text-align:center;margin-top:6px;';
  helper.textContent = 'Also adds to your binder';
  modalBtnRow.appendChild(helper);
}
```

- [ ] **Step 5: Update `openModal()` pack mode button logic**

In the existing `openModal()` function, the `addToPackAndReturn` function (line ~6815) currently calls `openFab()` and `renderPackStep2()`. Replace it:

Find the `addToPackAndReturn` function body (lines 6815-6833) and replace with:

```javascript
    function addToPackAndReturn(cn, asFoil) {
      addCardToPackModal(cn, asFoil);
      _packModeAdding = false;
      const addedCard = getCardByNumber(cn);
      const cardName = addedCard ? addedCard.name : '';
      const allSearchable = [...allCards, ...variantCards];
      const sameNameCount = cardName ? allSearchable.filter(c => c.name === cardName).length : 0;
      closeModal();
      // If multiple variants share this name, restore search so user can pick another
      setTimeout(() => {
        if (sameNameCount > 1) {
          const psi = document.getElementById('pack-modal-search');
          if (psi) { psi.value = cardName; psi.dispatchEvent(new Event('input')); psi.focus(); }
        }
      }, 350);
    }
```

- [ ] **Step 5: Verify the full pack-add flow**

1. Click 📦 → enter price → Start Opening
2. Search for a card → click result
3. Pack modal dims, card modal opens on top
4. Click "Add to Pack" → card modal closes, pack modal un-dims
5. Card appears in pack list, counter updates
6. Repeat for more cards

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: wire pack modal search with card modal stacking"
```

---

### Task 9: Wire Pack Finalize and Summary

**Files:**
- Modify: `index.html` — add `finalizePackModal()` function

- [ ] **Step 1: Add the finalize function**

```javascript
function finalizePackModal() {
  if (!currentPack || currentPack.cards.length === 0) return;

  const pack = {
    id: Date.now(),
    date: new Date().toISOString(),
    price: currentPack.price,
    cards: [...currentPack.cards],
  };
  boosterPacks.push(pack);
  savePacks();

  // Find best rarity in the pack for celebration
  const rarityOrder = { mythic: 4, rare: 3, uncommon: 2, common: 1 };
  let bestRarity = 'common';
  for (const cn of pack.cards) {
    const card = getCardByNumber(cn);
    if (card && (rarityOrder[card.rarity] || 0) > (rarityOrder[bestRarity] || 0)) {
      bestRarity = card.rarity;
    }
  }

  // Close the pack modal cleanly (force=true to skip confirm)
  currentPack = null;
  packModalOverlay.classList.remove('visible', 'dimmed');
  document.body.style.overflow = '';
  packModalBody.innerHTML = '';
  packModalFooter.innerHTML = '';

  // Show pack summary celebration (reuse existing function)
  showPackSummary(pack, bestRarity);

  // Refresh active tabs
  if (document.getElementById('tab-dashboard').classList.contains('active')) renderDashboard();
  if (document.getElementById('tab-portfolio').classList.contains('active')) renderPortfolio();
  if (document.getElementById('tab-timeline').classList.contains('active')) renderTimeline();
}
```

- [ ] **Step 2: Verify pack finalization**

1. Open pack modal, add 2-3 cards
2. Click "Done — Save Pack"
3. Pack modal closes, celebration overlay appears with card images
4. Celebration effects fire based on best rarity
5. Summary auto-dismisses after 3.5 seconds

- [ ] **Step 3: Verify cancel with undo**

1. Open pack modal, add 2 cards
2. Click Cancel or ✕
3. Confirm dialog appears: "Discard this pack? 2 card(s) will be removed from your binder."
4. Confirm → cards are removed from collection
5. Check Dashboard stats reflect the removal

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: wire pack finalize with celebration and cancel-undo"
```

---

### Task 10: Remove FAB HTML, CSS, and JS

**Files:**
- Modify: `index.html` — delete all FAB-related code

- [ ] **Step 1: Remove FAB HTML**

Delete the entire FAB block (lines 3046-3071):
```html
<!-- Floating Add Cards Button -->
<div class="fab-search" id="fab-search">
  ...
</div>
```

- [ ] **Step 2: Remove FAB CSS**

Delete all CSS from `/* ── FAB (Floating Action Button) ── */` through the end of `.fab-qa-btn.recent:hover` (lines 2223-2433). This includes:
- `.fab-search`, `.fab-btn`, `.fab-panel`, `.fab-panel-header`, `.fab-close`
- `.search-wrap` scoped inside `.fab-panel`
- `.autocomplete` scoped inside `.fab-panel`
- `.ac-item`, `.ac-rarity-dot`, `.ac-foil-toggle` (the `.ac-item` styles used in the old pack autocomplete — check if pack modal still needs them. It does — keep `.ac-item`, `.ac-rarity-dot`, `.owned-tag`, `.ac-variant-badge` styles. Only delete FAB-specific styles.)
- `.fab-recent`, `.fab-recent-item`, `.fab-empty`
- `.ac-card-row`, `.ac-add-btn`, `.ac-foil-btn`, `.ac-owned-badge` (these are reused by header search — **keep them**)
- `.fab-quick-actions`, `.fab-qa-btn`

**Be careful:** Keep these CSS rules that are reused:
- `.ac-card-row` and children (used by header search autocomplete)
- `.ac-item`, `.ac-rarity-dot`, `.owned-tag` (used by pack modal autocomplete)
- `.ac-variant-badge` variants (used everywhere)
- Pack mode styles `.pack-step` through `.pack-summary` (reused by pack modal)

Delete only FAB-specific rules:
- `.fab-search`, `.fab-btn`, `.fab-panel`, `.fab-panel-header`, `.fab-close`
- `.fab-panel .search-wrap`, `.fab-panel .autocomplete` (scoped to fab-panel)
- `.fab-recent`, `.fab-recent-title`, `.fab-recent-item`, `.fab-empty`
- `.fab-quick-actions`, `.fab-qa-btn`

- [ ] **Step 3: Remove FAB JS**

Delete the following JS sections:
1. `fabMode` variable declaration (line 4068): `let fabMode = 'quick';`
2. The FAB element references (lines 4939-4944): `const fabBtn = ...`, `const fabPanel = ...`, `const fabClose = ...`, `const searchInput = ...`, `const acList = ...`
3. `toggleFab()`, `openFab()`, `window.openFab = openFab`, `closeFab()` functions (lines 4946-4965)
4. FAB event listeners (lines 4967-4979): `fabBtn.addEventListener`, `fabClose.addEventListener`, document click-outside, Escape handler for FAB
5. The old FAB search autocomplete (lines 4982-5093): `searchInput.addEventListener('input', ...)`, keyboard nav, click handler with `fabMode` check
6. `renderFabRecent()` function (lines 5096-5134)
7. Booster Pack Mode section (lines 5136-5361): `fabQuickMode`, `fabPackMode`, `packStepContainer`, `fabQuickActions` references, `fab-qa-pack` and `fab-qa-recent` listeners, `renderPackMode()`, `renderPackStep1()`, `startPack()`, `renderPackStep2()`, `addCardToPack()`, `renderPackCardList()`, `finalizePack()`
8. **Keep `showPackSummary()`** (lines 5363-5396) — it's reused by the new `finalizePackModal()`. Do NOT delete it.

- [ ] **Step 4: Update `openFab()` references in Dashboard and Timeline**

Find the two `onclick="openFab()"` references in the dashboard (line ~5483) and timeline (line ~5871). Replace them:

Line ~5483 (Dashboard empty state):
```javascript
// old: onclick="openFab()"
// new: onclick="document.getElementById('header-search-btn').click()"
```

Line ~5871 (Timeline empty state):
```javascript
// old: onclick="openFab()">Add Your First Card</button>
// new: onclick="document.getElementById('header-search-btn').click()">Add Your First Card</button>
```

- [ ] **Step 5: Remove `renderFabRecent()` calls**

Find and remove all remaining calls to `renderFabRecent()`:
- In `addCard()` (line ~4906): delete the `renderFabRecent();` line
- In `removeCard()` (line ~4930): delete the `renderFabRecent();` line
- In `renderAll()` (line ~7346): delete the `renderFabRecent();` line
- In the two data fetch callbacks (lines ~7679 and ~7707): delete `renderFabRecent();`

- [ ] **Step 6: Remove the `window.openFab` assignment**

Delete `window.openFab = openFab;` (was at line 4958, already removed with the function).

- [ ] **Step 7: Verify everything works without FAB**

1. Page loads — no FAB button visible
2. Header search icon works: search, select, modal opens, add card, close modal
3. Pack icon works: price entry, search cards, card modal stacking, done/save
4. Dashboard and Timeline "Add Cards" buttons open header search
5. Pack summary celebration still fires
6. No console errors

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: remove FAB — all card-add and pack flows use header actions"
```

---

### Task 11: Final Polish and Edge Cases

**Files:**
- Modify: `index.html` — handle edge cases

- [ ] **Step 1: Ensure keyboard shortcut for search**

The old FAB had no global keyboard shortcut. Add one for the new search — Ctrl+K or `/` (common patterns):

```javascript
document.addEventListener('keydown', (e) => {
  // Ctrl+K or / to open search (when not already typing)
  if ((e.key === 'k' && (e.ctrlKey || e.metaKey)) || 
      (e.key === '/' && !e.target.closest('input, textarea'))) {
    e.preventDefault();
    if (headerSearchBar.classList.contains('open')) closeHeaderSearch();
    else openHeaderSearch();
  }
});
```

- [ ] **Step 2: Handle pack modal overlay click-outside properly**

Verify that clicking on the dimmed overlay behind a pack modal (when card modal is stacked) doesn't close the pack modal. The `dimmed` class sets `pointer-events: none` on the pack modal itself, but the overlay click handler still fires. Add a guard:

In the pack modal overlay click handler, add:
```javascript
packModalOverlay.addEventListener('click', (e) => {
  if (e.target === packModalOverlay && !packModalOverlay.classList.contains('dimmed')) {
    closePackModal(false);
  }
});
```
(Replace the existing handler from Task 6.)

- [ ] **Step 3: Clean up mobile bottom nav FAB references**

Search for any remaining references to the FAB in the mobile bottom nav or responsive CSS. Check for `.bnav-fab` or similar classes and remove them if present.

- [ ] **Step 4: Full end-to-end test**

Run through these scenarios:
1. **Quick add flow**: Search → select → modal → add → close → search preserved → add another → ESC
2. **Pack flow**: Pack icon → price → start → search → select → card modal → add to pack → repeat × 3 → done → celebration
3. **Pack cancel**: Pack icon → add 2 cards → cancel → confirm discard → cards removed
4. **Keyboard**: Ctrl+K opens search, arrows navigate, Enter selects, ESC closes
5. **Mobile**: All of the above on a narrow viewport
6. **Theme toggle**: Switch light/dark while search or pack modal is open
7. **Cloud sync**: Add cards via new flows, verify sync still triggers

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: polish header search and pack modal edge cases"
```
