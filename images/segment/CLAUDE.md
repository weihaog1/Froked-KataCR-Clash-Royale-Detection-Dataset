# images/segment/ - Sprite Cutout Dataset

This directory is the single most important data source in the CS175 "The Elixir Optimizers" project (UC Irvine, Winter 2026). It feeds the synthetic data generator that creates all training images for the YOLOv8 object detection model. Without these sprites, there is no training data and no detection model.

## Project Context

This repository is a fork of the KataCR Clash Royale Detection Dataset (https://github.com/wty-yy/Clash-Royale-Detection-Dataset). The Elixir Optimizers project builds an AI agent that plays Clash Royale by detecting in-game entities with a YOLOv8 model trained exclusively on synthetic data. The synthetic data generator (located at `Project/cr-object-detection/src/generation/` in the main project) composites these sprite cutouts onto arena backgrounds to produce labeled training images on-the-fly.

## Contents

- **153 class subdirectories** - each contains transparent PNG cutouts of a specific Clash Royale entity class
- **1 `backgrounds/` directory** - 27 arena background JPGs (background01.jpg through background27.jpg, numbered with gaps) plus 1 utility PNG (`red_bound.png`)
- **4,626 sprite cutout PNGs** across all class directories
- **Total: 4,654 image files** (sprites + backgrounds)

## File Naming Convention

The primary naming pattern is:

```
{class_name}_{belong}_{id}.png
```

- `class_name`: matches the parent directory name (e.g., `archer`, `hog-rider`)
- `belong`: `0` = ally (blue team / bottom half / friendly side), `1` = enemy (red team / top half / opponent side)
- `id`: numeric identifier, sometimes zero-padded to 7 digits (e.g., `0000007`), sometimes not (e.g., `11`)

**Examples:**
- `hog-rider_0_0000004.png` - ally hog-rider sprite, cutout #4
- `archer_1_0000007.png` - enemy archer sprite, cutout #7

### Extended Naming Variants

Some sprites include state/variant tags between the belong flag and the ID:

```
{class_name}_{belong}_{variant}_{id}.png
```

Known variant tags (129 sprites total use these):
- `attack` - unit in attack animation (e.g., `barbarian_1_attack_37.png`)
- `shield` - unit with shield visible (e.g., `royal-recruit_1_shield_685.png`)
- `freeze` - unit frozen (e.g., `flying-machine_1_freeze_0.png`)
- `attack_shield` - attacking with shield (e.g., `royal-recruit_1_attack_shield_405.png`)
- `freeze_shield` - frozen with shield (e.g., `royal-recruit_1_freeze_shield_165.png`)

The generator treats these the same as regular sprites for the same class. The variant tags exist because KataCR's author segmented different visual states separately.

## How the Synthetic Generator Uses These Sprites

The synthetic data generator (`src/generation/` in the `cr-object-detection` codebase) works as follows:

1. Picks a random background from `backgrounds/`
2. Randomly selects sprite PNGs from the class directories
3. Places them on the arena at random positions, using the `belong` flag to determine side: ally (`_0_`) sprites go on the bottom half, enemy (`_1_`) sprites go on the top half
4. Applies augmentation (scaling, rotation, color jitter)
5. Outputs a composite image + YOLO-format label containing class ID, bounding box, and belong flag for each placed sprite

Each training epoch generates ~20,000 unique synthetic images. The model never sees the same image twice.

## Backgrounds Directory

`backgrounds/` contains:
- **27 arena background JPGs** (background01.jpg through background27.jpg, with some gaps in numbering) - screenshots of empty Clash Royale arenas with different visual themes from seasons and special events
- **1 utility file** (`red_bound.png`) - used for rendering/masking

**Important: We only use `background15` for training.** This is a stone/railroad-themed arena that matches our actual gameplay environment (Google Play Games on PC). Using all 27 backgrounds would introduce unnecessary visual domain gap from arena styles we never encounter during real gameplay.

## The Ally Sprite Gap (CRITICAL PROBLEM)

Of the 153 class directories, **105 contain ONLY enemy (`_1_`) sprites**. Only 48 directories have any ally (`_0_`) sprites at all, and 8 of those are non-side-specific UI elements (elixir, emote, clock, etc.).

The dataset is heavily skewed: **925 ally sprites vs. 3,627 enemy sprites**.

### Why This Is a Problem

The belong flag (`_0_` or `_1_`) controls which half of the arena the sprite appears on during synthetic generation. If a unit class has zero ally sprites, the generator can only ever place it as an enemy (top half). The YOLOv8 model then learns "this unit only appears on the enemy side," which is wrong when the player deploys that same unit. This breaks side attribution during real gameplay.

KataCR's ally sprites primarily cover their specific deck (Hog 2.6 Cycle: hog-rider, musketeer, ice-spirit, ice-golem, cannon, skeleton, the-log, fireball). If our deck uses units outside that set - and most units ARE outside that set - we have a belong classification problem.

**This is the reason we forked this repository**: to add our own ally cutouts for the units in our deck.

### Classes WITH Ally Sprites

**Classes with substantial ally representation (10+ ally sprites):**

| Class | Ally | Enemy | Notes |
|-------|------|-------|-------|
| skeleton | 95 | 53 | KataCR Hog 2.6 deck |
| musketeer | 75 | 12 | KataCR Hog 2.6 deck |
| bar | 61 | 75 | UI health bar |
| ice-spirit | 48 | 18 | KataCR Hog 2.6 deck |
| hog-rider | 47 | 21 | KataCR Hog 2.6 deck |
| ice-golem | 41 | 13 | KataCR Hog 2.6 deck |
| elixir | 32 | 0 | Non-side-specific UI |
| dirt | 30 | 0 | Non-side-specific effect |
| evolution-symbol | 29 | 0 | Non-side-specific UI |
| skeleton-dragon | 29 | 72 | |
| king-tower | 29 | 47 | Tower (always both sides) |
| goblin | 26 | 37 | |
| the-log | 25 | 9 | KataCR Hog 2.6 deck |
| emote | 25 | 0 | Non-side-specific UI |
| tower-bar | 23 | 23 | UI element |
| dagger-duchess-tower-bar | 22 | 0 | UI element |
| skeleton-evolution | 21 | 25 | |
| knight | 21 | 32 | |
| queen-tower | 19 | 35 | Tower (always both sides) |
| cannon | 19 | 6 | KataCR Hog 2.6 deck |
| fireball | 19 | 18 | KataCR Hog 2.6 deck |
| axe | 18 | 0 | Non-side-specific projectile |
| small-text | 17 | 0 | Non-side-specific UI |
| clock | 17 | 13 | Non-side-specific UI |
| bomber-evolution | 16 | 14 | |
| cannoneer-tower | 13 | 7 | Tower |
| dark-prince | 13 | 21 | |
| king-tower-bar | 10 | 7 | UI element |

**Classes with minimal ally representation (1-9 ally sprites):**

| Class | Ally | Enemy |
|-------|------|-------|
| bar-level | 9 | 13 |
| dagger-duchess-tower | 8 | 10 |
| golem | 8 | 22 |
| golemite | 8 | 18 |
| giant | 7 | 16 |
| big-text | 6 | 0 |
| elite-barbarian | 6 | 28 |
| spear-goblin | 6 | 85 |
| ice-spirit-evolution | 5 | 6 |
| battle-ram | 3 | 18 |
| mighty-miner | 3 | 13 |
| background-items | 2 | 2 |
| goblin-cage | 2 | 2 |
| goblin-hut | 2 | 1 |
| mortar | 2 | 8 |
| royal-giant | 2 | 8 |
| tesla-evolution | 2 | 10 |
| valkyrie | 2 | 20 |
| firecracker | 1 | 25 |
| goblin-drill | 1 | 7 |

### Classes with ZERO Ally Sprites (105 classes)

All remaining 105 class directories contain only enemy sprites. Notable examples that could appear in a player's deck: archer (44 enemy), barbarian (42), wizard (41), baby-dragon (35), bandit (36), pekka (22), mini-pekka (26), prince (58), mega-knight (60), lumberjack (22), witch (17), balloon (8), and all evolution variants except bomber-evolution, skeleton-evolution, ice-spirit-evolution, and tesla-evolution.

## Class Categories

The 153 entity classes (excluding backgrounds) fall into these categories:

**Troops (ground):** archer, archer-queen, bandit, barbarian, battle-healer, bowler, dark-prince, dart-goblin, electro-giant, electro-spirit, electro-wizard, elite-barbarian, executioner, fire-spirit, firecracker, fisherman, giant, giant-skeleton, goblin, goblin-brawler, golden-knight, golem, golemite, guard, heal-spirit, hog, hog-rider, hunter, ice-golem, ice-spirit, ice-wizard, knight, little-prince, lumberjack, magic-archer, mega-knight, mighty-miner, miner, mini-pekka, monk, mother-witch, musketeer, night-witch, pekka, prince, princess, ram-rider, rascal-boy, rascal-girl, royal-ghost, royal-giant, royal-guardian, royal-hog, royal-recruit, skeleton, skeleton-king, sparky, spear-goblin, valkyrie, wall-breaker, witch, wizard, zappy

**Troops (air):** baby-dragon, balloon, bat, electro-dragon, flying-machine, inferno-dragon, lava-hound, lava-pup, mega-minion, minion, phoenix-big, phoenix-small, skeleton-barrel, skeleton-dragon

**Evolution variants:** archer-evolution, barbarian-evolution, bat-evolution, battle-ram-evolution, bomber-evolution, firecracker-evolution, ice-spirit-evolution, knight-evolution, mortar-evolution, royal-giant-evolution, royal-recruit-evolution, skeleton-evolution, tesla-evolution, valkyrie-evolution, wall-breaker-evolution

**Buildings:** barbarian-hut, bomb-tower, cannon, cannon-cart, elixir-collector, furnace, goblin-cage, goblin-drill, goblin-hut, inferno-tower, mortar, tesla, tombstone, x-bow

**Towers:** cannoneer-tower, dagger-duchess-tower, king-tower, queen-tower

**Spells and effects:** arrows, clone, earthquake, fireball, freeze, giant-snowball, graveyard, lightning, poison, rage, rocket, royal-delivery, the-log, tornado, zap

**UI and HUD elements:** bar, bar-level, big-text, clock, dagger-duchess-tower-bar, elixir, emote, evolution-symbol, king-tower-bar, skeleton-king-bar, skeleton-king-skill, small-text, tower-bar

**Special/miscellaneous:** axe (executioner projectile), bomb (from giant-skeleton/bomb-tower), bomber, dirt (miner/goblin-drill effect), elixir-golem-big, elixir-golem-mid, elixir-golem-small, goblin-ball, goblin-barrel, goblin-giant, ice-spirit-evolution-symbol, phoenix-egg, background-items

## Directory Size Reference

Top 30 directories by file count:

| Directory | Count | Directory | Count |
|-----------|-------|-----------|-------|
| royal-recruit-evolution | 150 | skeleton | 148 |
| bar | 136 | skeleton-dragon | 101 |
| rascal-girl | 98 | bat | 95 |
| spear-goblin | 91 | musketeer | 87 |
| background-items | 78 | king-tower | 76 |
| guard | 69 | hog-rider | 68 |
| royal-recruit | 67 | ice-spirit | 66 |
| goblin | 63 | mega-knight | 60 |
| ice-wizard | 60 | bat-evolution | 59 |
| prince | 58 | archer-evolution | 57 |
| skeleton-king | 56 | barbarian-evolution | 55 |
| queen-tower | 54 | ice-golem | 54 |
| sparky | 53 | little-prince | 53 |
| knight | 53 | rascal-boy | 52 |
| tower-bar | 46 | skeleton-evolution | 46 |

Smallest directories (1-4 files): tombstone (1), barbarian-hut (1), clone (2), furnace (2), graveyard (3), goblin-hut (3), goblin-cage (4), skeleton-king-skill (4), x-bow (4).

## How Sprites Were Originally Created

KataCR's author (WTY) created these cutouts by:
1. Recording YouTube gameplay videos of Clash Royale
2. Extracting frames at 15-frame intervals
3. Segmenting units from frames using a combination of manual segmentation and SAM (Segment Anything Model) auto-segmentation
4. Organizing cutouts into class directories with ally/enemy labels based on which side the unit belonged to in the source video

This process spanned 15 labeling sessions from November 2023 to March 2024.

## Adding New Sprites to This Fork

To add ally cutouts for units in our deck (the primary reason this repo was forked):

1. **Create transparent PNGs** - crop the unit from gameplay screenshots, ensuring a transparent (alpha channel) background
2. **Name correctly** - use `{class_name}_0_{id}.png` where `0` indicates ally; use a high numeric ID to avoid conflicts with existing files (e.g., start at 9000)
3. **Place in the right directory** - put files in the matching `{class_name}/` subdirectory
4. **Match existing scale** - cutout dimensions should be roughly similar to existing sprites in that class directory; the generator handles scaling but wildly different base sizes could cause issues
5. **Verify transparency** - sprites must have an alpha channel for proper compositing onto backgrounds; opaque rectangular crops will produce unrealistic training images

After adding sprites, no code changes are needed. The synthetic generator automatically discovers all PNG files in each class directory at runtime.

## Relationship to Other Dataset Directories

This repository (`Froked-KataCR-Clash-Royale-Detection-Dataset`) contains two other key directories at `images/`:

- `images/part2/` - 7,380 fully-annotated detection images (YOLO + Labelme format labels) used as the validation set. These are KataCR's human-labeled real gameplay frames. We do NOT train on these; we evaluate our synthetically-trained model against them.
- `images/card_classification/` - 227 card images for the Hog 2.6 Cycle deck, used for card classification (separate from object detection).
