# Dicey Game Layout Architecture

Developer reference for building new games on the Dicey platform. Every game is a single-file HTML/CSS/JS page following this exact layout structure.

---

## Layout Stack

```
┌─────────────────────── 420px max ───────────────────────┐
│  HEADER  (50px)                                         │
│  Logo / Brand         Balance Pill  Avatar              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  GAME VIEWPORT  (dynamic height — fills remaining)      │
│                                                         │
│  ┌── Result Pills ──────────── Rakeback ──┐  abs top   │
│  └────────────────────────────────────────┘             │
│                                                         │
│           [ Game Content — flex: 1 ]                    │
│           Dice icon, Limbo multiplier, etc.             │
│                                                         │
│  ┌─────────── Game Bottom Card ───────────┐  frosted   │
│  │  Sliders / Options / Controls          │  glass     │
│  └────────────────────────────────────────┘             │
│                                                         │
├──────────────── seamless join (no border) ──────────────┤
│                                                         │
│  BET CONTROLS  (fixed height — flex-shrink: 0)          │
│  ┌────────────────────────────────────────┐             │
│  │  Bet Button  (48px)                    │             │
│  │  Bet Amount  [$] [input] [½] [2×] [Max]│             │
│  │  Mode Toggle  [Manual Bet | Auto Bet]  │             │
│  │  Manual Settings / Auto Settings       │             │
│  │  Settings Bar  ⚙ ⓘ 📊      🛡         │             │
│  └────────────────────────────────────────┘             │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  BOTTOM NAV  (fixed, ~60px)                             │
│  Explore    Casino    Sports                            │
└─────────────────────────────────────────────────────────┘
```

---

## App Shell

The outermost container. Every game wraps everything in this.

```css
.app-shell {
  width: 100%;
  max-width: 420px;        /* Mobile-first — never wider */
  min-height: 100dvh;      /* Full dynamic viewport */
  display: flex;
  flex-direction: column;
  background: #0e100f;
  border: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 70px;    /* Reserve space for fixed bottom-nav */
}
```

The body centers it:
```css
body {
  display: flex;
  flex-direction: column;
  align-items: center;     /* Centers the 420px shell */
}
```

**Key:** `padding-bottom: 70px` prevents content from hiding behind the fixed nav.

---

## Header (50px)

```html
<header class="header">
  <div class="header-brand">
    <div>
      <span>dicey</span>
      <small>by Magic Eden</small>
    </div>
  </div>
  <div class="header-right">
    <div class="balance-pill">
      <div class="balance-dot"></div>
      $100.00
    </div>
    <div class="avatar">🎲</div>
  </div>
</header>
```

- **Height:** ~50px (14px padding top/bottom + content)
- **Balance pill:** rounded capsule with green dot indicator, 13px font
- **Avatar:** 34×34px circle, shows game-specific emoji
- **z-index: 10** — sits above viewport content

---

## Game Viewport (Dynamic Height)

This is the "screen" area where game visuals live. It takes all remaining vertical space.

```css
.game-viewport {
  border-radius: 8px 8px 0 0;          /* Rounded top only */
  overflow: hidden;
  position: relative;                    /* For absolute children */
  background: linear-gradient(165deg,
    #0a1a0f 0%, #0f2d14 30%, #00ab4c 100%);
  border: 1px solid #1e201e;
  border-bottom: none;                   /* Seamless join with bet-controls */
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  padding: 24px 20px 20px;
  height: calc(100dvh - 50px - 70px - 48px - 110px - 32px);
  min-height: 200px;
}
```

### Height Calculation Breakdown

```
100dvh                    Full viewport
 - 50px                   Header
 - 70px                   Bottom nav (fixed)
 - 48px                   Bet button
 - 110px                  Bet amount input + label + mode toggle
 - 32px                   Gaps and padding
 = remaining              → Game viewport gets this
```

`min-height: 200px` prevents the viewport from collapsing on very short screens.

### Viewport Internal Layout

```
.game-viewport (flex column)
├── .rakeback-row          (position: absolute, top: 10px)
│   ├── .result-pill-track (scrolling pill row)
│   └── .rakeback-pill     (pinned right)
├── .game-content          (flex: 1 — fills middle)
│   └── Game-specific UI   (dice icon, limbo multiplier, etc.)
└── .game-bottom           (frosted glass card, pinned bottom)
    └── Sliders, controls
```

### Dot Pattern Overlay

Every viewport gets a subtle dot pattern via `::after`:
```css
.game-viewport::after {
  content: '';
  position: absolute;
  inset: 0;
  background: url("data:image/svg+xml,...");  /* Tiny dots */
  pointer-events: none;
}
```

---

## Game Content (flex: 1)

The middle area of the viewport. Takes all available space between the result pills and the game-bottom card.

```css
.game-viewport .game-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  width: 100%;
}
```

This is where each game puts its unique display — dice roll number, limbo multiplier animation, coin flip visual, etc.

---

## Game Bottom Card (Frosted Glass)

A card pinned to the bottom of the viewport. Holds sliders, game options, probability controls.

```css
.game-bottom {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 340px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  background: rgba(0, 0, 0, 0.35);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 8px;
  padding: 16px;
  backdrop-filter: blur(12px);
}
```

**Key:** `max-width: 340px` keeps controls narrower than the viewport for visual breathing room.

---

## Result Pills (Absolute Overlay)

Floating row at the top of the viewport showing recent win/loss results.

```css
.rakeback-row {
  position: absolute;
  top: 10px;
  left: 0; right: 0;
  z-index: 10;
  display: flex;
  padding: 0 12px;
  gap: 6px;
  pointer-events: none;       /* Click-through */
}
```

Pills scroll horizontally with a fade mask on the left edge:
```css
.result-pill-track {
  display: flex;
  gap: 6px;
  overflow-x: auto;
  flex: 1;
  mask-image: linear-gradient(to right, transparent, black 40px);
}
```

Win pills = green, Loss pills = red. Each animates in with `pillSlideIn`:
```css
@keyframes pillSlideIn {
  from { transform: translateX(40px); opacity: 0; }
  to   { transform: translateX(0);    opacity: 1; }
}
```

---

## Bet Controls Section

Everything below the viewport. Dark surface background, seamless top border.

```css
.bet-controls {
  background: #161816;
  border-radius: 0 0 8px 8px;           /* Rounded bottom only */
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-top: none;                      /* Seamless with viewport */
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  flex-shrink: 0;                        /* Never collapse */
}
```

### Child order (top to bottom):
1. **Bet Button**
2. **Bet Amount Input**
3. **Bet Mode Toggle**
4. **Manual Settings** or **Auto Bet Settings** (one visible at a time)
5. **Settings Bar** + Settings Popup

---

## Bet Button (48px)

Full-width green action button. Fixed 48px height.

```css
.bet-button {
  width: 100%;
  height: 48px;
  background: #00ab4c;
  color: #fff;
  font-size: 17px;
  font-weight: 700;
  border-radius: 8px;
  border: none;
  box-shadow: inset 0 1px 0 rgba(255,255,255,0.15);
}
```

Has two pseudo-element layers:
- `::before` — Top-down gradient shine
- `::after` — Radial hover glow (opacity 0 → 1 on hover)

Press feedback: `transform: scale(0.97)` on `:active`.

Text changes based on mode:
- Manual: **"Bet Now"**
- Auto (idle): **"Start Autobet"**
- Auto (running): **"Stop Autobet"**

---

## Bet Amount Input

```html
<div class="input-group">
  <div class="input-label">
    <span>Bet Amount</span>
    <span class="potential-win">Win: $10.00 (2.00x)</span>
  </div>
  <div class="input-row">
    <div class="currency-tag">$</div>
    <input class="bet-input" type="number" placeholder="0.00">
    <div class="quick-amounts">
      <button class="quick-btn" data-action="half">½</button>
      <button class="quick-btn" data-action="double">2×</button>
      <button class="quick-btn" data-action="max">Max</button>
    </div>
  </div>
</div>
```

- **Label row:** "BET AMOUNT" left, green "Win: $X.XX (X.XXx)" right
- **Input row:** `$` tag | number input (18px) | ½ | 2× | Max buttons
- **Focus state:** green border + green glow ring (`box-shadow: 0 0 0 3px rgba(0, 171, 76, 0.15)`)
- **Max button:** requires double-click confirmation (first click shows "Confirm" in green, resets after 3s)

---

## Bet Mode Toggle

Segmented pill toggle between Manual Bet and Auto Bet.

```css
.bet-mode-toggle {
  display: flex;
  background: #111311;
  border: 1px solid #1e201e;
  border-radius: 8px;
  padding: 4px;
  gap: 4px;
}

.bet-mode-btn {
  flex: 1;
  padding: 9px 10px;
  font-size: 13px;
  font-weight: 600;
}

.bet-mode-btn.active {
  background: #161816;
  color: #e8eee8;
}
```

Toggling switches visibility of `#manualSettings` vs `#autobetSettings`.

---

## Manual Settings

Two stat rows showing Multiplier and Win Probability. Both are editable inputs that bidirectionally sync with the game's slider/controls.

```html
<div class="manual-settings" id="manualSettings">
  <div class="stat-row">
    <span class="stat-label">Multiplier</span>
    <div class="stat-value-wrap">
      <input class="stat-input" id="multInput" type="number">
      <span class="stat-suffix">x</span>
    </div>
  </div>
  <div class="stat-row">
    <span class="stat-label">Win Probability</span>
    <div class="stat-value-wrap">
      <input class="stat-input" id="winProbInput" type="number">
      <span class="stat-suffix">%</span>
    </div>
  </div>
</div>
```

---

## Auto Bet Settings

Full autobet control panel. Hidden by default (`style="display:none"`).

```
Number of Bets    [ input ] [10] [100] [∞]
On Win            [Reset | Increase by]  [___%]
On Loss           [Reset | Increase by]  [___%]
Stop on Profit    [$] [input]
Stop on Loss      [$] [input]
```

- **Number chips** (`data-bets="10"`, `"100"`, `"Infinity"`) set the input value
- **On Win/Loss** use a segmented toggle (`.autobet-seg`); "Increase by" reveals a percentage input
- **Stop inputs** have a `$` currency tag prefix

---

## Settings Bar

Row of icon buttons at the very bottom of bet-controls. Has `position: relative` to anchor the popup.

```html
<div class="settings-bar">
  <div class="settings-left">
    <button class="settings-btn" id="settingsBtn">⚙</button>
    <button class="settings-btn" id="infoBtn">ⓘ</button>
    <button class="settings-btn" id="chartBtn">📊</button>
  </div>
  <button class="settings-btn" id="fairnessBtn">🛡</button>
</div>
```

Icons are 16×16 SVGs. Active state turns green.

### Settings Popup

Absolutely positioned above the settings bar. Toggled by clicking the gear icon.

```css
.settings-popup {
  position: absolute;
  bottom: 52px;              /* Above the settings bar */
  left: 0;
  width: 220px;
  z-index: 30;
  background: #161816;
  border: 1px solid #1e201e;
  border-radius: 8px;
  padding: 14px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
}
```

Contains toggle switches (40×22px pills) for Sound and Turbo Mode.

---

## Bottom Nav (Fixed)

Fixed to the bottom of the screen. Stays visible while scrolling.

```css
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 420px;
  padding: 10px 16px 20px;     /* Extra bottom padding for safe area */
  background: #0e100f;
  z-index: 20;
  border-top: 1px solid #1e201e;
}
```

Three items: **Explore**, **Casino** (active), **Sports**. Each is a flex column with 20×20 SVG icon + 11px label.

---

## Design Tokens

```css
:root {
  --bg: #0a0b0a;
  --surface: #111311;
  --surface-raised: #161816;
  --border: #1e201e;
  --border-focus: #00ab4c;
  --green-primary: #00ab4c;
  --green-dark: #007a36;
  --green-gradient-start: #0a1a0f;
  --green-gradient-end: #00ab4c;
  --text-primary: #e8eee8;
  --text-secondary: #6b756b;
  --text-muted: #3d433d;
  --radius: 8px;
  --radius-sm: 8px;
  --radius-xs: 8px;
}
```

**Color hierarchy:** `--bg` (darkest) → `--surface` → `--surface-raised` (lightest dark)

**Text hierarchy:** `--text-primary` (bright) → `--text-secondary` (gray) → `--text-muted` (dim)

**Fonts:**
- UI text: `'Outfit', sans-serif`
- Numbers/code: `'JetBrains Mono', monospace`

---

## Z-Index Layers

| Layer | z-index | Element |
|-------|---------|---------|
| Settings popup | 30 | `.settings-popup` |
| Bottom nav | 20 | `.bottom-nav` |
| Header | 10 | `.header` |
| Result pills | 10 | `.rakeback-row` |
| Game bottom card | 1 | `.game-bottom` |
| Dot overlay | 0 | `.game-viewport::after` |

---

## How to Add a New Game

1. **Copy `index.html`** as your starting template
2. **Replace viewport content** — swap `.game-content` children with your game-specific display
3. **Replace `.game-bottom` content** — swap sliders/options with your game controls
4. **Rewrite `<script>`** — keep the bet input, quick amounts, mode toggle, autobet, and settings popup JS. Replace game logic functions.
5. **Update `casino.html`** — add a game card with icon, name, and "Live" tag
6. **Keep identical:** header, bottom nav, bet button, bet amount input, mode toggle, autobet panel structure, settings bar/popup

### Checklist for new games:
- [ ] App shell with 420px max-width
- [ ] Header with balance pill
- [ ] Game viewport with gradient background
- [ ] Result pill track + rakeback pill
- [ ] Game-specific content area (flex: 1)
- [ ] Game bottom frosted card with controls
- [ ] Bet button (48px, green)
- [ ] Bet amount with $, ½, 2×, Max
- [ ] Manual/Auto bet toggle
- [ ] Manual settings (multiplier + probability)
- [ ] Auto settings (bets, on win/loss, stop conditions)
- [ ] Settings bar with gear/info/chart/fairness
- [ ] Settings popup with Sound + Turbo toggles
- [ ] Bottom nav (Explore, Casino, Sports)
- [ ] `turboMode` and `soundEnabled` state variables
- [ ] Game card in casino.html
