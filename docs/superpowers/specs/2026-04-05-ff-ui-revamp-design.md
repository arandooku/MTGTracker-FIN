# FF UI Revamp — Design Spec

## Context

The MTG FIN tracker currently uses a generic Catppuccin-inspired color scheme with Inter font. It has no Final Fantasy visual identity despite being a Final Fantasy set tracker. The goal is to revamp the UI to feel authentically FF — using the iconic blue gradient menu boxes, FF-style typography, crystal/materia motifs, and HP/MP-style progress bars — while keeping all existing functionality intact.

## Design Decisions

- **Dark theme**: FF blue gradient menu boxes with double-border inset (FF7–FF10 iconic style)
- **Light theme**: Warm Sand — sandy beige backgrounds, amber-gold accents, rich brown text (FFX Zanarkand feel)
- **Typography**: Cinzel serif for headings (FF logo style), Rajdhani for UI/body text (FF10+ menu feel)
- **Motifs**: Crystal hexagon icon, diamond cursor indicators, materia-colored accents, HP/MP-style glowing gradient bars
- **Animations**: Crystal rotation for loading, mako-pulse for sync, damage-number pop for toasts, cure-sparkle for card additions

## Color System

### Dark Theme (FF Blue Gradient)

| Variable | Current | New | Notes |
|----------|---------|-----|-------|
| `--bg` | `#1e1e2e` | `#080818` | Deep FF midnight blue-black |
| `--surface` | `#313244` | `#0c0c42` | Base surface color (FF panel gradient applied via `.dash-card`/`.ff-box` class, not variable — CSS vars can't hold gradients) |
| `--surface2` | `#45475a` | `#1a1a5e` | Lighter blue for secondary surfaces |
| `--surface3` | `#585b70` | `#2a2a6e` | Tertiary surface |
| `--accent` | `#f38ba8` | `#6688ee` | FF crystal blue |
| `--accent2` | `#cba6f7` | `#ffdd44` | FF gold for values/highlights |
| `--text` | `#cdd6f4` | `#ffffff` | Pure white for high contrast |
| `--text-muted` | `#a6adc8` | `#bbbbdd` | Readable purple-white |
| `--owned-border` | `#a6e3a1` | `#44cc66` | Green with glow |
| `--slot-bg` | `#181825` | `#0a0a3a` | Deep blue slot |
| `--shadow` | `rgba(17,17,27,0.5)` | `rgba(10,10,60,0.5)` | Blue-tinted shadow |
| `--modal-bg` | `rgba(17,17,27,0.85)` | `rgba(6,6,20,0.88)` | Dark blue overlay |
| `--chart-empty` | `#45475a` | `#1a1a4e` | Dark blue empty |

**New dark-only variables:**
- `--ff-border`: `#6666bb` — panel border color
- `--ff-border-inner`: `#3a3a99` — inner inset border
- `--ff-glow`: `rgba(40,40,140,0.3)` — box glow shadow
- `--gold`: `#ffdd44` — monetary values, selected states
- `--cursor-color`: `#ffdd44` — diamond/arrow cursor

### Light Theme (Warm Sand)

| Variable | Current | New | Notes |
|----------|---------|-----|-------|
| `--bg` | `#f5f3ef` | `#f5f0e6` | Sandy beige |
| `--surface` | `#eae7e0` | `#f0e8d8` | Base surface (panel gradient applied via class, same as dark) |
| `--surface2` | `#ddd9d0` | `#e8dcc8` | Lighter sand |
| `--surface3` | `#c5c0b6` | `#d0c4a8` | Warm border tone |
| `--accent` | `#c0392b` | `#cc8822` | Amber gold |
| `--accent2` | `#7e57c2` | `#996622` | Deep gold |
| `--text` | `#3b3632` | `#3a2e18` | Rich dark brown |
| `--text-muted` | `#655c52` | `#8a7a58` | Warm muted brown |
| `--owned-border` | `#4caf50` | `#558833` | Earthy green |
| `--slot-bg` | `#e8e4dc` | `#efe8d8` | Light sand slot |
| `--shadow` | `rgba(59,54,50,0.08)` | `rgba(100,80,40,0.08)` | Warm shadow |
| `--modal-bg` | `rgba(59,54,50,0.55)` | `rgba(60,48,28,0.55)` | Warm overlay |
| `--chart-empty` | `#ddd9d0` | `#ddd4c0` | Sand empty |

**New light-only variables:**
- `--ff-border`: `#d0c4a8` — panel border (warm)
- `--ff-border-inner`: `#c8b898` — inner inset border
- `--ff-glow`: `rgba(180,140,40,0.06)` — subtle warm glow
- `--gold`: `#996622` — monetary values
- `--cursor-color`: `#8b6914` — cursor in light mode

## Typography

### Font Loading
Replace the current Inter-only stack with:
```css
@import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;700;900&family=Rajdhani:wght@400;500;600;700&display=swap');
```

### Font Assignments
| Element | Current | New |
|---------|---------|-----|
| Body / UI text | `'Inter', sans-serif` | `'Rajdhani', sans-serif` |
| Page title | Inter 800 | `'Cinzel', serif` 700 |
| Dashboard headings (h3) | Inter uppercase | `'Cinzel', serif` 600, uppercase |
| Big numbers (`.dash-big-num`) | Inter 800, gradient | Rajdhani 700, `var(--gold)` color with text-shadow glow |
| Tab labels | Inter 500 | Rajdhani 600, `letter-spacing: 0.1em` |
| Stat labels | Inter uppercase | Rajdhani 600, `letter-spacing: 0.12em` |
| Modal card name | — | `'Cinzel', serif` for card names |

### Typographic Details
- All uppercase labels get `letter-spacing: 0.08em` to `0.15em`
- Number values use `font-variant-numeric: tabular-nums` for alignment
- Active tab/menu items get `►` arrow prefix or `◆` diamond cursor with blinking animation

## Panel Styling (The FF Menu Box)

The single most impactful change — replacing flat surface cards with the iconic FF gradient panel.

### Dark Theme Panel
```css
.dash-card, .stat-box, .modal, [ff-panel] {
  background: linear-gradient(135deg, #12125a 0%, #0c0c42 40%, #080834 100%);
  border: 2px solid var(--ff-border);
  box-shadow:
    inset 0 0 0 1px var(--ff-border-inner),
    0 0 14px var(--ff-glow),
    0 4px 16px rgba(0,0,0,0.4);
  border-radius: 5px;
}
/* Inner frame line (the classic double-border) */
.dash-card::before {
  content: '';
  position: absolute;
  inset: 2px;
  border: 1px solid rgba(100,100,180,0.08);
  border-radius: 3px;
  pointer-events: none;
}
```

### Light Theme Panel
```css
[data-theme="light"] .dash-card {
  background: linear-gradient(135deg, #faf5ea, #f0e8d8);
  border: 1.5px solid var(--ff-border);
  box-shadow: 0 2px 8px var(--ff-glow);
}
/* Gold top-line accent */
[data-theme="light"] .dash-card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; height: 2px;
  background: linear-gradient(90deg, transparent, rgba(200,140,40,0.2), transparent);
}
```

## Progress Bars (HP/MP Style)

Replace flat progress fills with glowing gradient bars:

| Bar Type | Gradient | Glow |
|----------|----------|------|
| Collection / general | `#4466dd` → `#7799ff` | `0 0 10px rgba(80,120,255,0.5)` |
| HP / green (owned) | `#22bb55` → `#55ee77` | `0 0 8px rgba(40,220,100,0.4)` |
| MP / blue (foils) | `#3355dd` → `#5588ff` | `0 0 8px rgba(60,100,255,0.4)` |
| Gold (value) | `#dd9900` → `#ffcc22` | `0 0 8px rgba(255,200,0,0.3)` |
| Mythic (orange-red) | `#dd3322` → `#ff7755` | `0 0 8px rgba(255,70,50,0.3)` |

Light theme uses same gradients but without glow shadows, using `border: 1px solid var(--ff-border)` on tracks.

## Crystal Icon

Replace the app logo/brand area with a hexagonal crystal:
```css
.crystal-icon {
  width: 22px; height: 22px;
  background: radial-gradient(circle, rgba(120,160,255,0.9), rgba(60,60,220,0.4) 60%, transparent 70%);
  clip-path: polygon(50% 0%, 85% 25%, 85% 75%, 50% 100%, 15% 75%, 15% 25%);
  animation: crystalGlow 3s ease-in-out infinite;
}
```
Light theme crystal uses gold tones: `radial-gradient(circle, #eebb55, #cc8822 60%, #885511)`.

## Animations

### New Keyframes

| Animation | Purpose | Replaces |
|-----------|---------|----------|
| `crystalGlow` | Crystal icon pulse (3s ease-in-out infinite) | — |
| `crystalRotate` | Loading spinner (4s linear infinite) | `spin` keyframe |
| `cursorBlink` | Diamond cursor blink (1s steps(2) infinite) | — |
| `makoGlow` | Sync status pulse (2s ease-in-out) | `pulse-dot` |
| `dmgPop` | Toast number pop-up (1.5s ease-out) | `toast-in` (enhance, don't replace) |

### Enhanced Existing Animations
- `card-enter`: Keep 0.4s ease, but add subtle blue glow fade-in for dark theme
- `modal-in`: Keep cubic-bezier bounce, add blue outer-glow on appearance
- Toast: Damage-number style float-up-and-fade in addition to current pop-in
- Celebrations: Keep all existing (fireworks, sparkles, gold shower) — they already work well

## Components to Modify

### Header
- Crystal icon replaces/supplements the brand area
- Title uses Cinzel serif
- "MTG Collection Tracker" subtitle in Rajdhani with `letter-spacing: 0.2em`
- Header buttons get `border: 1px solid var(--ff-border)` styling

### Tab Navigation
- Active tab gets `►` cursor prefix in dark theme
- Active indicator bar uses accent blue with glow
- Inactive tabs use `var(--text-muted)` 
- Desktop pill tray and mobile bottom nav both get the same treatment

### Dashboard Cards
- All `.dash-card` elements get FF menu box styling
- `.dash-big-num` uses `var(--gold)` color with `text-shadow` glow instead of gradient clip
- Card headings (h3) use Cinzel serif

### Binder
- Slots use darker blue backgrounds `var(--slot-bg)` 
- Owned slots glow green: `border-color: var(--owned-border); box-shadow: 0 0 10px rgba(60,210,90,0.2)`
- Page labels in Rajdhani uppercase

### Modal
- Gets full FF menu box treatment (gradient + double border + glow)
- Card name in Cinzel serif
- Price chips with dark inner background + gold values
- Action buttons styled as FF menu items with diamond cursor

### FAB
- Blue gradient circle with glow: `linear-gradient(135deg, #3355dd, #5577ff)`
- `box-shadow: 0 0 22px rgba(70,100,255,0.45)`
- Light theme: amber gradient `linear-gradient(135deg, #cc8822, #eebb55)`

## What Does NOT Change

- All existing functionality (data layer, localStorage, Scryfall API, cloud sync)
- HTML structure (no new elements needed beyond decorative pseudo-elements)
- Responsive breakpoints and layout grid
- Foil shimmer system (already has great variant-specific effects)
- Celebration animations (fireworks, sparkles, gold shower, rainbow burst)
- Reduced motion support
- PWA manifest generation

## Verification

1. Open `index.html` in browser
2. Verify dark theme: FF blue gradient panels, white text, gold values, crystal icon, glowing bars
3. Toggle to light theme: warm sand panels, brown text, amber accents, gold crystal
4. Check all 6 tabs render correctly with new styles
5. Test card modal: Cinzel name, FF-framed panel, gold prices, styled buttons
6. Test FAB: blue/amber circle with glow, search panel in FF box style
7. Test binder: slot borders, owned glow, page navigation
8. Test mobile: bottom nav styling, responsive layout
9. Verify `prefers-reduced-motion` still disables animations
10. Check foil shimmer effects still work correctly on collector tab
