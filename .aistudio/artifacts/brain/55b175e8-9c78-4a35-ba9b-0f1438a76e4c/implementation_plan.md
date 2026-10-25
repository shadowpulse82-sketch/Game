# Meme Vault Defense — Economy Rebalance & Multi-Egg Progression Plan

This update rebalances the **Meme Vault Defense** economy for deep long-term progression up to Level 100, introduces 3 tiered Egg types (**Basic**, **Golden**, **Cosmic**), scales **Defense** upgrades to Level 10, extends big-number formatting through Decillions (`Dc`) and scientific notation, enforces `saveVersion = 2` with a Settings/Reset modal, and adds a rapid-tap **Debug Tools** panel—all while preserving the existing look, layout, and single-file `index.html` architecture.

## User Review & Critical Decisions

> [!IMPORTANT]
> The following design choices were confirmed during clarification and are locked into the specification:

- **Milestone Gold Star Badge (`★N`)**: Each creature plate displays a compact gold star badge showing the count of 10-level milestones reached (`★1` at Lv. 10, `★2` at Lv. 20, ..., `★10` at Lv. 100) alongside the level badge, with the `MAX` label appearing strictly at Level `100`.
- **Defense Upgrade Cost Curve (`600 × 2^currentLevel`)**: All 3 defense upgrades start unowned at Level `0` and scale up to Level `10`. Unlocking Level 1 (`currentLevel = 0`) costs `600` (`600 × 2^0`), upgrading Lv. 1 → Lv. 2 costs `1,200` (`600 × 2^1`), up to Lv. 9 → Lv. 10 (`600 × 2^9 = 307,200`).

---

## 1. Overview & Economy Rebalance Summary

- **Creature Progression (Max Level 100)**:
  - **Income Formula**: `baseIncome × level × 2^(floor(level / 10))` — every 10 levels doubles the creature's output on top of linear level scaling, making milestone levels (`10, 20, 30, ..., 100`) feel exciting and impactful.
  - **Upgrade Cost Formula**: `Math.round(20 × baseIncome × 1.15^level)` — smooth exponential growth across 100 levels.
  - **Converted Creature Payout**: Full-vault replacements/conversions and duplicates of Level 100 creatures grant **60 seconds of that creature's current income** (`60 × getCreatureIncome(id, level)`).
- **Three Egg Tiers & Unlock Goals**:
  - **Basic Egg**: Cost `50 × 1.15^basicHatches`. Always unlocked. Odds: Common `60%`, Uncommon `25%`, Rare `10%`, Epic `4%`, Legendary `0.9%`, Mythic `0.1%`.
  - **Golden Egg**: Cost `5,000 × 1.12^goldenHatches`. Unlocks at `2,000` `totalCoinsEarned`. Odds: Rare `60%`, Epic `30%`, Legendary `9%`, Mythic `1%`.
  - **Cosmic Egg**: Cost `500,000 × 1.15^cosmicHatches`. Unlocks at `200,000` `totalCoinsEarned`. Odds: Epic `55%`, Legendary `38%`, Mythic `7%`.
- **Offline Earnings**: Awards `50%` of passive income × time away (`0.5 × incomePerSec × elapsedSec`), clamped between `0` and `2 hours` (`7,200s`).

---

## 2. User Experience & Interface Additions

### Compact Egg Selector Strip
- Positioned directly above the bottom bar, a compact 3-pill selector (`🥚 Basic`, `✨ Golden`, `🌌 Cosmic`) lets the player choose which egg the **Hatch** button buys.
- Locked eggs appear greyed out with their unlock goal (`🔒 2K Total` / `🔒 200K Total`). Once `state.totalCoinsEarned` reaches the threshold, the egg unlocks automatically.

### Creature Plate Milestone Stars (`★N`)
- When a creature reaches Level `10` or higher (`Math.floor(level / 10) >= 1`), a gleaming gold star badge (`★1` through `★10`) appears on the top-left of its circular plate.
- The top-right badge displays `Lv.X` for levels `1–99` and `MAX` only at Level `100`.

### Settings Gear (`⚙️`) & Reset Progress Confirmation
- A **Gear (`⚙️`)** button next to the Mute button in the top bar opens a compact **Settings Popup** showing lifetime stats and a **Reset Progress** button.
- Tapping **Reset Progress** reveals an inline confirmation prompt (`"Confirm Reset — Start Fresh (120 Coins)"` / `"Cancel"`) so players never wipe their vault accidentally.

### Debug Tools Panel (`const DEBUG_TOOLS = true`)
- Tapping the top-bar coin counter **5 times quickly** (within `1.5s`) when `DEBUG_TOOLS === true` opens the **Debug Tools Popup**:
  - **+1M Coins**: Instantly adds `1,000,000` coins to `coins` and `totalCoinsEarned`.
  - **Skip 1 Hour of Income**: Instantly grants `3,600 × incomePerSecond` coins.
  - **Speed ×10 Toggle**: Multiplies the game loop delta time (`dtSec`) by `10×` for rapid testing of passive income, raid timers, and kidnap countdowns.

---

## 3. Technical Architecture & Data Strategy *(Technical Reference)*

### Architecture & Economy Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SINGLE-FILE APP (index.html)                         │
│  const DEBUG_TOOLS = true;                                              │
│  const CONFIG = { SAVE_VERSION: 2, MAX_CREATURE_LEVEL: 100, ... }       │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 SAVE STATE (saveVersion = 2 Validation)                 │
│  • If parsed.saveVersion !== 2 -> Ignore old save, start fresh (120c)   │
│  • Tracks: coins, totalCoinsEarned, eggHatches {basic,golden,cosmic},   │
│            selectedEggType, slotsUnlocked, ownedCreatures[],            │
│            discoveredCreatures[], defenseUpgrades {1..10},              │
│            raidNumber, raidTimerSec, muted, lastSavedAt                 │
└───────┬───────────────────────────┬─────────────────────────────┬───────┘
        │                           │                             │
        ▼                           ▼                             ▼
┌──────────────────────┐  ┌──────────────────────────┐  ┌─────────────────┐
│   ECONOMY FORMULAS   │  │   3-TIER EGG HATCHING    │  │ DEFENSE (Lv1-10)│
│ • Income:            │  │ • Basic:  50 × 1.15^n    │  │ • Cost:         │
│   base×lvl×2^[lvl/10]│  │ • Golden: 5K × 1.12^n    │  │   600 × 2^lvl   │
│ • Upgrade:           │  │   (Unlocks @ 2K earned)  │  │ • Dog: 4.0→1.2s │
│   20×base×1.15^lvl   │  │ • Cosmic: 500K × 1.15^n  │  │ • Laser: 5%/lvl │
│ • Convert:           │  │   (Unlocks @ 200K earned)│  │ • Door: +0.5s/l │
│   60s current income │  │                          │  │                 │
└──────────────────────┘  └──────────────────────────┘  └─────────────────┘
```

### Extended Number Formatting Specification
- Suffix array in `CONFIG.NUMBER_SUFFIXES`:
  `['', 'K', 'M', 'B', 'T', 'Qa', 'Qi', 'Sx', 'Sp', 'Oc', 'No', 'Dc']`
  - Covers `10^3` (`K`) through `10^33` (`Dc`).
  - For values `>= 10^36` (`1e36`), formats cleanly in scientific notation with 2 decimal places (e.g., `1.23e36`).

### Defense Scaling Table (`Levels 1–10`)
- **Upgrade Cost**: `600 × 2^(currentLevel)` (`Lv.0→1: 600`, `Lv.1→2: 1.2K`, ..., `Lv.9→10: 307.2K`).
- **Guard Dog 🐕**: Interval `4.0 - (level - 1) × (2.8 / 9)` seconds (`4.00s` at Lv. 1 linearly down to `1.20s` at Lv. 10).
- **Laser Fence ⚡**: Slows raiders by `5% × level` (`5%` at Lv. 1 up to `50%` at Lv. 10).
- **Vault Door 🚪**: Adds `0.5s × level` before a raider at the vault can steal (`+0.5s` at Lv. 1 up to `+5.0s` at Lv. 10).
