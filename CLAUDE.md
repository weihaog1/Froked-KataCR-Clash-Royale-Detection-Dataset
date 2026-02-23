# Forked KataCR Clash Royale Detection Dataset

## What This Repo Is

This is a fork of `wty-yy/Clash-Royale-Detection-Dataset`, the dataset companion to the KataCR project. Our fork lives at `weihaog1/Froked-KataCR-Clash-Royale-Detection-Dataset`.

It is used by the CS175 project "The Elixir Optimizers" - a reinforcement learning agent that plays Clash Royale through screen capture and computer vision. The project is at UC Irvine, Winter 2026.

**Team:** Alan Guo (weihaog1), Josh Talla (tallaj), Lakshya Shrivastava (Lshrivas)

## Why We Forked

KataCR's dataset only has ally-side (`_0_`) sprite cutouts for their specific deck (Hog 2.6 Cycle: hog-rider, musketeer, ice-spirit, ice-golem, cannon, skeletons, the-log, fireball). Our project uses a DIFFERENT deck, and needs ally cutouts for our cards too. We forked to add our own sprite cutouts while preserving the original dataset structure.

**The ally sprite cutout gap is a critical problem.** Without ally sprites for our deck's cards, the synthetic data generator cannot create training images where our units appear as allies (blue side). The model would only learn enemy appearances for those units, breaking side classification entirely.

## Upstream Source

- **Original dataset repo:** https://github.com/wty-yy/Clash-Royale-Detection-Dataset
- **KataCR project:** https://github.com/wty-yy/KataCR
- **License:** MIT

## How This Dataset Is Used

This dataset is consumed by our synthetic data generation pipeline located at `CS175/Project/cr-object-detection/src/generation/`. The generator works as follows:

1. Loads arena backgrounds from `images/segment/backgrounds/`
2. Loads sprite cutouts from `images/segment/{class_name}/`
3. Composites sprites onto backgrounds to create synthetic training images with auto-generated YOLO labels
4. The trained YOLOv8s model detects objects in real Clash Royale gameplay

KataCR's approach (which we follow): Train ONLY on synthetic data, validate on real human-labeled images. The real images in `images/part2/` are never used for training.

## Repository Structure

```
Froked-KataCR-Clash-Royale-Detection-Dataset/
  images/
    segment/                # 4,627 sprite cutouts across 154 classes (TRAINING data source)
      backgrounds/          # 28 arena background images (background01.jpg - background28.jpg)
      {class_name}/         # Transparent PNG cutouts per unit class
    part2/                  # 7,380 human-labeled gameplay frames (VALIDATION data only)
      {video_name}/         # Organized by source video (e.g., WTY_20240301)
        {episode}/          # Each round in a match
          NNNNN.jpg         # Screenshot frame
          NNNNN.json        # Labelme bounding box annotations
          NNNNN.txt         # YOLO format labels (auto-converted from json)
      ClashRoyale_detection.yaml  # Dataset config for YOLO training
    card_classification/    # 228 card images (Hog 2.6 deck ONLY - not useful for our deck)
    elixir_classification/  # 526 elixir count images across 6 classes
  version_info/             # Annotation and segment version history
  asserts/                  # README images (generation examples, segment size distribution)
  README.md                 # Original Chinese README
  README_en.md              # English README
  README_old.md             # Older README version
  Clash Royale dataset annotation.md   # Detailed annotation format documentation
  Images annotate logs.md              # Annotation progress logs (Chinese)
  LICENSE                   # MIT license
  CLAUDE.md                 # This file
```

## Sprite Naming Convention

Files in `images/segment/{class_name}/` follow this exact format:

```
{class_name}_{belong}_{id}.png
```

- `class_name`: Unit name using hyphens for multi-word names (e.g., `archer`, `hog-rider`, `knight`)
- `belong`: `0` = ally (blue/bottom side), `1` = enemy (red/top side)
- `id`: Zero-padded numeric identifier (e.g., `0000007`)

Examples:
- `archer_1_0000007.png` - enemy archer, ID 7
- `hog-rider_0_0000003.png` - ally hog-rider, ID 3
- `knight_0_0000059.png` - ally knight, ID 59

All sprites are transparent PNGs suitable for compositing onto arena backgrounds.

## Ally vs Enemy Sprite Gap (The Key Problem)

Most classes only have ENEMY (`_1_`) sprites. Only a subset have ALLY (`_0_`) sprites. This is the primary reason this fork exists.

### Classes WITH ally (`_0_`) sprites

**Hog 2.6 deck (KataCR's deck) - well-represented:**
| Class | Ally Count | Enemy Count |
|-------|-----------|-------------|
| hog-rider | 47 | 21 |
| musketeer | 75 | 12 |
| ice-spirit | 48 | - |
| ice-golem | 41 | - |
| cannon | 19 | - |
| skeleton | 95 | - |
| the-log | 25 | - |
| fireball | 19 | - |

**Towers and UI elements:**
king-tower (29), queen-tower (19), cannoneer-tower (13), dagger-duchess-tower (8), bar (61), bar-level (9), tower-bar (23), king-tower-bar (10), dagger-duchess-tower-bar (22)

**Non-side elements:**
clock (17), elixir (32), emote (25), dirt (30), evolution-symbol (29), big-text (6), small-text (17), background-items (2), axe (18)

**Other units with some ally sprites (sparse):**
knight (21), dark-prince (13), elite-barbarian (6), giant (7), goblin (26), golem (8), golemite (8), skeleton-evolution (21), skeleton-dragon (29), bomber-evolution (16), battle-ram (3), firecracker (1), goblin-cage (2), goblin-drill (1), goblin-hut (2), mortar (2), royal-giant (2), spear-goblin (6), tesla-evolution (2), valkyrie (2), mighty-miner (3), ice-spirit-evolution (5)

### Classes WITHOUT any ally sprites (examples)

archer, baby-dragon, bandit, barbarian, bowler, electro-wizard, and many others. These units can only appear as enemies in synthetic training data. If our deck contains any of these units, the model will not learn their ally appearance.

## YOLO Label Format (part2)

Each `.txt` file in `images/part2/` has one line per detection:

```
class_id center_x center_y width height belong s1 s2 s3 s4 s5 s6
```

| Field | Description |
|-------|-------------|
| `class_id` | Integer class index (0-154) |
| `center_x, center_y` | Normalized bbox center (0-1) |
| `width, height` | Normalized bbox dimensions (0-1) |
| `belong` | 0 = ally, 1 = enemy |
| `s1-s6` | State flags: movement, shield, visible, rage, slow, heal/clone (usually all zeros) |

This is an extended YOLO format. Standard YOLO only has the first 5 fields (class_id through height). The `belong` and state fields are KataCR-specific additions.

## 155 Class List

The full class list is defined in KataCR's source at `katacr/constants/label_list.py`. The 154 class directories in `images/segment/` correspond to classes 0-153, with one additional class at index 154. Classes include:

- **Troops:** Ground and air units, plus evolution variants (e.g., archer, archer-evolution)
- **Buildings:** Towers, cannons, spawners (e.g., cannon, barbarian-hut, inferno-tower)
- **Spells:** fireball, the-log, arrows, freeze, lightning, etc.
- **UI elements:** bar, tower-bar, clock, emote, big-text, small-text
- **Special:** elixir drops, evolution symbols, dirt (from miner digging)

## Key Stats

| Metric | Value |
|--------|-------|
| Total sprite cutouts | 4,627 PNGs across 154 classes |
| Arena backgrounds | 28 (we only use background15 to match our gameplay arena) |
| Validation images | 7,380 human-labeled gameplay frames |
| Card classification images | 228 (Hog 2.6 deck only) |
| Elixir classification images | 526 across 6 classes |
| Source video episodes | ~28 matches across multiple recording sessions |

## Path Resolution

The synthetic generator finds this dataset via `constant.py` in the generation code. The path must resolve to the parent of `images/segment/`. If you rename or move this directory, update:

1. `CS175/Project/cr-object-detection/src/generation/constant.py` (or its path config)
2. Any symlinks on the remote training server

## What We Need To Add

1. **Ally sprite cutouts for our deck's cards** - Transparent PNGs following the `{class_name}_0_{id}.png` naming convention, placed in the appropriate `images/segment/{class_name}/` directories
2. **Potentially: card_classification images for our deck** - The existing card classification images are for Hog 2.6 only and not useful for our deck

## Related Project Files

- **Main codebase:** `CS175/Project/cr-object-detection/`
- **Synthetic generator:** `CS175/Project/cr-object-detection/src/generation/`
- **Generator config:** `CS175/Project/cr-object-detection/src/generation/constant.py`
- **Root project CLAUDE.md:** `CS175/CLAUDE.md`
- **Detection project CLAUDE.md:** `CS175/Project/cr-object-detection/CLAUDE.md`
