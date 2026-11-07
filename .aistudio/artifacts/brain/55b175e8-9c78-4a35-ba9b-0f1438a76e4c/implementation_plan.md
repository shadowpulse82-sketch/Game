# Early-Game Economy & Pacing Retune

Retune creature base incomes, upgrade cost curves, and egg/slot progression so the early game delivers immediate rewards in the first 2–3 minutes, maintains a steady purchase cadence every 30–60 seconds during the first 10 minutes, reaches roughly 100K coins after 1 hour of active play, positions the Cosmic Egg as a multi-hour milestone, and preserves the player's current save.

## User Review & Critical Decisions

> [!IMPORTANT]
> Your existing save will be preserved (`SAVE_VERSION` remains unchanged). Only economy balance constants, egg drop tables, and the upgrade cost formula are updated.

- **Confirmed Decision 1 (Level 20 Upgrade Curve Transition)**: Upgrade costs grow at `1.12x` per level for Levels 1–20, then smoothly compound at `1.18x` per level from the Level 20 cost (`baseCost * 1.12^19 * 1.18^(level - 20)`) with no discontinuity at Level 21.
- **Confirmed Decision 2 (Cosmic Egg Multi-Hour Goal)**: Cosmic Egg `unlockTotalCoins` is raised to `300,000` and `baseCost` to `500,000` so it serves as a multi-hour progression milestone rather than a first-hour unlock.
- **Confirmed Decision 3 (Egg Drop Tables & Mythic Exclusivity)**:
  - **Basic Egg**: Common `60%`, Uncommon `25%`, Rare `11%`, Epic `4%` (no Legendary or Mythic).
  - **Golden Egg**: Rare `68%`, Epic `30%`, Legendary `2%`, Mythic `0%` (Legendary is rare at `2%`, and Mythic never drops from Golden Egg).
  - **Cosmic Egg**: Epic `55%`, Legendary `38%`, Mythic `7%` (Mythic creatures only come from the Cosmic Egg).

## 1. Overview & Core Concept

- **What It Does**: Multiplies all 12 creature base incomes by 10x, lowers early-level upgrade friction, and calibrates egg and slot pricing for a responsive early game that naturally decelerates after Level 20 and past the 1-hour mark.
- **Target Experience**:
  - **Minutes 0–3**: Immediate first Basic Egg hatch and several quick creature upgrades, bringing passive income from `1.0/sec` (Noodle Cat Lv.1) up to `3–8/sec` within minutes.
  - **Minutes 3–10**: One meaningful upgrade or egg hatch every 30–60 seconds as the player fills their initial 4 vault slots, unlocks Slot 5, and clears the first raids.
  - **Minutes 10–60**: Transition into Golden Eggs (`6,000` unlock, `2,500` base cost) and mid-tier Rare/Epic creatures (with a `2%` chance at a Legendary), reaching ~100K coins earned around 60 minutes of active play.
  - **Multi-Hour Endgame**: Steep `1.18x` post-Level-20 upgrade scaling and the `300,000` unlock / `500,000` base cost Cosmic Egg provide a long-term goal where Mythic creatures (`7%`) become obtainable.

## 2. User Experience & Visual Design

- **Visuals, Art & Layout Unchanged**: Zero changes to creature canvas art, raider animations, DOM layout, or mobile performance optimizations.
- **Preserved Rules**:
  - Offline earnings remain at **25% passive rate** capped at **1 hour**.
  - Raid victory bonus remains at **10 seconds of income** (scaled up to `2.0x` by raid hit combo).
  - Raid defeat money sink remains at **5% of current coins**.
  - All `try/catch` safety guards remain intact.

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: 10x Creature Base Incomes**
  - *Chosen Approach*: Scale every creature's `baseIncome` by 10x:
    - **Common**: Noodle Cat `1.0/s`, Soggy Toast `2.0/s`
    - **Uncommon**: Disco Duck `5.0/s`, Cactus Bro `9.0/s`
    - **Rare**: Cyber Frog `20.0/s`, Lava Llama `35.0/s`
    - **Epic**: Ghost Burger `80.0/s`, Moon Penguin `140.0/s`
    - **Legendary**: Golden Capy `400.0/s`, Thunder Taco `700.0/s`
    - **Mythic**: Galaxy Jelly `2,500/s`, King Pickle `5,000/s`
  - *Why*: Restores tangible whole-number coin flow from the very first creature (`1.0/s` at Lv.1, `2.0/s` at Lv.10) instead of fractional `0.1/s` trickles.

- **Decision 2: Piecewise Smooth Upgrade Cost Formula (1.12x -> 1.18x)**
  - *Chosen Approach*: Set `UPGRADE_BASE_MULTIPLIER = 12`, `UPGRADE_EARLY_EXPONENT = 1.12` (Levels 1–20), and `UPGRADE_EXPONENT = 1.18` (Levels 21–100).
  - *Why*: At `12 * baseIncome`, upgrading a Lv.1 Noodle Cat costs `12` coins (~12 seconds of its own income) and Lv.10 costs `33` coins, enabling brisk early progression. Above Level 20, the `1.18x` exponent outpaces the `1.08x` income growth steeply enough to prevent runaway inflation.

- **Decision 3: Tuned Egg Costs, Drop Tables, Slots & Defense**
  - *Chosen Approach*:
    - **Basic Egg**: `baseCost: 50`, `costMult: 1.15`, `ownedCostMult: 1.18`; Odds: Common `60%`, Uncommon `25%`, Rare `11%`, Epic `4%`.
    - **Golden Egg**: `unlockTotalCoins: 6,000`, `baseCost: 2,500`, `costMult: 1.18`, `ownedCostMult: 1.15`; Odds: Rare `68%`, Epic `30%`, Legendary `2%` (`0%` Mythic).
    - **Cosmic Egg**: `unlockTotalCoins: 300,000`, `baseCost: 500,000`, `costMult: 1.18`, `ownedCostMult: 1.15`; Odds: Epic `55%`, Legendary `38%`, Mythic `7%` (exclusive source of Mythic creatures).
    - **Vault Slots**: `BASE_SLOT_COST: 250`, `SLOT_COST_MULTIPLIER: 2.15` (Slot 5 costs `250`, Slot 6 costs `538`, Slot 7 costs `1,156`).
    - **Defense Upgrades**: `DEFENSE_BASE_COST: 200`, `DEFENSE_COST_MULTIPLIER: 2.0` (Lv.1 defense affordable right around the first few raids).

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

```
┌───────────────────────────────────────────────────────────────────┐
│                         CONFIG Economy Constants                  │
│  • CREATURES[0..11].baseIncome (×10)                              │
│  • UPGRADE_BASE_MULTIPLIER (12)                                   │
│  • UPGRADE_EARLY_LEVEL_CAP (20), UPGRADE_EARLY_EXPONENT (1.12)    │
│  • UPGRADE_EXPONENT (1.18)                                        │
│  • EGG_TYPES:                                                     │
│      - Basic (cost 50, Common/Uncommon/Rare/Epic)                 │
│      - Golden (unlock 6K, cost 2.5K, Rare 68%/Epic 30%/Leg 2%)    │
│      - Cosmic (unlock 300K, cost 500K, Epic 55%/Leg 38%/Myth 7%)  │
└─────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
┌───────────────────────────────────────────────────────────────────┐
│                    getCreatureUpgradeCost(id, lvl)                │
│  • If lvl <= 20:                                                  │
│      round(12 * baseIncome * 1.12^(lvl - 1))                      │
│  • If lvl > 20:                                                   │
│      round(12 * baseIncome * 1.12^19 * 1.18^(lvl - 20))           │
└─────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
┌───────────────────────────────────────────────────────────────────┐
│                Existing Save & Game Loop Integration              │
│  • SAVE_VERSION = 3 (unchanged — existing save loads intact)      │
│  • HUD & Upgrade Modals dynamically compute new incomes & costs   │
└───────────────────────────────────────────────────────────────────┘
```

- **Data Model & State**: `state.ownedCreatures`, `state.coins`, `state.eggHatches`, and `state.slotsUnlocked` are read directly from `localStorage` without version mismatch resets, immediately applying the new income and cost curves to the player's existing progress.
