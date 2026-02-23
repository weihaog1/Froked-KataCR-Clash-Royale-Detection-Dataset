# images/part2/ - Human-Labeled Validation Dataset

## What This Is

This directory contains 7,380 real Clash Royale gameplay screenshots with human-drawn bounding box annotations. These are the ONLY real gameplay images in the entire CS175 "The Elixir Optimizers" project. They are used exclusively for VALIDATION when training YOLOv8 object detection models on synthetic data. They must never be used for training.

This dataset is part of a forked KataCR detection dataset (`weihaog1/Froked-KataCR-Clash-Royale-Detection-Dataset`), used by the CS175 project at UC Irvine (Winter 2026) to build a reinforcement learning agent that plays Clash Royale through screen capture and computer vision.

## File Structure Per Frame

Each annotated frame consists of 3 files sharing the same numeric filename:

| File | Format | Description |
|------|--------|-------------|
| `NNNNN.jpg` | JPEG image | Gameplay screenshot |
| `NNNNN.json` | Labelme JSON | Bounding box annotations with pixel coordinates, shape labels, image dimensions |
| `NNNNN.txt` | Extended YOLO | One detection per line with class ID, bbox, belonging, and state flags |

Frame numbers are zero-padded 5-digit integers based on the original video frame index. Most episodes sample every 15th frame (e.g., `00000`, `00015`, `00030`). Some later episodes sample every 5th frame.

**Exception:** 4 subdirectories under `WTY_20240404/` (zap_evolution1 through zap_evolution4, totaling 408 frames) have JPG files only - no JSON or TXT annotations. These were likely extracted but never labeled.

## Image Resolutions

Images are NOT all the same resolution. The distribution:

| Resolution | Count | Source |
|------------|-------|--------|
| 568x896 | 5,911 | Most OYASSU and WTY episodes |
| 566x896 | 1,158 | Some WTY episodes |
| 552x896 | 284 | WTY_20240218_episodes |
| 1030x1622 | 25 | background/ reference images (not gameplay) |
| 1036x1632 | 1 | background/ |
| 1080x2400 | 1 | background/ |

All gameplay frames are approximately 568x896 (portrait orientation, roughly 720p source). The background/ directory contains larger reference images.

Note: Our project's actual gameplay environment (Google Play Games on PC) captures at 1080x1920, so there is a resolution domain gap between these validation images and our inference target.

## Directory Layout

```
part2/
  CLAUDE.md                         # This file
  ClashRoyale_detection.yaml        # YOLO dataset config (201 classes, paths)
  annotation.txt                    # 6,939 image/label path pairs (all annotated frames)
  train_annotation.txt              # 5,551 frames - KataCR's train split (we do NOT use this)
  val_annotation.txt                # 1,388 frames - KataCR's val split (we use this)
  yolo_annotation.txt               # 6,939 image paths only
  train_annotation_mini.txt         # Mini subset train split (OYASSU_20210528 only, 1,816 frames)
  val_annotation_mini.txt           # Mini subset val split (OYASSU_20210528 only, 454 frames)
  mini_dataset_readme.md            # Notes on the mini subset
  yolo_label.cache                  # Ultralytics label cache (auto-generated)
  background/                       # 29 labeled arena background reference images (no gameplay)
  OYASSU_20210528_episodes/         # 5 episodes from player OYASSU (YouTube, 2021)
    1/                              # 521 frames
    2/                              # 490 frames
    3/                              # 587 frames
    5/                              # 483 frames (episode 4 is absent)
    6/                              # 193 frames
  OYASSU_20230203_episodes/
    2/                              # 260 frames
  OYASSU_20230212_episodes/
    4/                              # 600 frames
  OYASSU_20230305_episodes/
    4/                              # 557 frames
  WTY_20240218_episodes/
    1/                              # 284 frames
  WTY_20240222_8spells/
    1/                              # 110 frames (8 spell types)
  WTY_20240227_miners/
    1/                              # 432 frames (miner, goblin-drill, etc.)
  WTY_20240301/1/                   # 242 frames (archer-evolution, balloon, bowler, golem)
  WTY_20240303/1/                   # 219 frames (bomber, electro-dragon, executioner)
  WTY_20240304/1/                   # 264 frames (skeleton, fisherman, furnace, giant-skeleton)
  WTY_20240305/1/                   # 320 frames (cannon, guard, heal-spirit, ice-wizard)
  WTY_20240306/1/                   # 344 frames (lava-hound, mega-knight, mini-pekka)
  WTY_20240307/
    1/                              # 342 frames (phoenix, prince, sparky, tesla)
    2/                              # 26 frames
  WTY_20240308/
    1/                              # 190 frames (tombstone, valkyrie, wall-breaker, witch)
    2/                              # 54 frames
    3/                              # 52 frames
  WTY_20240309/                     # Evolution card recordings
    WTY_20240309_barbarian/         # 40 frames
    WTY_20240309_barbarian_2/       # 22 frames
    WTY_20240309_bat/               # 36 frames
    WTY_20240309_firecracker/       # 82 frames
    WTY_20240309_ice_spirit/        # 20 frames
    WTY_20240309_royal_recurit/     # 52 frames (note: typo in dir name, "recurit")
    WTY_20240309_wall_breaker/      # 12 frames
    WTY_20240309_wall_breaker_2/    # 12 frames
  WTY_20240404/                     # Zap evolution recordings
    zap_evolution1/                 # 107 frames [JPG ONLY - no annotations]
    zap_evolution2/                 # 131 frames [JPG ONLY - no annotations]
    zap_evolution3/                 # 94 frames [JPG ONLY - no annotations]
    zap_evolution4/                 # 76 frames [JPG ONLY - no annotations]
    zap_evolution5/                 # 31 frames (fully annotated)
  WTY_20240412/                     # Dagger duchess + cannoneer tower recordings
    dagger0_cannoneer1_1/           # 16 frames
    dagger0_cannoneer1_2/           # 1 frame
    dagger1_cannoneer0_1/           # 3 frames
    dagger1_cannoneer0_2/           # 2 frames
    dagger1_cannoneer0_3/           # 2 frames
    dagger1_cannoneer0_4/           # 1 frame
    dagger1_cannoneer0_5/           # 1 frame
    dagger1_cannoneer0_6/           # 1 frame
    dagger1_cannoneer0_7/           # 1 frame
    dagger1_cannoneer0_evolution_ram/ # 27 frames
  lan77_20240406_episodes/
    2/                              # 11 frames
```

## Labelme JSON Format

Each `.json` file follows the Labelme v5.3.1 annotation format:

```json
{
  "version": "5.3.1",
  "flags": {},
  "shapes": [
    {
      "label": "queen-tower1",
      "points": [[407.19, 166.55], [498.94, 265.52]],
      "group_id": null,
      "description": "",
      "shape_type": "rectangle",
      "flags": {}
    }
  ],
  "imagePath": "00000.jpg",
  "imageData": null,
  "imageHeight": 896,
  "imageWidth": 568
}
```

### Labelme Label Naming Convention

Labels in the JSON encode the class name, belonging side, and state:

```
{class_name}{belong_suffix}[_{state}]
```

- `belong_suffix`: `0` = ally, `1` = enemy, or omitted for neutral elements
- `state` (optional): `_freeze`, `_attack`, `_deploy`, `_slow`, `_rage`, etc.

Examples:
- `queen-tower1` - enemy queen tower
- `queen-tower0` - ally queen tower
- `king-tower1_freeze` - enemy king tower, not yet activated ("freeze" means inactive)
- `hog-rider0_attack` - ally hog rider in attack state
- `cannon0_deploy` - ally cannon being deployed
- `barbarian1_attack_slow` - enemy barbarian attacking while slowed
- `text` - neutral text element (no belong suffix)
- `clock` - neutral clock element
- `elixir` - neutral elixir drop

Note: `_freeze` on king towers specifically means the tower has NOT been activated yet (it does not mean the freeze spell). Similarly, `queen-tower_destory` (sic) means the queen tower has been destroyed.

## YOLO Label Format (.txt files)

Each `.txt` file contains one line per detected object:

```
class_id center_x center_y width height belong s1 s2 s3 s4 s5 s6
```

| Field | Type | Range | Description |
|-------|------|-------|-------------|
| `class_id` | int | 0-154 | Class index (see class list in ClashRoyale_detection.yaml) |
| `center_x` | float | 0.0-1.0 | Normalized bounding box center X |
| `center_y` | float | 0.0-1.0 | Normalized bounding box center Y |
| `width` | float | 0.0-1.0 | Normalized bounding box width |
| `height` | float | 0.0-1.0 | Normalized bounding box height |
| `belong` | int | 0 or 1 | 0 = ally (blue/bottom side), 1 = enemy (red/top side) |
| `s1` | int | 0-3 | Movement state (0=normal, 1=attack, 2=deploy, 3=freeze/inactive) |
| `s2` | int | 0-1 | Shield present |
| `s3` | int | 0-1 | Visible/invisible |
| `s4` | int | 0-1 | Rage effect active |
| `s5` | int | 0-1 | Slow effect active |
| `s6` | int | 0-1 | Heal or clone effect |

Example line:
```
0 0.501761 0.130580 0.215987 0.167986 1 3 0 0 0 0 0
```
This means: class 0 (king-tower), centered at (0.50, 0.13), size 0.22x0.17, enemy side, movement state 3 (frozen/inactive), no other states.

**Important:** Standard YOLO format uses only the first 5 fields (`class_id center_x center_y width height`). The remaining 7 fields (`belong` + 6 state flags) are KataCR-specific extensions. Standard YOLOv8 training with Ultralytics silently ignores these extra columns, which is how our training pipeline works - we train a standard 5-field detector and do not use the belonging or state information during training.

## Class List

The `ClashRoyale_detection.yaml` file defines 201 classes (indices 0-200):

- **Indices 0-154:** 155 actual game object classes (troops, buildings, spells, towers, UI elements)
- **Indices 155-199:** `pad_0` through `pad_44` - padding classes used by KataCR's custom model head
- **Index 200:** `pad_belong` - additional padding class

Key class examples:

| Index | Class | Index | Class |
|-------|-------|-------|-------|
| 0 | king-tower | 80 | hog-rider |
| 1 | queen-tower | 112 | prince |
| 5 | tower-bar | 133 | pekka |
| 11 | text | 135 | mega-knight |
| 12 | elixir | 136 | lava-hound |
| 15 | skeleton | 138 | golem |
| 35 | archer | 140 | little-prince |
| 58 | miner | 147 | evolution-symbol |
| 69 | fireball | 154 | zap-evolution |

The full 155-class list is in `ClashRoyale_detection.yaml` at the top of this directory.

## Annotation Index Files

| File | Lines | Purpose |
|------|-------|---------|
| `annotation.txt` | 6,939 | Pairs: `./path/to/NNNNN.jpg ./path/to/NNNNN.txt` for all annotated frames |
| `yolo_annotation.txt` | 6,939 | Image paths only: `./path/to/NNNNN.jpg` |
| `train_annotation.txt` | 5,551 | KataCR's 80/20 train split (we do NOT use this for training) |
| `val_annotation.txt` | 1,388 | KataCR's 80/20 val split (our project uses this subset for validation) |
| `train_annotation_mini.txt` | 1,816 | Mini train split from OYASSU_20210528 episodes only |
| `val_annotation_mini.txt` | 454 | Mini val split from OYASSU_20210528 episodes only |

The train/val split of these real images was created by KataCR for their own evaluation purposes. In our project, we only use `val_annotation.txt` (1,388 images) as the validation set during YOLOv8 training. We never train on any of these real images.

## How This Is Used in Our Project

1. The file `configs/synthetic_data.yaml` in our main codebase (`CS175/Project/cr-object-detection/`) points its `val:` field to a prepared copy of 1,388 images from this dataset (stored in `data/prepared/images/val/`)
2. During YOLOv8 training on synthetic data, the model is evaluated against these real images to measure generalization across the synthetic-to-real domain gap
3. Our primary performance metric is mAP50 on this validation set (current best: 0.804)
4. The 1,388-image subset comes from `val_annotation.txt`

## Labeling History and Provenance

The dataset was built incrementally by the KataCR project author (WTY) from late 2023 through April 2024 across 15+ labeling sessions. Each session focused on specific unit types to ensure class coverage:

- **OYASSU sessions (2021-2023):** Recorded from a Japanese YouTube player named OYASSU. These are the oldest and largest episode sets, covering diverse decks.
- **WTY sessions (Feb-Apr 2024):** Recorded from KataCR author's own gameplay. Each session targets specific units (e.g., WTY_20240304 focuses on skeleton, fisherman, furnace, giant-skeleton). The author used Labelme for annotation, with YOLOv4/v5 model predictions assisting later sessions to speed up the process.
- **lan77 session (Apr 2024):** One match from a contributor named lan77.

Per the KataCR annotation logs, each labeling session took approximately 60-160 minutes, with an additional 60-150 minutes for sprite extraction from the same frames.

## Important Warnings

- **NEVER use these images for training.** The entire point of the KataCR approach is to train on synthetic data and validate on real data. Training on these images produces inflated metrics (we got mAP50=0.944 by mistake) but terrible generalization to new gameplay.
- **Resolution domain gap.** These frames are roughly 568x896 from 720p YouTube recordings. Our actual gameplay captures are 1080x1920 from Google Play Games on PC. The visual differences (resolution, rendering quality, UI scaling) contribute to the domain gap between validation metrics and real-world detection performance.
- **Not all frames are annotated.** 408 frames in WTY_20240404 (zap_evolution1-4) have JPG images but no JSON or TXT label files. These are excluded from the annotation index files.
- **The `yolo_label.cache` file** is auto-generated by Ultralytics during training. It caches parsed label data for faster subsequent loads. Delete it if you modify any label files, as it will contain stale data.
- **Directory name typo:** `WTY_20240309_royal_recurit` is misspelled (should be "recruit"). Do not rename it, as annotation index files reference this exact path.

## Related Files and Paths

- **YOLO dataset config:** `ClashRoyale_detection.yaml` (in this directory)
- **Main project codebase:** `CS175/Project/cr-object-detection/`
- **Synthetic data generator:** `CS175/Project/cr-object-detection/src/generation/`
- **Prepared val subset (1,388 images):** `CS175/Project/cr-object-detection/data/prepared/images/val/`
- **Training config:** `CS175/Project/cr-object-detection/configs/synthetic_data.yaml`
- **Parent directory CLAUDE.md:** `../CLAUDE.md` (covers all image directories)
- **Repository root CLAUDE.md:** `../../CLAUDE.md` (covers the full forked dataset)
- **Upstream dataset:** https://github.com/wty-yy/Clash-Royale-Detection-Dataset
