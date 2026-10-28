# Meme Vault Defense — Original Drawn Creature Art (First 6 Creatures)

This update replaces the emoji creature art for the first six creatures (**Noodle Cat**, **Soggy Toast**, **Disco Duck**, **Cactus Bro**, **Cyber Frog**, and **Lava Llama**) with original, silhouette-distinct 2D canvas illustrations while keeping gameplay, economy numbers, saving, and layout completely unchanged.

## User Review & Critical Decisions

> [!IMPORTANT]
> All confirmed design choices from our clarification step are incorporated below. The remaining six creatures (**Ghost Burger**, **Moon Penguin**, **Golden Capy**, **Thunder Taco**, **Galaxy Jelly**, and **King Pickle**) will continue using their emoji fallback until a future step.

- **Confirmed Decision 1 — Pre-Rendered Open & Closed-Eye Canvases**: Each of the first six creatures will be pre-rendered once at startup (scaled by `devicePixelRatio`) into two cached offscreen canvases (`open` and `blink` frames, plus a `dataURL` pair for instant `<img>` or `<canvas>` swapping), and each creature instance will swap to its closed-eye frame for ~130ms every 3 to 5 seconds.
- **Confirmed Decision 2 — Art Placement vs. Compact Button Text**: Drawn creature art will appear on the **vault slot plates**, in the **egg hatch reveal popup**, and in the **creature upgrade/rescue popup**, while compact inline button labels (such as the bottom Quick Upgrade button) keep their lightweight emoji indicator.

---

## 1. Overview & Core Concept

- **What It Does**: Introduces bespoke vector/canvas character illustrations for the Common, Uncommon, and Rare creature tiers in a unified neon-candy cartoon style without external image files.
- **Target Audience / Persona**: Mobile and desktop idle-game players who want expressive, collectible characters with unmistakable silhouettes at `56px` and lively personality animations.
- **Key Value**: Gives every early- and mid-game creature a distinct silhouette, custom idle motion, and periodic eye blink at zero runtime draw overhead.

---

## 2. User Experience & Visual Design

- **Unified Art Style Rules**:
  - **Outline**: Thick `#1B1040` outer stroke (`~4px` on a `100×100` logical coordinate space, scaled by `devicePixelRatio`).
  - **Facial Features**: Big glossy dark eyes with dual white specular highlights (or curved happy/sleepy closed eyelids on the blink frame), soft pink/coral oval blush marks, and a clean expressive mouth.
  - **Silhouette-First Character Designs (First 6 Only)**:
    1. **Noodle Cat (`noodle_cat`)**: Pointed triangular cat ears on a rounded head, compact sitting feline body, long high-arching S-curved tail shaped like a chopstick on the right, and a flowing wavy golden ramen-noodle scarf with hanging noodle strands.
    2. **Soggy Toast (`soggy_toast`)**: Wide mushroom-domed top crust over a square bread slice silhouette, droopy half-lidded sad eyes with a tiny tear gleam, wavy frown, and a melting golden pat of butter dripping down the top-right crust.
    3. **Disco Duck (`disco_duck`)**: Asymmetric side-profile duck silhouette facing right with a protruding orange bill, upturned tail feathers, flat webbed feet, cool angled magenta-rimmed star/aviator sunglasses, and a shiny faceted silver disco-ball hat perched on top.
    4. **Cactus Bro (`cactus_bro`)**: Tall vertical saguaro column with two wide, asymmetrical L-shaped flexing arms raised on both sides, cool dark shades, a toothy grin, subtle vertical rib lines, and a bright pink five-petal flower crown on top.
    5. **Cyber Frog (`cyber_frog`)**: Low, ultra-wide crouching amphibian silhouette with two large hemispherical eye domes protruding high above the head, neon green skin, a glowing cyan wraparound holographic visor, and pulsing cyan circuit traces on its belly and cheeks.
    6. **Lava Llama (`lava_llama`)**: Tall, slender vertical neck and long upright banana-shaped ears rising above an angled snout and fluffy shoulders, warm volcanic orange coat, jagged glowing yellow-red magma hair tuft on the crown, and glowing ember cheek spots.
- **Creature-Specific Idle Motions & Blinking**:
  - Each drawn creature uses a dedicated GPU-composited CSS `@keyframes` transform animation (`idle-sway` for Noodle Cat, `idle-wobble` for Soggy Toast, `idle-hop` for Disco Duck, `idle-flex` for Cactus Bro, `idle-float` for Cyber Frog, and `idle-prance` for Lava Llama).
  - Every 3 to 5 seconds (randomized per slot/modal instance), the creature briefly swaps to its pre-rendered closed-eye blink frame (`~140ms`) unless kidnapped (where animations pause and the plate is greyed out).

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: Offscreen Canvas Pre-Rendering + Cached High-DPI Bitmaps**
  - *Chosen Approach*: Render each of the 6 creatures (`open` and `blink` variants) once onto offscreen canvases scaled by `Math.min(window.devicePixelRatio || 1, 2)` at `112×112` logical pixels, and blit/display them into lightweight `<canvas>` or cached bitmap elements inside slots and modals.
  - *Why*: Complex bezier paths, outlines, and specular highlights are computed only once at initialization (and on DPI change), keeping the 60fps `requestAnimationFrame` loop completely free of per-frame vector path drawing.
  - *Alternatives Considered*: Drawing 12 creature vector paths every frame on a shared canvas was rejected to avoid mobile GPU/CPU overhead.
- **Decision 2: Fallback Strategy for Creatures 7–12**
  - *Chosen Approach*: Keep a clean lookup `DRAWN_CREATURE_RENDERERS` containing the first 6 creature IDs (`noodle_cat`, `soggy_toast`, `disco_duck`, `cactus_bro`, `cyber_frog`, `lava_llama`). Any creature ID not in this map automatically renders its existing `.creature-emoji` span.
  - *Why*: Guarantees zero regressions for Epic, Legendary, and Mythic creatures until their bespoke art is added in a later step.

---

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

```
┌──────────────────────────────────────────────────────────────────────┐
│                  OFFSCREEN CREATURE ART CACHE (Init)                 │
│  For each id in [noodle_cat, soggy_toast, disco_duck,                │
│                  cactus_bro, cyber_frog, lava_llama]:                │
│    ┌─────────────────────────┐      ┌─────────────────────────┐      │
│    │ Offscreen Canvas (Open) │      │ Offscreen Canvas (Blink)│      │
│    │ 112×112 × dpr           │      │ 112×112 × dpr           │      │
│    └────────────┬────────────┘      └────────────┬────────────┘      │
└─────────────────┼────────────────────────────────┼───────────────────┘
                  │                                │
       ┌──────────┴────────────────────────────────┴──────────┐
       ▼                          ▼                           ▼
┌──────────────────┐    ┌────────────────────┐     ┌────────────────────┐
│ 4×3 Vault Plates │    │ Hatch Reveal Modal │     │ Upgrade/Rescue     │
│ - Per-creature   │    │ - Drawn art inside │     │ Popup Modal        │
│   CSS idle class │    │   rarity ring      │     │ - Drawn art inside │
│ - 3–5s blink     │    │ - Idle + blink     │     │   modal plate      │
│   scheduler      │    │   active           │     │ - Pauses if kidnap │
└──────────────────┘    └────────────────────┘     └────────────────────┘
```

- **Data Model & State**:
  - `CONFIG`, `state`, and `localStorage` (`saveVersion: 2`) remain 100% untouched.
  - A new `creatureArtCache` dictionary maps each of the 6 creature IDs to `{ openUrl, blinkUrl, idleClass }`.
  - A lightweight blink scheduler updates visible drawn creature `<img>` elements every frame or timer tick, flipping `src` from `openUrl` to `blinkUrl` for `140ms` when an instance's individual `nextBlinkAt` timestamp is reached, then scheduling the next blink randomly `3.0s–5.0s` later.
- **Interactive Component & State Mapping**:
  - **Vault Slot Plates (`renderSlotsGrid`)**: If `CREATURE_ART_SPECS[owned.id]` exists, renders `<img class="creature-art-img idle-<motion>" ...>` inside `.slot-plate`; if kidnapped, disables the idle animation and displays the closed-eye frame under the plate's greyscale filter. Otherwise falls back to `<span class="creature-emoji">`.
  - **Hatch Reveal (`startHatchSequence`)**: Uses the same helper (`getCreatureVisualHTML(creatureId, opts)`) so hatched creatures pop out with their full drawn artwork and idle motion inside the rarity ring.
  - **Upgrade & Rescue Modal (`renderUpgradeModalContent`)**: Replaces `#upgrade-modal-emoji` content with the drawn creature artwork (or emoji fallback for creatures 7–12) while keeping milestone star badges and rarity rings intact.
