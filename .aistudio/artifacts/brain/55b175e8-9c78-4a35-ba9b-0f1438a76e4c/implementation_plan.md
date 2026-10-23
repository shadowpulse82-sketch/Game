# Meme Vault Defense

**Meme Vault Defense** is a self-contained, neon-candy cartoon idle game built entirely inside a single vanilla HTML/CSS/JavaScript application where players tap a glowing cyber-vault, hatch quirky meme creatures from eggs, and level up their squad to generate massive passive coin income.

## User Review & Critical Decisions

> [!IMPORTANT]
> The following core gameplay behaviors and revisions are locked into the design specification:

- **Full-Vault Hatch & Auto-Replace Rule**: Every hatched creature is immediately recorded in `discoveredCreatures` (even if converted). Then, the hatch resolves in this exact order:
  1. **Already Owned**: If the player already owns the hatched creature in a slot, it levels up by `+1`. If that owned creature is already at Max Level (`10`), the hatched creature is converted into coins instead.
  2. **Open Slot Available**: Else if an unlocked slot is empty, the new creature is placed in the next open slot at Level 1.
  3. **Vault Full (Smart Auto-Replace vs. Convert)**: Else (all unlocked slots are occupied), find the owned creature with the **lowest current income per second** (`baseIncome × level`).
     - If the newly hatched creature's Level 1 income (`baseIncome × 1`) is **strictly higher** than that lowest owned creature's current income/sec, the new creature **takes its slot** and the replaced creature is converted to coins.
     - Otherwise, the newly hatched creature is converted to coins.
  - **Conversion Formula & Banner**: `Converted Coins = 60 × baseIncome × creatureLevel`. A prominent notification banner displays the exact outcome, e.g., `"Ghost Burger replaced Soggy Toast (+120 coins)"` or `"Soggy Toast converted (+120 coins)"`.
- **Permanent Slot Assignment**: Players cannot manually move, swap, or dismiss creatures; smart replacement happens automatically during hatching when a stronger creature arrives.
- **Bottom "Upgrade" Action Button**: Scans all owned creatures below Level 10 to find the lowest-cost upgrade. If that cheapest upgrade is currently affordable, the button pulses and displays the target creature's emoji and upgrade cost (tapping it immediately upgrades that creature). When no upgrade is affordable (or when the vault has no upgradeable creatures / all owned creatures are Level 10), the button appears greyed out with the text `"Nothing to upgrade"`.
- **Expanded Validated Save State**: Persistent state tracks and validates `coins`, `totalCoinsEarned` (all coins ever earned across passive income, vault taps, and conversion bonuses), `hatchCount`, `slotsUnlocked`, `ownedCreatures`, `discoveredCreatures`, `muted`, and `lastSavedAt`.
- **Clock-Safe Offline Earnings**: Elapsed offline time is clamped to `Math.max(0, Math.min(elapsedSeconds, 7200))` (between `0` and `2 hours`), ensuring a backward or forward system clock adjustment never produces negative or unbounded offline payouts.

---

## 1. Overview & Core Concept

- **What It Does**: Players start with 120 coins and a glowing cyan Vault surrounded by 4 unlocked creature plates (expandable up to 12 slots). Players can tap the Vault directly for active coins (`1 + 5% of income/sec`), hatch Basic Eggs with weighted rarity odds across 12 original emoji creatures, and level up creatures up to Level 10 to multiply their passive coin generation.
- **Target Audience / Persona**: Casual and idle/incremental game players on both mobile portrait (`360×640`) and desktop landscape (`1280×720+`) who enjoy tactile feedback, juicy animations, gacha collection loops, and satisfying number progression.
- **Key Value**: Instant zero-dependency load time, zero scroll lock layout across any viewport aspect ratio, synthesized WebAudio sound effects, smooth rolling number counters, and automatic offline earnings calculation.

---

## 2. User Experience & Visual Design

### Key User Flows
1. **First Launch & Active Tapping**: The player enters with 120 coins. The **Hatch Egg** button (`50 coins`) pulses with a soft affordability glow. Tapping the central cyan Vault triggers a squash-and-stretch bounce, spawns a floating `+N` number and flying gold coin particle toward the top counter, increments both `coins` and `totalCoinsEarned`, and plays a synthesized pop/coin sound.
2. **Egg Hatching Sequence**: Tapping **Hatch** deducts the current egg cost (`50 × 1.12^hatchCount`), increments `hatchCount`, and opens the **Hatch Theatre Overlay**:
   - An egg appears in the center, shakes with increasing intensity, cracks, flashes bright white, and reveals the creature with a springy scale-overshoot pop and fanfare audio.
   - The hatched creature ID is added to `discoveredCreatures` (if not already present).
   - Resolution follows the **Full-Vault Hatch Rule**: leveling up a duplicate, claiming an open slot, replacing the weakest owned creature if the newcomer's Lv.1 income beats its current income/sec, or converting to `60 × baseIncome × level` bonus coins with a clear banner (`"Ghost Burger replaced Soggy Toast (+120 coins)"` or `"Soggy Toast converted (+120 coins)"`).
3. **Creature Inspection & Upgrading**:
   - **Direct Slot Tap**: Tapping any occupied creature plate squashes/stretches the emoji and opens a compact **Creature Upgrade Popup** displaying its name, rarity badge, current level (`1–10`), current income/sec, next-level income preview, and an **Upgrade** button (`20 × baseIncome × 1.6^level`).
   - **Quick Upgrade Button**: The bottom bar **Upgrade** button checks the lowest-cost upgrade among owned creatures below Level 10. When affordable, it pulses and shows the creature emoji + cost; when nothing is affordable or all owned creatures are Level 10, it renders greyed out with `"Nothing to upgrade"`.
4. **Expanding the Vault**: In the vault grid, the next locked slot features a **Buy Slot** button (starting at `500` for Slot 5, multiplying by `×2.5` for each subsequent slot up to Slot 12).
5. **Offline Welcome Back Flow**: Returning to the game after being away calculates elapsed time clamped to `[0, 7200]` seconds multiplied by passive income per second, presenting a **"Welcome back! You earned X"** modal with a shiny **Collect** button that adds the payout to both `coins` and `totalCoinsEarned`.

### Visual Identity & Theme
- **Aesthetic Direction**: **Neon Candy Cartoon Arcade** — tactile 3D glossy buttons with top shine highlights and darker bottom ledges, luminous rarity rings, and playful spring physics set against a deep cosmic indigo backdrop.
- **Color Palette & Mood**:
  - **Background Canvas (60%)**: Deep indigo vertical gradient (`#1B1040` to `#3A1C71`) overlaid with a hardware-accelerated canvas of slowly drifting starfield particles.
  - **Structural Surfaces & Vault (30%)**: Translucent cosmic indigo glass panels (`rgba(27, 16, 64, 0.78)`) with crisp borders, anchored by the central Vault glowing in electric cyan (`#35E0FF`).
  - **Primary Accents & Currency (10%)**: Gold coin highlights (`#FFD34D`) and vibrant candy pink action accents (`#FF4FA3`).
  - **Rarity Ring Spectrum**:
    - *Common*: `#B0B8C5` (Silver Mist)
    - *Uncommon*: `#5CFF9D` (Mint Neon)
    - *Rare*: `#35A7FF` (Sky Bolt)
    - *Epic*: `#B15CFF` (Amethyst Glow)
    - *Legendary*: `#FFB020` (Solar Gold)
    - *Mythic*: Animated conic/linear rainbow spectrum glow
- **Typography & Hierarchy**:
  - Bold, rounded system font stack (`'Trebuchet MS', 'Segoe UI', system-ui, sans-serif`) with `font-variant-numeric: tabular-nums` so rapidly rolling counters never jitter horizontally.
  - Big coin readout in the top bar with smooth interpolation and suffix formatting (`1.2K`, `3.4M`, `5.6B`, `7.8T`).
- **Component Styling & Layout**:
  - **Zero-Scroll Responsive Viewport (`100dvh`)**: Uses a locked 3-zone vertical flex/grid architecture (Top HUD Bar → Middle Vault & Creature Arena → Bottom Control Dock) that adapts seamlessly between `360×640` portrait phones and `1280×720` landscape desktops.
  - **Ergonomic Touch Targets**: All interactive buttons and creature plates are at least `48px` tall, with `touch-action: manipulation`, disabled text selection, and disabled context menus.

### Interactive Feedback & Motion
- **Idle Bobbing & Squash/Stretch**: Every seated creature gently bobs on a sine wave offset by its slot index and performs a snappy squash-and-stretch transform when tapped or leveled up.
- **Affordability Pulse**: Any purchasable button (**Hatch**, **Upgrade**, **Buy Slot**) that the player can currently afford emits a soft rhythmic glow; unaffordable buttons shift to a muted desaturated state.
- **Particle Engine (Capped at 150)**: High-DPI `<canvas>` layer renders drifting background stars, flying gold coins that arc toward the top coin counter, and floating `+N` income numbers that drift upward and fade out.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: Single-File Zero-Dependency Architecture (`index.html`)**
  - *Chosen Approach*: Consolidate all HTML structure, CSS animations/styles, WebAudio synthesis, particle canvas rendering, and game state logic into one self-contained `index.html` file, removing template scaffolding files (`src/`, TypeScript/React build dependencies) as requested.
  - *Why*: Guarantees zero external network failures (no external fonts, images, or audio files), instant startup, and strict compliance with the single-file vanilla specification.
- **Decision 2: Centralized `CONFIG` Object & Delta-Time Game Loop**
  - *Chosen Approach*: Define every balance constant, rarity probability, creature definition, cost multiplier, conversion multiplier (`60`), and particle cap in a single top-level `CONFIG` constant, driven by a `requestAnimationFrame` loop with delta time capped at `100ms` and automatic pause on `document.hidden`.
  - *Why*: Prevents spiral-of-death frame jumps when switching tabs, guarantees deterministic math without `NaN` or negative coin bugs, and makes balance auditing effortless.
- **Decision 3: Fault-Tolerant `localStorage` Persistence & Strict Schema Validation**
  - *Chosen Approach*: Wrap all save/load serialization in `try/catch` blocks with strict per-field validation (`coins`, `totalCoinsEarned`, `hatchCount`, `slotsUnlocked`, `ownedCreatures`, `discoveredCreatures`, `muted`, `lastSavedAt`), autosaving every 10 seconds and on `visibilitychange` / `pagehide`.
  - *Why*: Ensures private browsing modes, clock tampering, or corrupted save data never crash the game or produce `NaN` values.

---

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

### Architecture & Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       SINGLE-FILE APP (index.html)                      │
├─────────────────────────────────────────────────────────────────────────┤
│  CONFIG (Balance, Odds, 12 Creatures, Costs, Conversion=60x, Caps)      │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         GAME STATE & PERSISTENCE                        │
│  • State: coins, displayedCoins, totalCoinsEarned, hatchCount,          │
│           slotsUnlocked (4..12), ownedCreatures[],                      │
│           discoveredCreatures[], muted, lastSavedAt                     │
│  • Persistence: Safe localStorage load/save (10s interval + tab hide)   │
└───────┬───────────────────────────┬─────────────────────────────┬───────┘
        │                           │                             │
        ▼                           ▼                             ▼
┌──────────────────────┐  ┌──────────────────────────┐  ┌─────────────────┐
│   RAF GAME ENGINE    │  │   DOM UI & INTERACTION   │  │ WEBAUDIO SYNTH  │
│ • dt capped ≤ 100ms  │  │ • Top Bar (Coins, /sec,  │  │ • Lazy init on  │
│ • Passive income/sec │  │   Mute toggle, Banner)   │  │   first pointer │
│ • Smooth coin roller │  │ • Middle Arena: Glowing  │  │ • Pop, Coin,    │
│ • Canvas Starfield & │  │   Cyan Vault + 12-Slot   │  │   Hatch Shake,  │
│   Particle Pool      │  │   Glossy Plate Grid      │  │   Fanfare,      │
│   (Max 150 active)   │  │ • Bottom Bar: Hatch,     │  │   Level-Up      │
│                      │  │   Quick Upgrade, Locked  │  │                 │
│                      │  │   Defense & Book         │  │                 │
│                      │  │ • Modals: Hatch Reveal,  │  │                 │
│                      │  │   Slot Upgrade, Offline  │  │                 │
└──────────────────────┘  └──────────────────────────┘  └─────────────────┘
```

### Data Model & State
- **Creature Catalog (`CONFIG.CREATURES`)**:
  - **Common (60%)**: Noodle Cat 🐱 (`1/s`), Soggy Toast 🍞 (`2/s`)
  - **Uncommon (25%)**: Disco Duck 🦆 (`5/s`), Cactus Bro 🌵 (`9/s`)
  - **Rare (10%)**: Cyber Frog 🐸 (`20/s`), Lava Llama 🦙 (`35/s`)
  - **Epic (4%)**: Ghost Burger 👻 (`80/s`), Moon Penguin 🐧 (`140/s`)
  - **Legendary (0.9%)**: Golden Capy 🦫 (`400/s`), Thunder Taco 🌮 (`700/s`)
  - **Mythic (0.1%)**: Galaxy Jelly 🪼 (`2500/s`), King Pickle 🥒 (`5000/s`)
- **Formulas**:
  - **Creature Income**: `baseIncome * level` (Max Level = `10`)
  - **Creature Upgrade Cost**: `Math.round(20 * baseIncome * Math.pow(1.6, level))`
  - **Egg Cost**: `Math.round(50 * Math.pow(1.12, hatchCount))`
  - **Slot Cost (Slot `n` where `5 <= n <= 12`)**: `Math.round(500 * Math.pow(2.5, n - 5))`
  - **Converted Creature Coins**: `60 * baseIncome * creatureLevel`
  - **Vault Tap Reward**: `1 + 0.05 * totalIncomePerSecond`
  - **Offline Earnings**: `totalIncomePerSecond * Math.max(0, Math.min(elapsedSeconds, 7200))`
- **Validated State Schema**:
  - `coins`: non-negative finite number (default `120`)
  - `totalCoinsEarned`: non-negative finite number `>= coins` (default `120`)
  - `hatchCount`: non-negative integer (default `0`)
  - `slotsUnlocked`: integer clamped `4..12` (default `4`)
  - `ownedCreatures`: array of `{ id, level }` validated against `CONFIG.CREATURES`, length `<= slotsUnlocked`, `level` clamped `1..10`, deduplicated by `id`
  - `discoveredCreatures`: array of valid creature `id` strings (automatically includes all currently `ownedCreatures` IDs)
  - `muted`: boolean (default `false`)
  - `lastSavedAt`: valid timestamp `<= Date.now()`

### Interactive Component & State Mapping
- **Central Vault Button**: On `pointerdown`, calculates tap reward, adds to `state.coins` and `state.totalCoinsEarned`, triggers vault squash-stretch CSS animation, spawns a floating `+N` text particle and 1–3 coin particles flying to the top coin badge, and plays the WebAudio pop + coin ding.
- **Hatch Resolution & Banner**:
  - Adds hatched creature ID to `state.discoveredCreatures`.
  - Evaluates duplicate level-up / Lv.10 conversion → open slot placement → lowest-income comparison & slot replacement (`"<New> replaced <Old> (+X coins)"`) or newcomer conversion (`"<New> converted (+X coins)"`).
  - Conversion coins are added to both `state.coins` and `state.totalCoinsEarned`.
- **12-Slot Responsive Grid**:
  - *Occupied Slot*: Renders creature emoji (`≥56px`), glossy radial plate, rarity ring glow, level badge (`Lv.X`), and income/sec readout. Tapping opens the **Creature Upgrade Popup** for that slot.
  - *Next Locked Slot (`index === state.slotsUnlocked` and `< 12`)*: Renders the **Buy Slot** interactive plate showing the exact unlock cost and pulsing when affordable.
  - *Remaining Locked Slots*: Rendered as dimmed locked plates showing slot number.
- **Bottom Action Dock**:
  - **Hatch Button**: Displays current egg price (`50 × 1.12^hatchCount`), pulses when `coins >= eggCost`, and launches the animated Egg Hatch sequence on tap.
  - **Upgrade Button**: Finds the owned creature below Level 10 with the lowest upgrade cost. If found and `state.coins >= cost`, pulses and displays `<emoji> Upgrade (<cost>)`; otherwise renders greyed out with `"Nothing to upgrade"`.
  - **Defense & Book Buttons**: Styled as locked preview buttons with padlock icons (`🔒 Defense`, `🔒 Book`) and a gentle shake + floating `"Locked for now!"` feedback when tapped.
