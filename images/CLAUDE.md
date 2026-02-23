# images/ - Clash Royale Detection Dataset

## Overview

This directory contains all image data for a Clash Royale object detection dataset, forked from the KataCR project (wty-yy/Clash-Royale-Detection-Dataset). It is part of the CS175 "The Elixir Optimizers" project at UC Irvine (Winter 2026) - a reinforcement learning agent that plays Clash Royale through screen capture and computer vision.

The dataset feeds a synthetic data generation pipeline that creates training images for YOLOv8 object detection. The core philosophy: train on synthetic data only, validate on real gameplay frames only.

## Directory Structure

```
images/
  segment/                    # Sprite cutouts - source material for synthetic training data
    backgrounds/              # 28 arena background images (27 arenas + 1 red_bound)
    {class_name}/             # 154 class directories, 4,627 transparent PNGs total
  part2/                      # Human-labeled gameplay frames - validation data ONLY
    {video_name}/{episode}/   # 7,380 frames across 18 video sources
    ClashRoyale_detection.yaml  # YOLO dataset config (201 classes including padding)
    annotation.txt            # Pairs of image paths and label paths
    background/               # 27 labeled background-only reference frames
    *.txt                     # Various annotation index files
  card_classification/        # 228 card hand crop images (Hog 2.6 deck only)
    {card_name}/              # 11 class directories + _augmentation/
  elixir_classification/      # 526 elixir count images
    {class}/                  # 6 classes: -1, -2, -3, -4, 0, 1
```

## 1. segment/ - Sprite Cutouts (TRAINING DATA SOURCE)

This is the most important directory. The synthetic data generator composites these sprites onto arena backgrounds to create unlimited unique training images with auto-generated YOLO labels.

### Contents

- **154 class directories** containing **4,627 transparent PNG cutouts** total
- **28 files in backgrounds/**: `background01.jpg` through `background27.jpg` plus `red_bound.png`
- Classes cover troops, buildings, spells, towers, UI elements, and special effects

### Sprite Naming Convention

```
{class_name}_{belong}_{id}.png
```

| Field | Description | Examples |
|-------|-------------|---------|
| `class_name` | Unit name, hyphens for multi-word | `archer`, `hog-rider`, `ice-spirit` |
| `belong` | Side ownership | `0` = ally (blue/bottom), `1` = enemy (red/top) |
| `id` | Numeric identifier (often zero-padded) | `0000007`, `105`, `29` |

Some sprites also include a state suffix before the ID, such as `_attack_` (e.g., `hog-rider_0_attack_114.png`).

Examples:
- `archer_1_0000007.png` - enemy archer
- `hog-rider_0_0000003.png` - ally hog-rider
- `knight_0_0000059.png` - ally knight

### Ally vs Enemy Sprite Distribution

The dataset has **925 ally (`_0_`) sprites** and **3,627 enemy (`_1_`) sprites**. Most classes only have enemy sprites. Ally sprites exist primarily for:

- **KataCR's Hog 2.6 deck**: hog-rider (47 ally), musketeer (75), ice-spirit (48), ice-golem (41), cannon (19), skeleton (95), the-log (25), fireball (19)
- **Towers and UI**: king-tower, queen-tower, cannoneer-tower, bar, tower-bar, clock, elixir, emote, etc.
- **A few other units** with sparse ally coverage: knight (21), dark-prince (13), giant (7), goblin (26), golem (8), skeleton-evolution (21), skeleton-dragon (29), etc.

This ally sprite gap is the primary reason this fork exists. Without ally sprites for a given card, the synthetic generator cannot create training images showing that unit on the blue (player) side.

### How the Generator Uses This

1. Picks a random background from `backgrounds/`
2. Randomly selects sprites from class directories
3. Composites them onto the background with augmentations (scale, rotation, etc.)
4. Auto-generates YOLO labels with bounding boxes and belonging info
5. Our project only uses `background15` to match our specific gameplay arena

### Not All Directories Are Unit Classes

Some directories contain non-unit visual elements:
- `background-items/` - Arena decorative elements
- `big-text/`, `small-text/` - In-game text overlays
- `bar/`, `bar-level/`, `tower-bar/`, `king-tower-bar/` - HP and level bars
- `clock/` - Game timer
- `elixir/` - Elixir drops
- `emote/` - Player emotes
- `dirt/` - Miner digging effects
- `evolution-symbol/` - Evolution indicator icons
- `axe/` - Executioner's returning axe (separate from the unit)
- `bomb/` - Bomber/bomb-tower projectile

## 2. part2/ - Human-Labeled Gameplay Frames (VALIDATION ONLY)

Real gameplay screenshots with human-drawn bounding box annotations. These are used exclusively for model evaluation - never for training.

### Contents

- **7,380 labeled frames** extracted from YouTube gameplay videos
- **18 video sources** from 3 players/channels:
  - `OYASSU_*` - 4 recording sessions (2021-2023), match episodes from a Japanese player
  - `WTY_*` - 13 recording sessions (Feb-Apr 2024), various decks and scenarios
  - `lan77_*` - 1 recording session (Apr 2024)
- **background/** - 27 labeled empty arena reference frames

### File Format Per Frame

Each frame has 3 files:

| File | Format | Description |
|------|--------|-------------|
| `NNNNN.jpg` | JPEG image | The gameplay screenshot |
| `NNNNN.json` | Labelme JSON | Bounding box annotations with shapes, labels, image dimensions |
| `NNNNN.txt` | Extended YOLO | One detection per line (see format below) |

Frame numbers (NNNNN) are zero-padded 5-digit integers, often at 15-frame intervals (e.g., 00000, 00015, 00030).

### Extended YOLO Label Format

Each line in a `.txt` file:

```
class_id center_x center_y width height belong s1 s2 s3 s4 s5 s6
```

| Field | Type | Description |
|-------|------|-------------|
| `class_id` | int | Class index (0-154), per `ClashRoyale_detection.yaml` |
| `center_x` | float | Normalized bbox center X (0-1) |
| `center_y` | float | Normalized bbox center Y (0-1) |
| `width` | float | Normalized bbox width (0-1) |
| `height` | float | Normalized bbox height (0-1) |
| `belong` | int | 0 = ally, 1 = enemy |
| `s1-s6` | int | State flags (movement, shield, visible, rage, slow, heal/clone) |

This extends standard YOLO format (which has only the first 5 fields). The `belong` and state fields are KataCR-specific. Standard YOLOv8 training ignores the extra columns.

### Class Mapping

`ClashRoyale_detection.yaml` defines **201 classes** (indices 0-200):
- 0-154: Actual game object classes (155 total)
- 155-200: `pad_0` through `pad_44` plus `pad_belong` - padding classes used by KataCR's custom model head

Key class examples: `0: king-tower`, `1: queen-tower`, `15: skeleton`, `35: archer`, `69: fireball`, `80: hog-rider`, `133: pekka`, `135: mega-knight`

### Additional Files in part2/

| File | Purpose |
|------|---------|
| `ClashRoyale_detection.yaml` | YOLO dataset config with class names and paths |
| `annotation.txt` | Image/label path pairs for validation |
| `val_annotation.txt` | Validation split index |
| `train_annotation.txt` | Training split index (not used by us - we train on synthetic) |
| `*_mini.txt` variants | Smaller subset from OYASSU_20210528 only (2,270 images) |
| `mini_dataset_readme.md` | Notes on the mini dataset subset |
| `yolo_annotation.txt` | YOLO-formatted annotation index |
| `yolo_label.cache` | Ultralytics label cache file |

## 3. card_classification/ - Card Hand Images

Cropped images of cards as they appear in the player's hand during gameplay. These are deck-specific and only cover the Hog 2.6 Cycle deck.

### Contents

228 images across 12 directories (11 card classes + augmentation):

| Directory | Image Count | Description |
|-----------|-------------|-------------|
| `cannon` | 20 | Cannon card in hand |
| `empty` | 59 | Empty card slot |
| `fireball` | 15 | Fireball card |
| `hog-rider` | 19 | Hog Rider card |
| `ice-golem` | 18 | Ice Golem card |
| `ice-spirit` | 20 | Ice Spirit card |
| `ice-spirit-evolution` | 5 | Ice Spirit (evolved) card |
| `musketeer` | 21 | Musketeer card |
| `skeletons` | 24 | Skeletons card |
| `skeletons-evolution` | 8 | Skeletons (evolved) card |
| `the-log` | 18 | The Log card |
| `_augmentation` | 1 | Contains `word.png` (augmentation reference) |

### Relevance to Our Project

**Not useful for our project.** These images only cover the Hog 2.6 deck. Our project uses a different deck, so we trained our own MiniResNet card classifier from 8 reference PNGs with heavy augmentation instead.

## 4. elixir_classification/ - Elixir Count Images

Cropped screenshots of the elixir counter area, used to classify the current elixir state.

### Contents

526 images across 6 classes:

| Class | Image Count | Meaning |
|-------|-------------|---------|
| `-4` | 159 | 4 elixir deficit |
| `-3` | 23 | 3 elixir deficit |
| `-2` | 88 | 2 elixir deficit |
| `-1` | 80 | 1 elixir deficit |
| `0` | 3 | Neutral elixir state |
| `1` | 173 | 1 elixir advantage |

Negative numbers represent elixir deficit states relative to some reference point in KataCR's system.

### Relevance to Our Project

**Not actively used.** Our pipeline uses PaddleOCR to read elixir values directly from the screen instead.

## Critical: Training vs Validation Separation

KataCR's core design principle (which we follow):

| Purpose | Data Source | Directory |
|---------|-------------|-----------|
| **Training** | Synthetic images generated from `segment/` sprites | Generated on-the-fly, not stored here |
| **Validation** | Real gameplay frames from `part2/` | `part2/` |

Never mix these. Training on real gameplay images (we did this by mistake early on) produces inflated metrics (mAP50=0.944) but poor generalization to new gameplay. Training on synthetic data produces lower validation metrics (mAP50=0.804) but much better real-world performance.

## Path Resolution

The synthetic data generator in `CS175/Project/cr-object-detection/src/generation/` expects this dataset's path to resolve to the parent of `images/segment/`. The path is configured in the generator's `constant.py`. If this directory is moved or renamed, that config must be updated.

## Upstream and Fork Context

- **Upstream:** `wty-yy/Clash-Royale-Detection-Dataset` (MIT license)
- **Fork:** `weihaog1/Froked-KataCR-Clash-Royale-Detection-Dataset`
- **Fork reason:** Add ally sprite cutouts for our deck's cards to `segment/`
- **KataCR project:** https://github.com/wty-yy/KataCR
