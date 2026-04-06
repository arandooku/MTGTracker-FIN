# UI/UX Polish & Full Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign all 6 tabs, FAB search panel, and add visual polish + discoverability across the entire FIN Binder Tracker app.

**Architecture:** Single-file app (`index.html`) with inline CSS and JS. Each task rewrites one render function or adds one CSS/JS system. Tasks are ordered so foundational CSS changes come first, then tab-by-tab rewrites (each independent), then cross-cutting polish.

**Tech Stack:** Plain HTML/CSS/JS, no build step, no dependencies. Scryfall API for card data. GitHub Gist API for sync.

**Spec:** `docs/superpowers/specs/2026-04-06-ui-ux-polish-design.md`

---

## Task Ordering Strategy

1. **Tasks 1-2**: Foundation (CSS variables, typography, spacing, skeleton loading)
2. **Tasks 3-4**: Navigation polish (bottom nav labels, tab transitions)
3. **Tasks 5-10**: Tab rewrites (Dashboard, Portfolio, Timeline, Binder, Collector, Settings) — each independent
4. **Task 11**: FAB Smart Panel redesign
5. **Tasks 12-14**: Cross-cutting polish (modal, card slot polish, discoverability)

Each task is a self-contained commit. Tasks 5-10 can be done in any order.

---

### Task 1: CSS Foundation — Spacing Variables & Typography System

**Files:**
- Modify: `index.html` (CSS section, lines ~22-67 theme variables area)

**Context:** Spec Section 5E. Establish spacing scale and typography hierarchy that all subsequent tasks will use.

- [ ] **Step 1: Add spacing CSS variables to both theme blocks**

In the `[data-theme="dark"]` block (around line 25), add after the existing variables:

```css
/* Spacing scale */
--sp-1: 4px; --sp-2: 8px; --sp-3: 12px; --sp-4: 16px; --sp-6: 24px; --sp-8: 32px;
```

Add the same variables to the `[data-theme="light"]` block (around line 47).

- [ ] **Step 2: Add typography utility classes after the body rule (around line 82)**

```css
/* Typography hierarchy */
.font-display { font-family: 'Cinzel', serif; text-transform: uppercase; letter-spacing: 0.1em; line-height: 1.2; }
.font-ui { font-family: 'Rajdhani', sans-serif; font-weight: 600; text-transform: uppercase; letter-spacing: 0.06em; line-height: 1.3; }
.font-body { font-family: 'Inter', -apple-system, sans-serif; line-height: 1.4; }
.font-num { font-variant-numeric: tabular-nums; line-height: 1.1; }
```

- [ ] **Step 3: Add shared card carousel CSS (used by Dashboard, Portfolio)**

After the toast styles (around line 880), add:

```css
/* ── Card Carousel ── */
.card-carousel {
  display: flex; gap: 8px; overflow-x: auto; padding-bottom: 4px;
  scrollbar-width: none; -webkit-overflow-scrolling: touch;
}
.card-carousel::-webkit-scrollbar { display: none; }
.card-carousel-item {
  flex-shrink: 0; width: 72px; cursor: pointer;
  transition: transform 0.2s;
}
.card-carousel-item:hover { transform: translateY(-2px); }
.card-carousel-item img {
  width: 72px; height: 100px; border-radius: 6px;
  object-fit: cover; margin-bottom: 4px;
}
.card-carousel-item .cc-name {
  font-size: 0.6rem; font-weight: 600; color: var(--text);
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.card-carousel-item .cc-price {
  font-size: 0.58rem; font-weight: 700;
}
.card-carousel-item.unowned img {
  opacity: 0.5; border: 1.5px dashed rgba(46,58,82,0.5);
}
.card-carousel-item.unowned .cc-name { color: var(--text-muted); }
```

- [ ] **Step 4: Add pill chip CSS (used by Binder, Collector)**

```css
/* ── Pill Chips ── */
.pill-chip {
  padding: 6px 14px; border-radius: 20px;
  font-size: 0.72rem; font-weight: 700;
  white-space: nowrap; flex-shrink: 0;
  cursor: pointer; transition: all 0.2s;
  border: 1px solid var(--ff-border);
  background: transparent; color: var(--text-muted);
}
.pill-chip.active { background: var(--accent); color: #fff; border-color: var(--accent); }
.pill-chip:hover:not(.active) { border-color: var(--accent); color: var(--text); }
.pill-scroll {
  display: flex; gap: 6px; overflow-x: auto;
  scrollbar-width: none; -webkit-overflow-scrolling: touch;
  padding-bottom: 4px;
}
.pill-scroll::-webkit-scrollbar { display: none; }
```

- [ ] **Step 5: Add rarity circle CSS (used by Collector)**

```css
/* ── Rarity Circles ── */
.rarity-circle {
  width: 24px; height: 24px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 0.55rem; font-weight: 800; cursor: pointer;
  transition: all 0.2s; border: 2px solid;
  background: transparent;
}
.rarity-circle.active { color: #fff !important; }
.rarity-circle.rc-common { border-color: #888; color: #888; }
.rarity-circle.rc-common.active { background: #888; }
.rarity-circle.rc-uncommon { border-color: #72b5b5; color: #72b5b5; }
.rarity-circle.rc-uncommon.active { background: #72b5b5; }
.rarity-circle.rc-rare { border-color: #c9a14e; color: #c9a14e; }
.rarity-circle.rc-rare.active { background: #c9a14e; }
.rarity-circle.rc-mythic { border-color: #e8702a; color: #e8702a; }
.rarity-circle.rc-mythic.active { background: #e8702a; }
```

- [ ] **Step 6: Add compact stats row CSS (used by Dashboard, Collector)**

```css
/* ── Compact Stats Row ── */
.stats-row {
  display: grid; grid-template-columns: 1fr 1fr 1fr; gap: var(--sp-2);
  margin-bottom: var(--sp-3);
}
.stat-pill {
  background: var(--surface); border-radius: 8px;
  padding: var(--sp-3); text-align: center;
  border: 1px solid var(--ff-border);
}
.stat-pill .stat-num {
  font-size: 1.1rem; font-weight: 800;
}
.stat-pill .stat-label {
  font-size: 0.6rem; color: var(--text-muted);
  text-transform: uppercase; letter-spacing: 0.04em;
}
```

- [ ] **Step 7: Add card name overlay CSS (used by Binder, Collector, Portfolio carousel)**

```css
/* ── Card Name Overlay ── */
.card-name-overlay {
  position: absolute; bottom: 0; left: 0; right: 0;
  padding: 4px 5px;
  background: linear-gradient(transparent, rgba(0,0,0,0.8));
  border-radius: 0 0 5px 5px;
  pointer-events: none;
}
.card-name-overlay .overlay-name {
  font-size: 0.5rem; color: #fff; font-weight: 600;
}
.card-name-overlay .overlay-sub {
  font-size: 0.42rem; text-transform: uppercase;
}
```

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "style: add CSS foundation — spacing vars, typography, shared components"
```

---

### Task 2: Skeleton Loading States

**Files:**
- Modify: `index.html` (CSS section + HTML `#loading-screen` + JS `loadCards` area)

**Context:** Spec Section 5D. Replace crystal spinner with skeleton screens.

- [ ] **Step 1: Add skeleton CSS after the loading styles (around line 2319)**

```css
/* ── Skeleton Loading ── */
.skeleton {
  background: linear-gradient(90deg, var(--surface) 25%, var(--surface2) 50%, var(--surface) 75%);
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.5s ease-in-out infinite;
  border-radius: 6px;
}
@keyframes skeleton-shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
.skeleton-header { height: 80px; margin-bottom: 20px; border-radius: 8px; }
.skeleton-tabs { height: 40px; margin-bottom: 20px; border-radius: 6px; }
.skeleton-grid {
  display: grid; grid-template-columns: 1fr 1fr; gap: 16px;
}
.skeleton-card { height: 120px; border-radius: 6px; }
@media (max-width: 600px) { .skeleton-grid { grid-template-columns: 1fr; } }
.skeleton-fade-out {
  animation: skeleton-out 0.3s ease both;
}
@keyframes skeleton-out {
  to { opacity: 0; }
}
```

- [ ] **Step 2: Replace the loading screen HTML (around line 2520)**

Find the current `#loading-screen` div:
```html
<div id="loading-screen" class="loading">
    <div class="spinner"></div>
    <p>Loading FIN set data from Scryfall...</p>
</div>
```

Replace with:
```html
<div id="loading-screen" class="loading" style="padding:16px;max-width:1200px;margin:0 auto;">
    <div class="skeleton skeleton-header"></div>
    <div class="skeleton skeleton-tabs"></div>
    <div class="skeleton-grid">
      <div class="skeleton skeleton-card"></div>
      <div class="skeleton skeleton-card"></div>
      <div class="skeleton skeleton-card" style="animation-delay:0.1s"></div>
      <div class="skeleton skeleton-card" style="animation-delay:0.1s"></div>
      <div class="skeleton skeleton-card" style="grid-column:1/-1;height:80px;animation-delay:0.2s"></div>
    </div>
    <p style="text-align:center;margin-top:16px;font-size:0.82rem;color:var(--text-muted)">Loading FIN set data...</p>
</div>
```

- [ ] **Step 3: Add crossfade to the main-content reveal in JS**

Find in the JS (around line 6230 area) where `main-content` is shown after data loads. It should be something like:
```js
document.getElementById('loading-screen').style.display = 'none';
document.getElementById('main-content').style.display = '';
```

Replace with:
```js
const loadingEl = document.getElementById('loading-screen');
loadingEl.classList.add('skeleton-fade-out');
setTimeout(() => {
  loadingEl.style.display = 'none';
  document.getElementById('main-content').style.display = '';
}, 300);
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "style: replace crystal spinner with skeleton loading screens"
```

---

### Task 3: Mobile Bottom Nav Polish

**Files:**
- Modify: `index.html` (CSS `.bnav-btn` styles + HTML bottom nav buttons)

**Context:** Spec Section 5F. Add labels, accent border, increase height.

- [ ] **Step 1: Update bottom nav CSS**

Find the `.bnav-btn` styles (around line 357). Replace the entire `.bnav-btn` rule and related rules:

```css
.bnav-btn {
  flex: 1; display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  background: none; border: none; border-top: 2px solid transparent;
  padding: 6px 4px 8px; color: var(--text-muted); cursor: pointer;
  transition: color 0.15s, border-color 0.15s;
  border-radius: 0; min-height: 56px; min-width: 44px;
  gap: 2px;
}
.bnav-btn svg { width: 20px; height: 20px; flex-shrink: 0; }
.bnav-btn .bnav-label {
  font-size: 0.58rem; text-transform: uppercase;
  letter-spacing: 0.04em; font-weight: 600;
}
.bnav-btn:hover { color: var(--text); }
.bnav-btn.active {
  color: var(--text);
  border-top-color: var(--accent);
}
.bnav-btn.active svg { filter: drop-shadow(0 0 6px var(--accent)); }
```

- [ ] **Step 2: Update bottom nav background in the mobile media query**

Find the `.bottom-nav` rule inside `@media (max-width: 700px)` (around line 1404). Update to:

```css
.bottom-nav {
  display: flex;
  position: fixed; bottom: 0; left: 0; right: 0;
  background: var(--ff-panel);
  border-top: 1px solid var(--ff-border);
  box-shadow: 0 -1px 0 var(--ff-border), 0 -8px 24px rgba(0,0,0,0.15);
  padding: 0 8px max(4px, env(safe-area-inset-bottom, 4px));
  z-index: 100;
}
```

- [ ] **Step 3: Add labels to each bottom nav button in HTML**

Find the bottom nav HTML (around line 2557). Add a `<span class="bnav-label">` to each button. For example, the dashboard button becomes:

```html
<button class="bnav-btn active" data-tab="dashboard">
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>
  <span class="bnav-label">Home</span>
</button>
```

Repeat for all 6 buttons with labels: Home, Value, Activity, Binder, Collector, Settings.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "style: polish mobile bottom nav with labels, accent border, larger targets"
```

---

### Task 4: Smooth Tab Transitions + Swipe

**Files:**
- Modify: `index.html` (CSS tab animations + JS `switchTab()` function)

**Context:** Spec Section 5A. Directional slide transitions and mobile swipe.

- [ ] **Step 1: Add directional tab transition CSS**

Find the `@keyframes tab-enter` block (around line 348). Replace it and add new animations:

```css
.tab-content { display: none; }
.tab-content.active { display: block; }
.tab-content.tab-slide-in-right { animation: tab-slide-in-right 0.3s ease both; }
.tab-content.tab-slide-in-left { animation: tab-slide-in-left 0.3s ease both; }
@keyframes tab-slide-in-right {
  from { opacity: 0; transform: translateX(30px); }
  to { opacity: 1; transform: translateX(0); }
}
@keyframes tab-slide-in-left {
  from { opacity: 0; transform: translateX(-30px); }
  to { opacity: 1; transform: translateX(0); }
}
```

- [ ] **Step 2: Update `switchTab()` to track direction and apply directional animation**

Find `function switchTab(tabName)` (line 6108). Replace the entire function:

```js
const TAB_ORDER = ['dashboard','portfolio','timeline','binder','collector','settings'];
let currentTabIndex = 0;

function switchTab(tabName) {
  const newIndex = TAB_ORDER.indexOf(tabName);
  const direction = newIndex >= currentTabIndex ? 'right' : 'left';
  currentTabIndex = newIndex !== -1 ? newIndex : currentTabIndex;

  document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
  document.querySelectorAll('.bnav-btn').forEach(b => b.classList.remove('active'));
  document.querySelectorAll('.tab-content').forEach(t => {
    t.classList.remove('active', 'tab-slide-in-right', 'tab-slide-in-left');
  });
  document.querySelectorAll(`.tab-btn[data-tab="${tabName}"]`).forEach(b => b.classList.add('active'));
  document.querySelectorAll(`.bnav-btn[data-tab="${tabName}"]`).forEach(b => b.classList.add('active'));
  const tabEl = document.getElementById(`tab-${tabName}`);
  if (tabEl) {
    tabEl.classList.add('active');
    if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
      tabEl.classList.add(direction === 'right' ? 'tab-slide-in-right' : 'tab-slide-in-left');
    }
  }
  if (tabName === 'binder') renderBinder();
  if (tabName === 'dashboard') renderDashboard();
  if (tabName === 'portfolio') renderPortfolio();
  if (tabName === 'timeline') renderTimeline();
  if (tabName === 'settings') renderSettings();
  if (tabName === 'collector') renderCollectorTab();
}
```

- [ ] **Step 3: Add mobile swipe gesture detection**

Add this after the bottom nav event listeners (around line 6130):

```js
// ── Mobile Tab Swipe ──
(function initTabSwipe() {
  let touchStartX = 0, touchStartY = 0;
  const app = document.getElementById('app');
  app.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
  }, { passive: true });
  app.addEventListener('touchend', e => {
    const dx = e.changedTouches[0].clientX - touchStartX;
    const dy = e.changedTouches[0].clientY - touchStartY;
    if (Math.abs(dx) < 50 || Math.abs(dy) > Math.abs(dx)) return;
    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
    const visibleTabs = TAB_ORDER.filter(t => {
      const btn = document.querySelector(`.tab-btn[data-tab="${t}"]`);
      return btn && btn.style.display !== 'none';
    });
    const curIdx = visibleTabs.indexOf(TAB_ORDER[currentTabIndex]);
    if (curIdx === -1) return;
    if (dx < -50 && curIdx < visibleTabs.length - 1) switchTab(visibleTabs[curIdx + 1]);
    else if (dx > 50 && curIdx > 0) switchTab(visibleTabs[curIdx - 1]);
  }, { passive: true });
})();
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: directional tab slide transitions + mobile swipe gesture"
```

---

### Task 5: Dashboard Redesign — Hero Dashboard

**Files:**
- Modify: `index.html` (JS `renderDashboard()` function, ~line 4652)

**Context:** Spec Section 7. Hero progress card, quick stats, session strip, binder map.

- [ ] **Step 1: Rewrite `renderDashboard()`**

Find `function renderDashboard()` (line 4652). Replace the entire function body (everything from the opening `{` to the closing `}` before the comment `// ── Rendering: Portfolio ──`) with the new hero dashboard layout. The function should:

1. Calculate the same data as before: `total`, `owned`, `pct`, `rarityCounts`, `totalBinderPages`, `fullPages`, `recentCNs`
2. Build the hero progress ring SVG inline (80x80px donut with gradient stroke)
3. Build the rarity stacked bar (4 proportional segments)
4. Build the 3-pill stats row using `.stats-row` and `.stat-pill` classes
5. Build the session card strip using `.card-carousel` class
6. Build the binder mini-map grid

The function is large — write the complete implementation. Key HTML structure:

```js
function renderDashboard() {
  const dash = document.getElementById('dashboard');
  const total = allCards.length;
  const owned = ownedSet.size;
  const missing = total - owned;
  const pct = total ? (owned / total * 100) : 0;
  const rarities = ['mythic', 'rare', 'uncommon', 'common'];
  const rarityColors = { mythic: '#e8702a', rare: '#c9a14e', uncommon: '#72b5b5', common: '#777' };
  const rarityCounts = {};
  for (const r of rarities) {
    rarityCounts[r] = {
      total: allCards.filter(c => c.rarity === r).length,
      owned: allCards.filter(c => c.rarity === r && ownedSet.has(c.collector_number)).length,
    };
  }
  const foilCount = Object.keys(foilOwned).length;
  const packCount = boosterPacks.length;
  const collValue = allCards.filter(c => ownedSet.has(c.collector_number)).reduce((s, c) => s + c.price_usd, 0);
  const recentCNs = packSession.slice(-10).reverse();

  // Binder map
  const totalBinderPages = Math.ceil(total / binderConfig.slotsPerPage);
  const pageStatus = [...Array(totalBinderPages)].map((_, i) => {
    const pageCards = allCards.slice(i * binderConfig.slotsPerPage, (i + 1) * binderConfig.slotsPerPage);
    if (pageCards.length === 0) return 'empty';
    const ownedOnPage = pageCards.filter(c => ownedSet.has(c.collector_number)).length;
    if (ownedOnPage === pageCards.length) return 'complete';
    if (ownedOnPage > 0) return 'partial';
    return 'empty';
  });
  const completePages = pageStatus.filter(s => s === 'complete').length;

  // Progress ring SVG
  const circumference = 2 * Math.PI * 32;
  const dashLen = (pct / 100) * circumference;
  const ringSVG = `<svg width="80" height="80" style="display:block;flex-shrink:0">
    <defs><linearGradient id="heroRingGrad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="var(--accent2)"/><stop offset="100%" stop-color="var(--accent)"/>
    </linearGradient></defs>
    <circle cx="40" cy="40" r="32" fill="none" stroke="var(--chart-empty)" stroke-width="6"/>
    <circle cx="40" cy="40" r="32" fill="none" stroke="url(#heroRingGrad)" stroke-width="6"
      stroke-dasharray="${dashLen.toFixed(1)} ${circumference.toFixed(1)}"
      stroke-dashoffset="${(circumference * 0.25).toFixed(1)}"
      stroke-linecap="round" transform="rotate(-90 40 40)"
      style="transition:stroke-dasharray 0.5s ease"/>
    <text x="40" y="36" text-anchor="middle" dominant-baseline="central"
      style="font-size:16px;font-weight:900;fill:var(--gold)">${pct.toFixed(0)}%</text>
    <text x="40" y="51" text-anchor="middle" style="font-size:7px;fill:var(--text-muted)">complete</text>
  </svg>`;

  // Rarity stacked bar
  const totalOwned = owned || 1;
  const rarityBar = rarities.map(r => {
    const w = (rarityCounts[r].owned / totalOwned) * 100;
    return w > 0 ? `<div style="flex:${w};height:6px;background:${rarityColors[r]};"></div>` : '';
  }).join('');
  const rarityLabels = rarities.map(r =>
    `<span style="font-size:0.58rem;color:${rarityColors[r]};">${r[0].toUpperCase()} ${rarityCounts[r].owned}/${rarityCounts[r].total}</span>`
  ).join(' ');

  // Session card strip
  const sessionStrip = recentCNs.length > 0
    ? `<div class="card-carousel">${recentCNs.map(cn => {
        const card = getCardByNumber(cn);
        if (!card) return '';
        const borderColor = rarityColors[card.rarity] || '#888';
        return `<div class="card-carousel-item" data-cn="${esc(cn)}">
          <img src="${esc(card.image_small)}" alt="${displayNameText(card)}"
            style="border:1.5px solid ${borderColor};${card.rarity === 'mythic' || card.rarity === 'rare' ? 'box-shadow:0 0 8px ' + borderColor + '30;' : ''}" loading="lazy">
          <div class="cc-name">${displayName(card)}</div>
        </div>`;
      }).join('')}</div>`
    : `<div style="text-align:center;padding:16px 0;color:var(--text-muted);font-size:0.82rem">
        Open a pack or search for cards to get started
        <br><button class="btn btn-primary" style="margin-top:8px;font-size:0.78rem;padding:6px 14px" onclick="openFab()">Add Cards</button>
      </div>`;

  // Binder mini-map
  const mapCells = pageStatus.map((s, i) => {
    const bg = s === 'complete' ? 'var(--owned-border)' : s === 'partial' ? 'rgba(91,127,199,0.4)' : 'var(--chart-empty)';
    return `<div style="width:14px;height:10px;border-radius:2px;background:${bg}" title="Page ${i + 1}: ${s}"></div>`;
  }).join('');

  dash.innerHTML = `
    <div class="dash-card" style="margin-bottom:var(--sp-3)">
      <div style="display:flex;align-items:center;gap:var(--sp-4);position:relative">
        <div style="position:absolute;top:-20px;right:-20px;width:120px;height:120px;background:radial-gradient(circle,rgba(212,168,80,0.08),transparent 70%);pointer-events:none"></div>
        ${ringSVG}
        <div style="flex:1">
          <div style="font-size:clamp(1.4rem,4vw,1.8rem);font-weight:900;color:var(--text)">${owned} <span style="font-size:0.9rem;color:var(--text-muted);font-weight:500">/ ${total}</span></div>
          <div style="font-size:0.78rem;color:var(--text-muted);margin-bottom:var(--sp-2)">${missing} cards remaining</div>
          <div style="display:flex;gap:0;border-radius:3px;overflow:hidden">${rarityBar}</div>
          <div style="display:flex;gap:var(--sp-2);margin-top:4px;flex-wrap:wrap">${rarityLabels}</div>
        </div>
      </div>
    </div>

    <div class="stats-row">
      <div class="stat-pill"><div class="stat-num" style="color:var(--gold)">✦ ${foilCount}</div><div class="stat-label">Foils</div></div>
      <div class="stat-pill"><div class="stat-num" style="color:var(--accent)">${packCount}</div><div class="stat-label">Packs</div></div>
      <div class="stat-pill"><div class="stat-num" style="color:var(--owned-border)">$${collValue.toFixed(0)}</div><div class="stat-label">Value</div></div>
    </div>

    <div class="dash-card" style="margin-bottom:var(--sp-3)">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:var(--sp-2)">
        <h3 style="margin:0">This Session</h3>
        <span style="font-size:0.68rem;color:var(--accent);font-weight:600">${recentCNs.length} cards added</span>
      </div>
      ${sessionStrip}
    </div>

    <div class="dash-card">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:var(--sp-2)">
        <h3 style="margin:0">Binder Map</h3>
        <span style="font-size:0.62rem;color:var(--text-muted)">${completePages} / ${totalBinderPages} pages complete</span>
      </div>
      <div style="display:flex;gap:2px;flex-wrap:wrap">${mapCells}</div>
      <div style="display:flex;gap:var(--sp-3);margin-top:6px">
        <div style="display:flex;align-items:center;gap:4px;font-size:0.55rem;color:var(--text-muted)"><div style="width:8px;height:6px;border-radius:1px;background:var(--owned-border)"></div>Complete</div>
        <div style="display:flex;align-items:center;gap:4px;font-size:0.55rem;color:var(--text-muted)"><div style="width:8px;height:6px;border-radius:1px;background:rgba(91,127,199,0.4)"></div>Partial</div>
        <div style="display:flex;align-items:center;gap:4px;font-size:0.55rem;color:var(--text-muted)"><div style="width:8px;height:6px;border-radius:1px;background:var(--chart-empty)"></div>Empty</div>
      </div>
    </div>
  `;

  // Wire session card clicks
  dash.querySelectorAll('.card-carousel-item[data-cn]').forEach(item => {
    item.addEventListener('click', () => {
      const card = getCardByNumber(item.dataset.cn);
      if (card) openModal(card);
    });
  });
}
```

- [ ] **Step 2: Verify by opening index.html and switching to Dashboard**

Expected: Hero ring at top with rarity bar, 3 stat pills, session card strip, binder mini-map at bottom.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: redesign Dashboard — hero progress ring, stat pills, card strip, binder map"
```

---

### Task 6: Portfolio Redesign — Wealth Showcase

**Files:**
- Modify: `index.html` (JS `renderPortfolio()` function, ~line 4825)

**Context:** Spec Section 8. Hero valuation, rarity value bar, card carousels, growth chart.

- [ ] **Step 1: Rewrite `renderPortfolio()`**

Replace the entire `renderPortfolio()` function. The new layout:

1. Hero valuation card (USD + MYR, one-line price sources, rarity value bar)
2. ROI + Still Need row (2-column grid)
3. Most Valuable Owned card carousel
4. Most Wanted (Missing) card carousel
5. Foil + Variant summary row (2-column grid)
6. Growth chart (SVG, moved from Timeline)

Key data calculations stay the same. Use `.card-carousel` and `.card-carousel-item` for the scrollable card rows. Each carousel item is clickable → `openModal(card)`.

The growth chart reuses the existing `dailyCounts`/`chartDays` logic from `renderTimeline()` but plots cumulative USD value instead of card count. Extract the chart data:

```js
// Growth chart data — cumulative value by date
const addEvents = timelineEvents.filter(e => e.type === 'add');
const dailyValues = {};
const seenForChart = new Set();
for (const evt of addEvents) {
  seenForChart.add(evt.cn);
  const day = evt.date.slice(0, 10);
  let cumVal = 0;
  for (const cn of seenForChart) {
    const c = getCardByNumber(cn);
    if (c) cumVal += c.price_usd || 0;
  }
  dailyValues[day] = cumVal;
}
const chartDays = Object.keys(dailyValues).sort();
```

The carousel item HTML template:

```js
function cardCarouselItem(card, opts) {
  const borderColor = rarityColors[card.rarity] || '#888';
  const isOwned = opts.owned !== false;
  const priceColor = opts.priceColor || 'var(--owned-border)';
  const price = opts.price || 0;
  return `<div class="card-carousel-item ${isOwned ? '' : 'unowned'}" data-cn="${esc(card.collector_number)}">
    <img src="${esc(card.image_small)}" alt="${displayNameText(card)}"
      style="border:1.5px solid ${isOwned ? borderColor : 'rgba(46,58,82,0.5)'};${card.rarity === 'mythic' || card.rarity === 'rare' ? 'box-shadow:0 0 8px ' + borderColor + '30;' : ''}" loading="lazy">
    <div class="cc-name">${displayName(card)}</div>
    <div class="cc-price" style="color:${priceColor}">$${price.toFixed(2)}</div>
  </div>`;
}
```

Note: define `cardCarouselItem` as a local helper inside `renderPortfolio()` or as a shared function if Dashboard also needs it. Since Dashboard uses simpler carousel items (no price), keep it local to Portfolio.

After setting `innerHTML`, wire click handlers:

```js
el.querySelectorAll('.card-carousel-item[data-cn]').forEach(item => {
  item.addEventListener('click', () => {
    const card = getCardByNumber(item.dataset.cn);
    if (card) openModal(card);
  });
});
```

- [ ] **Step 2: Verify Portfolio tab**

Expected: Hero valuation card with both currencies and rarity value bar, ROI + Still Need row, two card carousels (owned + missing), foil/variant summary, growth chart at bottom.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: redesign Portfolio — hero valuation, card carousels, growth chart"
```

---

### Task 7: Timeline Redesign — Journey Feed

**Files:**
- Modify: `index.html` (JS `renderTimeline()` function, ~line 5055)

**Context:** Spec Section 1. Streak banner, vertical journey timeline, day clusters, inline milestones.

- [ ] **Step 1: Add journey feed CSS**

Add after the existing timeline/milestone CSS (around line 1195):

```css
/* ── Journey Feed ── */
.journey-timeline {
  position: relative; padding-left: 28px;
  border-left: 2px solid var(--ff-border); margin-left: 12px;
}
.journey-day {
  margin-bottom: var(--sp-4); position: relative;
}
.journey-dot {
  position: absolute; left: -34px; top: 2px;
  width: 14px; height: 14px; border-radius: 50%;
  border: 3px solid var(--bg);
  background: var(--surface3);
}
.journey-dot.today { background: var(--accent); box-shadow: 0 0 8px var(--accent); }
.journey-dot.milestone { background: var(--accent2); box-shadow: 0 0 8px rgba(212,168,80,0.4); }
.journey-date {
  font-size: 0.7rem; font-weight: 700; text-transform: uppercase;
  letter-spacing: 0.1em; margin-bottom: 6px; color: var(--text-muted);
}
.journey-date.today { color: var(--accent); }
.journey-chip {
  background: var(--surface); border-radius: 8px;
  padding: var(--sp-2) var(--sp-3); display: inline-flex;
  align-items: center; gap: var(--sp-2);
  border: 1px solid var(--ff-border); font-size: 0.82rem;
  margin: 0 6px 6px 0;
}
.journey-chip img { width: 28px; height: 38px; border-radius: 4px; object-fit: cover; }
.journey-chip .chip-name { font-weight: 600; }
.journey-chip .chip-rarity { font-size: 0.7rem; text-transform: uppercase; }
.journey-chip.removed { opacity: 0.5; }
.journey-chip.removed .chip-name { text-decoration: line-through; }
.journey-milestone-card {
  background: linear-gradient(135deg, rgba(212,168,80,0.08), rgba(212,168,80,0.02));
  border: 1px solid rgba(212,168,80,0.3); border-radius: 8px;
  padding: var(--sp-3) var(--sp-4);
}
.journey-milestone-card .jm-title {
  font-size: 0.82rem; font-weight: 700; color: var(--accent2);
}
.journey-milestone-card .jm-sub {
  font-size: 0.72rem; color: var(--text-muted); margin-top: 2px;
}
.journey-expand {
  background: var(--surface); border: 1px dashed var(--ff-border);
  border-radius: 8px; padding: 6px 10px;
  font-size: 0.78rem; color: var(--text-muted);
  cursor: pointer; display: inline-flex; align-items: center;
  transition: all 0.2s;
}
.journey-expand:hover { border-color: var(--accent); color: var(--text); }
.streak-banner {
  background: var(--ff-panel); border-radius: var(--sp-3);
  padding: var(--sp-4) var(--sp-4);
  display: flex; align-items: center; gap: var(--sp-4);
  border: 1px solid var(--ff-border); margin-bottom: var(--sp-4);
}
```

- [ ] **Step 2: Rewrite `renderTimeline()`**

Replace the entire function. Key logic:

1. **Streak calculation**: walk backwards through sorted add events, count consecutive calendar days
2. **Day grouping**: group all events by `date.slice(0, 10)`, sort days descending
3. **Milestone detection**: walk forward through add events, track when 25%/50%/75%/100% milestones are hit, store the date
4. **Render streak banner** at top
5. **Render vertical journey** with `.journey-timeline` container
6. For each day: render `.journey-day` with dot, date label, and card chips
7. If a milestone was hit on that day, render `.journey-milestone-card` inline
8. If >3 cards in a day, show first 3 + expandable "+N more" chip

```js
function renderTimeline() {
  const el = document.getElementById('timeline');
  const total = allCards.length;
  const owned = ownedSet.size;
  const addEvents = timelineEvents.filter(e => e.type === 'add');

  // Streak calculation
  let streak = 0;
  if (addEvents.length > 0) {
    const today = new Date(); today.setHours(0,0,0,0);
    const dayMs = 86400000;
    const addDays = new Set(addEvents.map(e => e.date.slice(0, 10)));
    let checkDate = new Date(today);
    // If no cards added today, check if yesterday had cards (streak hasn't broken yet today)
    const todayStr = checkDate.toISOString().slice(0, 10);
    if (!addDays.has(todayStr)) {
      checkDate = new Date(today.getTime() - dayMs);
    }
    while (addDays.has(checkDate.toISOString().slice(0, 10))) {
      streak++;
      checkDate = new Date(checkDate.getTime() - dayMs);
    }
  }
  const cardsPerWeek = addEvents.length > 0
    ? (owned / Math.max(1, Math.floor((Date.now() - new Date(addEvents[0].date).getTime()) / 86400000))) * 7
    : 0;

  // Group events by day
  const dayMap = {};
  for (const evt of [...timelineEvents].reverse()) {
    const day = evt.date.slice(0, 10);
    if (!dayMap[day]) dayMap[day] = [];
    dayMap[day].push(evt);
  }
  const sortedDays = Object.keys(dayMap).sort().reverse();

  // Milestones
  const milestones = [
    { target: Math.round(total * 0.25), label: '25%' },
    { target: Math.round(total * 0.5), label: '50%' },
    { target: Math.round(total * 0.75), label: '75%' },
    { target: total, label: '100%' },
  ];
  const seen = new Set();
  const milestoneDays = {};
  for (const evt of addEvents) {
    seen.add(evt.cn);
    const day = evt.date.slice(0, 10);
    for (const m of milestones) {
      if (!m.date && seen.size >= m.target) {
        m.date = day;
        if (!milestoneDays[day]) milestoneDays[day] = [];
        milestoneDays[day].push(m);
      }
    }
  }

  const todayStr = new Date().toISOString().slice(0, 10);
  const rarityColors = { mythic: '#e8702a', rare: '#c9a14e', uncommon: '#72b5b5', common: '#888' };

  // Streak banner
  const streakHTML = `<div class="streak-banner">
    <div style="font-size:2rem">${streak > 0 ? '🔥' : '💤'}</div>
    <div style="flex:1">
      <div style="font-size:1.1rem;font-weight:700;color:var(--accent2)">${streak > 0 ? streak + '-Day Streak!' : 'Start a streak!'}</div>
      <div style="font-size:0.78rem;color:var(--text-muted)">${streak > 0 ? 'Keep adding cards daily' : 'Add a card today to begin'}</div>
    </div>
    <div style="text-align:right">
      <div style="font-weight:700;color:var(--text);font-size:1rem">${cardsPerWeek.toFixed(1)}</div>
      <div style="font-size:0.62rem;color:var(--text-muted)">cards/week</div>
    </div>
  </div>`;

  // Journey days
  let journeyHTML = '';
  if (sortedDays.length === 0) {
    journeyHTML = `<div style="text-align:center;padding:32px 0;color:var(--text-muted)">
      <div style="font-size:2rem;margin-bottom:8px">📭</div>
      <div style="font-size:0.9rem;font-weight:600;margin-bottom:4px">Your collection journey starts here</div>
      <button class="btn btn-primary" style="margin-top:8px;font-size:0.78rem;padding:6px 14px" onclick="openFab()">Add Your First Card</button>
    </div>`;
  } else {
    journeyHTML = '<div class="journey-timeline">';
    for (const day of sortedDays) {
      const events = dayMap[day];
      const isToday = day === todayStr;
      const hasMilestone = milestoneDays[day];

      // Day dot class
      const dotClass = isToday ? 'today' : (hasMilestone ? 'milestone' : '');

      journeyHTML += `<div class="journey-day">
        <div class="journey-dot ${dotClass}"></div>
        <div class="journey-date ${isToday ? 'today' : ''}">${isToday ? 'Today' : new Date(day + 'T00:00:00').toLocaleDateString('en', { weekday: 'short', month: 'short', day: 'numeric' })}</div>`;

      // Inline milestones
      if (hasMilestone) {
        for (const m of hasMilestone) {
          journeyHTML += `<div class="journey-milestone-card" style="margin-bottom:6px">
            <div class="jm-title">🏆 ${m.label} Milestone Reached!</div>
            <div class="jm-sub">${m.target} / ${total} cards · ${new Date(day + 'T00:00:00').toLocaleDateString()}</div>
          </div>`;
        }
      }

      // Card chips (max 3, then expand)
      const addEvts = events.filter(e => e.type === 'add');
      const removeEvts = events.filter(e => e.type === 'remove');
      const packEvts = events.filter(e => e.type === 'add' && e.source === 'pack');
      const visibleAdds = addEvts.slice(0, 3);
      const hiddenCount = addEvts.length - 3;

      // Pack summary if applicable
      if (packEvts.length > 0) {
        const packValue = packEvts.reduce((s, e) => {
          const c = getCardByNumber(e.cn);
          return s + (c ? c.price_usd : 0);
        }, 0);
        journeyHTML += `<div class="journey-chip" style="width:100%;margin-bottom:6px">
          <span>📦</span>
          <span>Opened pack · ${packEvts.length} cards · <span style="color:var(--owned-border)">+$${packValue.toFixed(2)}</span></span>
        </div>`;
      }

      journeyHTML += '<div style="display:flex;flex-wrap:wrap">';
      for (const evt of visibleAdds) {
        const card = getCardByNumber(evt.cn);
        if (!card) continue;
        journeyHTML += `<div class="journey-chip">
          <img src="${esc(card.image_small)}" alt="${displayNameText(card)}" loading="lazy">
          <div>
            <div class="chip-name">${displayName(card)}</div>
            <div class="chip-rarity" style="color:${rarityColors[card.rarity] || '#888'}">${card.rarity} · #${esc(card.collector_number)}</div>
          </div>
        </div>`;
      }
      if (hiddenCount > 0) {
        journeyHTML += `<div class="journey-expand" data-day="${day}">+${hiddenCount} more</div>`;
      }
      journeyHTML += '</div>';

      // Removed cards
      for (const evt of removeEvts) {
        const card = getCardByNumber(evt.cn);
        if (!card) continue;
        journeyHTML += `<div class="journey-chip removed">
          <div style="width:6px;height:6px;border-radius:50%;background:var(--accent);flex-shrink:0"></div>
          <span class="chip-name">${displayName(card)}</span>
        </div>`;
      }

      journeyHTML += '</div>';
    }
    journeyHTML += '</div>';
  }

  el.innerHTML = `<div class="dashboard">${streakHTML}${journeyHTML}</div>`;

  // Expand handlers
  el.querySelectorAll('.journey-expand').forEach(btn => {
    btn.addEventListener('click', () => {
      const day = btn.dataset.day;
      const events = dayMap[day]?.filter(e => e.type === 'add').slice(3) || [];
      let html = '';
      for (const evt of events) {
        const card = getCardByNumber(evt.cn);
        if (!card) continue;
        html += `<div class="journey-chip">
          <img src="${esc(card.image_small)}" alt="${displayNameText(card)}" loading="lazy">
          <div><div class="chip-name">${displayName(card)}</div>
          <div class="chip-rarity" style="color:${rarityColors[card.rarity] || '#888'}">${card.rarity}</div></div>
        </div>`;
      }
      btn.outerHTML = html;
    });
  });
}
```

- [ ] **Step 3: Verify Timeline tab**

Expected: Streak banner at top, vertical journey with day clusters, inline milestones, expandable card chips.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: redesign Timeline — journey feed with streaks, day clusters, inline milestones"
```

---

### Task 8: Binder Redesign — Modern Clean

**Files:**
- Modify: `index.html` (HTML binder top controls + JS `renderBinder()`/`renderSpread()`/`renderPageSelector()` + CSS)

**Context:** Spec Section 6. Stats banner with completion ring, simplified search, slim scrubber.

- [ ] **Step 1: Add binder stats banner CSS**

Add after the existing binder CSS (around line 1395):

```css
/* ── Binder Stats Banner ── */
.binder-stats-banner {
  display: flex; align-items: center; justify-content: space-between;
  padding: var(--sp-3) var(--sp-4);
  background: var(--ff-panel); border-radius: var(--sp-3);
  border: 1px solid var(--ff-border); margin-bottom: var(--sp-2);
  gap: var(--sp-3); flex-wrap: wrap;
}
.binder-stats-banner .bsb-pages {
  font-size: 1.1rem; font-weight: 800; color: var(--gold);
}
.binder-stats-banner .bsb-pages .bsb-of {
  font-size: 0.72rem; color: var(--text-muted); font-weight: 500;
}
.binder-stats-banner .bsb-spread-stats {
  display: flex; align-items: center; gap: var(--sp-2);
}
.binder-stats-banner .bsb-spread-count {
  font-size: 0.78rem; font-weight: 700; color: var(--text);
}
.binder-stats-banner .bsb-spread-label {
  font-size: 0.62rem; color: var(--text-muted);
}
.binder-search-slim {
  display: flex; align-items: center; gap: var(--sp-2);
  margin-bottom: var(--sp-3);
}
.binder-search-slim input {
  flex: 1; min-width: 120px; padding: 7px 12px;
  border-radius: 8px; border: 1px solid var(--surface3);
  background: var(--surface2); color: var(--text);
  font-size: 0.85rem; outline: none; transition: border-color 0.2s;
}
.binder-search-slim input:focus { border-color: var(--accent); }
.binder-search-slim .match-count {
  font-size: 0.75rem; color: var(--text-muted); white-space: nowrap;
}
/* Slim page scrubber */
.page-scrubber {
  display: flex; align-items: center; gap: var(--sp-2);
  padding: 0 36px; margin-top: var(--sp-3);
}
.page-scrubber .scrub-label {
  font-size: 0.62rem; color: var(--text-muted); flex-shrink: 0;
}
.page-scrubber .scrub-track {
  flex: 1; height: 6px; background: var(--surface2);
  border-radius: 3px; position: relative; cursor: pointer;
}
.page-scrubber .scrub-fill {
  height: 100%; border-radius: 3px;
  background: linear-gradient(90deg, var(--accent2), var(--accent));
  position: absolute; top: 0; left: 0;
  transition: width 0.3s ease;
}
.page-scrubber .scrub-thumb {
  width: 12px; height: 12px; border-radius: 50%;
  background: var(--accent); border: 2px solid var(--bg);
  box-shadow: 0 0 6px rgba(91,127,199,0.4);
  position: absolute; top: 50%; transform: translate(-50%, -50%);
  cursor: grab; z-index: 2;
  transition: left 0.3s ease;
}
.page-scrubber .scrub-thumb:active { cursor: grabbing; }
@media (max-width: 700px) {
  .binder-stats-banner { flex-direction: column; align-items: stretch; text-align: center; }
  .binder-stats-banner .bsb-spread-stats { justify-content: center; }
  .page-scrubber .scrub-thumb { width: 16px; height: 16px; }
}
```

- [ ] **Step 2: Update binder HTML structure**

Replace the binder top controls HTML (lines ~2613-2631) with the new stats banner + search. The binder spread and nav buttons stay, but the page-nav-widget is removed. This will be done in JS via `renderBinder()` which rebuilds the controls dynamically.

- [ ] **Step 3: Update `renderBinder()` and `renderSpread()` to use new layout**

The main changes to `renderBinder()`:
- Build the stats banner HTML with page numbers, completion ring SVG, and filter pills
- Build the simplified search bar with match count
- Replace `renderPageSelector()` with a slim scrubber at the bottom
- Add spread ownership stats calculation
- Card slots: owned cards get name overlay on hover, missing cards get the "+" quick-add affordance

The spread completion ring is a small 36x36 SVG donut calculated per-spread:
```js
function spreadCompletionRing(ownedOnSpread, totalOnSpread) {
  const pct = totalOnSpread > 0 ? (ownedOnSpread / totalOnSpread * 100) : 0;
  const r = 14, cx = 18, cy = 18, circ = 2 * Math.PI * r;
  const dash = (pct / 100) * circ;
  return `<svg width="36" height="36" style="display:block">
    <circle cx="${cx}" cy="${cy}" r="${r}" fill="none" stroke="var(--surface2)" stroke-width="3"/>
    <circle cx="${cx}" cy="${cy}" r="${r}" fill="none" stroke="var(--accent)"
      stroke-width="3" stroke-dasharray="${dash.toFixed(1)} ${circ.toFixed(1)}"
      stroke-dashoffset="${(circ * 0.25).toFixed(1)}" stroke-linecap="round"
      transform="rotate(-90 ${cx} ${cy})"/>
    <text x="${cx}" y="${cy}" text-anchor="middle" dominant-baseline="central"
      style="font-size:8px;font-weight:700;fill:var(--text)">${pct.toFixed(0)}%</text>
  </svg>`;
}
```

This is a large rewrite — implement the full `renderBinder()` with the new stats banner at top, simplified search row, existing spread layout (binder-nav + binder-spread), and slim scrubber at bottom.

- [ ] **Step 4: Verify Binder tab**

Expected: Stats banner with page numbers + completion ring + filter pills, search bar with match count, spread with card slots, slim scrubber at bottom.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: redesign Binder — stats banner, completion ring, slim scrubber"
```

---

### Task 9: Collector Tab Redesign — Gallery View

**Files:**
- Modify: `index.html` (JS `renderCollectorTab()` function, ~line 6328)

**Context:** Spec Section 9. Compact stats, pill chips, rarity circles, 4-col gallery grid with overlays.

- [ ] **Step 1: Rewrite `renderCollectorTab()`**

Replace the entire function. Key changes:
- Stats: compact 3-block stats row (donut + foil count + value)
- Filters: horizontal scrollable pill chips using `.pill-scroll` + `.pill-chip`
- Rarity: colored circles using `.rarity-circle`
- Grid: 4-column on desktop, 3 on mobile. Each card has `.card-name-overlay` with name + variant label
- Pagination: centered row with circle buttons
- Keep all existing filter logic, lazy loading, and event delegation

The donut SVG helper (reuse from current code):
```js
function miniDonut(pct, color) {
  const r = 12, cx = 16, cy = 16, circ = 2 * Math.PI * r;
  const dash = (pct / 100) * circ;
  return `<svg width="32" height="32" style="display:block;flex-shrink:0">
    <circle cx="${cx}" cy="${cy}" r="${r}" fill="none" stroke="var(--surface2)" stroke-width="3"/>
    <circle cx="${cx}" cy="${cy}" r="${r}" fill="none" stroke="${color}"
      stroke-width="3" stroke-dasharray="${dash.toFixed(1)} ${circ.toFixed(1)}"
      stroke-dashoffset="${(circ * 0.25).toFixed(1)}" stroke-linecap="round"
      transform="rotate(-90 ${cx} ${cy})"/>
    <text x="${cx}" y="${cy}" text-anchor="middle" dominant-baseline="central"
      style="font-size:7px;font-weight:800;fill:var(--text)">${pct.toFixed(0)}%</text>
  </svg>`;
}
```

Card grid uses `grid-template-columns: repeat(4, 1fr)` on desktop with a media query override. Each slot includes the overlay:
```js
const overlayName = `<div class="card-name-overlay">
  <div class="overlay-name">${esc(card.name)}</div>
  ${label ? `<div class="overlay-sub" style="color:${variantColor}">${esc(label)}</div>` : ''}
</div>`;
```

- [ ] **Step 2: Add collector gallery responsive CSS**

```css
/* ── Collector Gallery ── */
.coll-gallery-grid {
  display: grid; grid-template-columns: repeat(4, 1fr); gap: 6px;
}
@media (max-width: 700px) {
  .coll-gallery-grid { grid-template-columns: repeat(3, 1fr); }
}
.coll-gallery-grid .card-slot:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 16px var(--shadow);
}
.coll-pagination {
  display: flex; align-items: center; justify-content: center;
  gap: var(--sp-3); margin-top: var(--sp-3);
}
.coll-pagination .coll-pg-btn {
  width: 28px; height: 28px; border-radius: 50%;
  background: var(--surface2); border: 1px solid var(--ff-border);
  color: var(--text-muted); font-size: 0.8rem;
  cursor: pointer; display: flex; align-items: center; justify-content: center;
  transition: all 0.2s;
}
.coll-pagination .coll-pg-btn:hover { background: var(--accent); color: #fff; }
.coll-pagination .coll-pg-btn:disabled { opacity: 0.3; cursor: default; }
.coll-pagination .coll-pg-btn:disabled:hover { background: var(--surface2); color: var(--text-muted); }
@media (max-width: 700px) {
  .coll-pagination .coll-pg-btn { width: 36px; height: 36px; min-height: 44px; }
}
```

- [ ] **Step 3: Verify Collector tab**

Expected: Compact stats row, scrollable pill chips, rarity circles, 4-col gallery with name overlays, centered pagination.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: redesign Collector — gallery view with pill chips, rarity circles, overlays"
```

---

### Task 10: Settings Redesign — Preferences Hub

**Files:**
- Modify: `index.html` (JS `renderSettingsForm()` + sync modal removal + CSS)

**Context:** Spec Section 3. Profile summary, live preview, inline sync, appearance section.

- [ ] **Step 1: Rewrite `renderSettingsForm()`**

Replace the entire function to render the new section layout:

1. **Profile Summary Card**: full-width card showing collection summary + sync status
2. **Binder Layout**: keep preset dropdown + custom inputs, add live visual preview (small grid of grey rectangles)
3. **Collection Scope**: keep toggles, add card-count badges
4. **Cloud Sync** (inline, replaces modal): token input + connect button when unconfigured; status + push/pull/disconnect when configured
5. **Appearance**: theme toggle switch
6. **Data & Cache**: cache status, refresh buttons, export/import
7. **Danger Zone**: same but restyled

The live binder preview is a small grid:
```js
function binderPreview(rows, cols) {
  const cells = [];
  for (let i = 0; i < rows * cols; i++) {
    cells.push('<div style="aspect-ratio:5/7;background:var(--surface2);border-radius:2px;border:1px solid var(--ff-border)"></div>');
  }
  return `<div style="display:grid;grid-template-columns:repeat(${cols},1fr);gap:3px;max-width:180px;margin:8px auto">${cells.join('')}</div>`;
}
```

- [ ] **Step 2: Move sync logic inline**

Move the sync configuration UI from the modal into the settings section. The `openSyncModal()` function now switches to Settings tab and scrolls to the sync section:

```js
function openSyncModal() {
  switchTab('settings');
  setTimeout(() => {
    const syncSection = document.getElementById('settings-sync-section');
    if (syncSection) syncSection.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }, 100);
}
```

Keep the sync-btn click handler pointing to `openSyncModal()` (it now just navigates to settings).

- [ ] **Step 3: Remove the sync modal HTML and CSS**

Remove the `#sync-modal-overlay` div from the HTML (around lines 2698-2713).
Remove or comment out the `.sync-modal-overlay`, `.sync-modal`, and related CSS rules (around lines 240-290).

- [ ] **Step 4: Add appearance section CSS**

```css
.settings-theme-toggle {
  display: flex; align-items: center; justify-content: space-between;
  padding: var(--sp-3) 0;
}
```

- [ ] **Step 5: Verify Settings tab**

Expected: Profile summary at top with sync status, binder layout with live preview, collection scope with counts, inline cloud sync, appearance toggle, data/cache section, danger zone.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: redesign Settings — preferences hub with inline sync, live preview"
```

---

### Task 11: FAB Smart Panel Redesign

**Files:**
- Modify: `index.html` (HTML fab panel + JS autocomplete rendering + CSS)

**Context:** Spec Section 2. Search-first, inline card previews, quick actions bar.

- [ ] **Step 1: Update FAB panel CSS**

Update `.fab-panel` width to 400px. Update mobile panel to `calc(100vw - 24px)`.

Add new autocomplete result CSS:
```css
/* ── Smart Panel Autocomplete ── */
.ac-card-row {
  display: flex; align-items: center; gap: var(--sp-3);
  padding: var(--sp-2) var(--sp-3);
  border-bottom: 1px solid var(--surface2);
  transition: background 0.15s, border-color 0.15s;
  border: 1px solid transparent; border-radius: 8px;
  margin: 0 var(--sp-2) var(--sp-1);
}
.ac-card-row:hover, .ac-card-row.hl {
  background: var(--surface2);
  border-color: var(--accent);
}
.ac-card-row img {
  width: 48px; height: 67px; border-radius: 6px;
  object-fit: cover; flex-shrink: 0; cursor: pointer;
}
.ac-card-row .ac-info { flex: 1; min-width: 0; }
.ac-card-row .ac-name { font-weight: 700; font-size: 0.9rem; }
.ac-card-row .ac-detail { font-size: 0.72rem; color: var(--text-muted); margin-top: 2px; }
.ac-card-row .ac-prices { display: flex; gap: 6px; margin-top: 4px; }
.ac-card-row .ac-prices span { font-size: 0.7rem; font-weight: 600; }
.ac-card-row .ac-actions { display: flex; flex-direction: column; gap: 4px; flex-shrink: 0; }
.ac-card-row .ac-add-btn {
  background: var(--accent); color: #fff; border: none;
  border-radius: 6px; padding: 6px 12px; font-size: 0.75rem;
  font-weight: 700; cursor: pointer; transition: all 0.15s;
}
.ac-card-row .ac-add-btn:hover { filter: brightness(1.1); }
.ac-card-row .ac-add-btn.success { background: var(--owned-border); }
.ac-card-row .ac-foil-btn {
  background: none; border: 1px solid var(--gold);
  color: var(--gold); border-radius: 6px; padding: 4px 10px;
  font-size: 0.68rem; font-weight: 700; cursor: pointer;
  transition: all 0.15s;
}
.ac-card-row .ac-foil-btn:hover { background: rgba(212,168,80,0.1); }
.ac-card-row .ac-owned-badge {
  font-size: 0.68rem; color: var(--owned-border);
  padding: 4px 8px; border: 1px solid var(--owned-border);
  border-radius: 6px; font-weight: 700;
}
/* Quick actions bar */
.fab-quick-actions {
  display: flex; gap: var(--sp-2);
  padding: var(--sp-3) var(--sp-4); border-top: 1px solid var(--surface2);
}
.fab-quick-actions .fab-qa-btn {
  flex: 1; padding: var(--sp-3); border: none; border-radius: 8px;
  font-size: 0.85rem; font-weight: 700; cursor: pointer;
  text-align: center; transition: all 0.2s;
}
.fab-qa-btn.pack {
  background: linear-gradient(135deg, var(--accent), var(--accent2)); color: #fff;
}
.fab-qa-btn.pack:hover { transform: translateY(-2px); box-shadow: 0 4px 12px var(--shadow); }
.fab-qa-btn.recent {
  background: var(--surface2); color: var(--text-muted);
}
.fab-qa-btn.recent:hover { color: var(--text); }
```

- [ ] **Step 2: Update FAB panel HTML**

Replace the `fab-mode-toggle` div with the new quick actions bar at the bottom. Move it after the quick/pack mode containers:

```html
<div class="fab-quick-actions" id="fab-quick-actions">
  <button class="fab-qa-btn pack" id="fab-qa-pack">📦 Open Pack</button>
  <button class="fab-qa-btn recent" id="fab-qa-recent">📋 Recent</button>
</div>
```

Remove the `fab-mode-toggle`, `fab-mode-slider`, and `fab-mode-btn` elements.

- [ ] **Step 3: Update autocomplete rendering JS**

Find the autocomplete render code (the part that builds `.ac-item` elements). Replace the item template to use the new `.ac-card-row` layout with thumbnail, card info, prices, and action buttons.

The autocomplete should trigger after 1 character (find the length check and change from 2 to 1).

Each result item:
```js
const isOwned = ownedSet.has(card.collector_number);
const isFoilOwned = hasFoil(card.collector_number);
const canFoil = (card.finishes || []).includes('foil');
const actionHTML = isOwned
  ? `<span class="ac-owned-badge">Owned</span>`
  : `<button class="ac-add-btn" data-cn="${esc(card.collector_number)}">+ Add</button>`;
const foilHTML = canFoil && !isFoilOwned
  ? `<button class="ac-foil-btn" data-cn="${esc(card.collector_number)}">✦ Foil</button>`
  : '';

return `<div class="ac-card-row" data-cn="${esc(card.collector_number)}">
  <img src="${esc(card.image_small)}" alt="${displayNameText(card)}" loading="lazy">
  <div class="ac-info">
    <div class="ac-name">${displayName(card)}</div>
    <div class="ac-detail">#${esc(card.collector_number)} · ${card.rarity} · ${esc(card.type_line || '')}</div>
    <div class="ac-prices">
      ${card.price_usd > 0 ? `<span style="color:var(--owned-border)">$${card.price_usd.toFixed(2)}</span>` : ''}
      ${card.price_usd_foil > 0 ? `<span style="color:var(--gold)">✦ $${card.price_usd_foil.toFixed(2)}</span>` : ''}
    </div>
  </div>
  <div class="ac-actions">${actionHTML}${foilHTML}</div>
</div>`;
```

- [ ] **Step 4: Wire quick action buttons and new autocomplete actions**

```js
document.getElementById('fab-qa-pack').addEventListener('click', () => {
  // Switch to pack mode (reuse existing pack logic)
  document.getElementById('fab-quick-mode').style.display = 'none';
  document.getElementById('fab-pack-mode').style.display = 'flex';
  document.getElementById('fab-quick-actions').style.display = 'none';
  renderPackMode();
});
document.getElementById('fab-qa-recent').addEventListener('click', () => {
  // Ensure quick mode visible with recent list
  document.getElementById('fab-quick-mode').style.display = '';
  document.getElementById('fab-pack-mode').style.display = 'none';
  document.getElementById('fab-quick-actions').style.display = '';
  document.getElementById('search-input').value = '';
  // Close autocomplete, show recent
  document.getElementById('autocomplete').classList.remove('show');
  renderFabRecent();
});
```

Wire the Add button: clicking `.ac-add-btn` calls the existing `addCard()` logic, then shows a "✓" success state for 1 second:
```js
// Inside autocomplete event delegation
if (e.target.classList.contains('ac-add-btn')) {
  const cn = e.target.dataset.cn;
  // ... existing add logic ...
  e.target.textContent = '✓';
  e.target.classList.add('success');
  setTimeout(() => { e.target.textContent = '+ Add'; e.target.classList.remove('success'); }, 1000);
}
if (e.target.classList.contains('ac-foil-btn')) {
  const cn = e.target.dataset.cn;
  addFoil(cn);
  e.target.textContent = '✦ Added';
  e.target.disabled = true;
}
```

Clicking the thumbnail image opens the modal:
```js
if (e.target.tagName === 'IMG' && e.target.closest('.ac-card-row')) {
  const cn = e.target.closest('.ac-card-row').dataset.cn;
  const card = getCardByNumber(cn);
  if (card) openModal(card);
}
```

- [ ] **Step 5: Verify FAB panel**

Expected: Larger search input, rich card rows with thumbnails + prices + Add/Foil buttons, quick actions bar at bottom.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: redesign FAB — smart panel with card previews, quick actions bar"
```

---

### Task 12: Modal Refinement

**Files:**
- Modify: `index.html` (CSS modal styles + JS modal touch handlers)

**Context:** Spec Section 5C. Frosted glass, spring animation, swipe-to-dismiss.

- [ ] **Step 1: Update modal overlay CSS for frosted glass**

Find `.modal-overlay` CSS (around line 1559). Add backdrop-filter:

```css
.modal-overlay {
  display: none; position: fixed; inset: 0;
  background: rgba(6,8,18,0.65); z-index: 1000;
  justify-content: center; align-items: center;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}
```

Update `[data-theme="dark"]` modal-bg to lower opacity since blur adds separation:
```css
--modal-bg: rgba(6,8,18,0.65);
```

And for light theme:
```css
--modal-bg: rgba(60,48,28,0.35);
```

- [ ] **Step 2: Update modal animations**

Replace `@keyframes modal-in` and `@keyframes modal-out`:

```css
@keyframes modal-in {
  from { transform: scale(0.85) translateY(20px) rotate(-1deg); opacity: 0; }
  to { transform: scale(1) translateY(0) rotate(0deg); opacity: 1; }
}
@keyframes modal-out {
  from { transform: scale(1) translateY(0); opacity: 1; }
  to { transform: scale(0.92) translateY(12px); opacity: 0; }
}
```

Ensure the `.modal-overlay.visible .modal` animation uses the spring bezier:
```css
.modal-overlay.visible .modal {
  animation: modal-in 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}
.modal-overlay.closing .modal {
  animation: modal-out 0.2s ease both;
}
```

- [ ] **Step 3: Add overlay cursor hint**

```css
.modal-overlay.visible { cursor: pointer; }
.modal-overlay.visible .modal { cursor: default; }
```

- [ ] **Step 4: Add swipe-to-dismiss on mobile**

Add this JS after the `closeModal` function:

```js
// ── Modal Swipe-to-Dismiss ──
(function initModalSwipe() {
  const modal = document.getElementById('modal');
  let startY = 0, currentY = 0, isDragging = false;
  modal.addEventListener('touchstart', e => {
    if (e.target.closest('.card-visualizer') || e.target.closest('.card-3d')) return;
    startY = e.touches[0].clientY;
    isDragging = true;
    modal.style.transition = 'none';
  }, { passive: true });
  modal.addEventListener('touchmove', e => {
    if (!isDragging) return;
    currentY = e.touches[0].clientY;
    const dy = currentY - startY;
    if (dy > 0) {
      modal.style.transform = `translateY(${dy}px)`;
      modal.style.opacity = Math.max(0.5, 1 - dy / 400);
    }
  }, { passive: true });
  modal.addEventListener('touchend', () => {
    if (!isDragging) return;
    isDragging = false;
    modal.style.transition = '';
    const dy = currentY - startY;
    if (dy > 100) {
      closeModal();
    } else {
      modal.style.transform = '';
      modal.style.opacity = '';
    }
    startY = 0; currentY = 0;
  });
})();
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "style: modal refinement — frosted glass, spring animation, swipe-to-dismiss"
```

---

### Task 13: Card Slot Polish (Binder)

**Files:**
- Modify: `index.html` (CSS card-slot styles + JS parallax handler)

**Context:** Spec Section 5B. Parallax tilt on hover, breathing glow, card silhouette for missing.

- [ ] **Step 1: Add card slot polish CSS**

```css
/* ── Card Slot Polish ── */
@keyframes slot-breathe {
  0%, 100% { box-shadow: 0 0 6px var(--ff-glow); }
  50% { box-shadow: 0 0 12px var(--ff-glow); }
}
.card-slot.owned {
  animation: slot-breathe 3s ease-in-out infinite;
  box-shadow: 0 2px 8px rgba(91,127,199,0.12);
}
.card-slot.owned:nth-child(2n) { animation-delay: 1s; }
.card-slot.owned:nth-child(3n) { animation-delay: 2s; }
/* Missing card silhouette */
.card-slot .placeholder-silhouette {
  width: 70%; height: 75%;
  border: 1px dashed rgba(138,144,168,0.12);
  border-radius: 3px;
  display: flex; flex-direction: column;
  align-items: center; justify-content: center; gap: 2px;
}
.card-slot .placeholder-silhouette .sil-num {
  font-size: 0.65rem; color: rgba(138,144,168,0.35); font-weight: 700;
}
.card-slot .placeholder-silhouette .sil-name {
  font-size: 0.45rem; color: rgba(138,144,168,0.25);
}
.card-slot .quick-add-circle {
  width: 14px; height: 14px; border-radius: 50%;
  border: 1px solid rgba(91,127,199,0.3);
  display: flex; align-items: center; justify-content: center;
  margin-top: 2px; transition: all 0.2s;
}
.card-slot .quick-add-circle span {
  font-size: 0.55rem; color: rgba(91,127,199,0.5);
}
.card-slot:hover .quick-add-circle {
  border-color: var(--accent); background: rgba(91,127,199,0.1);
}
.card-slot:hover .quick-add-circle span { color: var(--accent); }
@media (max-width: 700px) {
  .card-slot .quick-add-circle { width: 20px; height: 20px; }
}
```

- [ ] **Step 2: Add parallax tilt JS (desktop only)**

Add after the binder event listeners:

```js
// ── Card Slot Parallax Tilt (desktop) ──
if (!('ontouchstart' in window)) {
  document.addEventListener('mousemove', e => {
    const slot = e.target.closest('.card-slot');
    if (!slot || !slot.closest('#binder-spread, #tab-collector')) return;
    const rect = slot.getBoundingClientRect();
    const x = (e.clientX - rect.left) / rect.width - 0.5;
    const y = (e.clientY - rect.top) / rect.height - 0.5;
    slot.style.transform = `perspective(600px) rotateX(${(-y * 8).toFixed(1)}deg) rotateY(${(x * 8).toFixed(1)}deg) scale(1.03)`;
    slot.style.transition = 'transform 0.1s ease';
  });
  document.addEventListener('mouseout', e => {
    const slot = e.target.closest('.card-slot');
    if (slot) {
      slot.style.transform = '';
      slot.style.transition = 'transform 0.3s ease';
    }
  });
}
```

- [ ] **Step 3: Update missing card placeholder in `renderSpread()`**

Find where missing card placeholders are rendered in `renderSpread()`. Replace the placeholder HTML with the silhouette:

```js
// Inside the slot rendering for missing cards:
`<div class="placeholder" style="display:flex;flex-direction:column;justify-content:center;align-items:center;height:100%;opacity:0.6">
  <div class="placeholder-silhouette">
    <span class="sil-num">#${esc(card.collector_number)}</span>
    <span class="sil-name">${esc(card.name.length > 12 ? card.name.slice(0,12) + '…' : card.name)}</span>
    <div class="quick-add-circle"><span>+</span></div>
  </div>
</div>`
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "style: card slot polish — parallax tilt, breathing glow, silhouette placeholders"
```

---

### Task 14: Discoverability — Tooltip Pulses & Empty States

**Files:**
- Modify: `index.html` (CSS + JS discovery system + empty state updates)

**Context:** Spec Sections 4A and 4B.

- [ ] **Step 1: Add discovery pulse CSS**

```css
/* ── Discovery Pulses ── */
.discover-pulse {
  position: relative;
}
.discover-pulse::after {
  content: '';
  position: absolute; top: -2px; right: -2px;
  width: 8px; height: 8px; border-radius: 50%;
  background: var(--accent);
  animation: discover-pulse 1.5s ease-in-out infinite;
  pointer-events: none; z-index: 10;
}
@keyframes discover-pulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(91,127,199,0.4); }
  50% { box-shadow: 0 0 0 6px rgba(91,127,199,0); }
}
```

- [ ] **Step 2: Add discovery tracking JS**

Add after the state variables (around line 3420):

```js
// ── Feature Discovery ──
const DISCOVER_KEY = 'fin-discovered-features';
function loadDiscovered() {
  try { return JSON.parse(localStorage.getItem(DISCOVER_KEY) || '{}'); }
  catch { return {}; }
}
function markDiscovered(feature) {
  const d = loadDiscovered();
  d[feature] = true;
  localStorage.setItem(DISCOVER_KEY, JSON.stringify(d));
  document.querySelectorAll(`[data-discover="${feature}"]`).forEach(el => {
    el.classList.remove('discover-pulse');
  });
}
function applyDiscoverPulses() {
  const d = loadDiscovered();
  const features = [
    { id: 'fab', selector: '#fab-btn' },
    { id: 'sync', selector: '#sync-btn' },
    { id: 'theme', selector: '#theme-toggle' },
  ];
  if (binderConfig.scope?.collectorBinder) {
    features.push({ id: 'collector', selector: '#tab-btn-collector' });
    features.push({ id: 'collector-bnav', selector: '#bnav-btn-collector' });
  }
  features.push({ id: 'binder-search', selector: '#binder-search-input' });
  for (const f of features) {
    if (d[f.id]) continue;
    const el = document.querySelector(f.selector);
    if (el) {
      el.classList.add('discover-pulse');
      el.setAttribute('data-discover', f.id);
    }
  }
}
```

- [ ] **Step 3: Wire discovery to existing interactions**

Add `markDiscovered()` calls to existing click handlers:

- In the FAB open handler: `markDiscovered('fab');`
- In the sync button handler: `markDiscovered('sync');`
- In the theme toggle handler: `markDiscovered('theme');`
- In the switchTab handler, when `tabName === 'collector'`: `markDiscovered('collector'); markDiscovered('collector-bnav');`
- In the binder search input focus handler: `markDiscovered('binder-search');`

Call `applyDiscoverPulses()` after `renderAll()` in the initialization code.

- [ ] **Step 4: Update empty states across tabs**

The empty states in each render function should already be partially updated by the tab rewrites (Tasks 5-10). Verify and add CTA buttons to any remaining "No data yet" messages:

- Dashboard session strip: already has CTA in Task 5
- Timeline empty: already has CTA in Task 7
- Portfolio carousels when empty: add "Start collecting to track your portfolio value" with button
- FAB recent list: update the empty text to "Search above or open a booster pack to begin"

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add discoverability — tooltip pulses, actionable empty states"
```

---

## Verification Checklist

After all tasks are complete, manually verify each tab on both desktop (>700px) and mobile (<700px viewport):

- [ ] Dashboard: hero ring, stat pills, session strip, binder map
- [ ] Portfolio: hero valuation, rarity bar, card carousels, growth chart
- [ ] Timeline: streak banner, journey feed, milestones, expand chips
- [ ] Binder: stats banner, completion ring, search, slim scrubber, card silhouettes
- [ ] Collector: compact stats, pill chips, rarity circles, 4-col gallery, overlays
- [ ] Settings: profile summary, live preview, inline sync, appearance
- [ ] FAB: smart panel, card previews, quick actions
- [ ] Modal: frosted glass, spring animation, swipe-to-dismiss
- [ ] Tab transitions: directional slides, mobile swipe
- [ ] Bottom nav: labels, accent border
- [ ] Skeleton loading: shown during initial data fetch
- [ ] Discovery pulses: visible on first load, disappear after interaction
- [ ] Light theme: verify all changes work in both themes
