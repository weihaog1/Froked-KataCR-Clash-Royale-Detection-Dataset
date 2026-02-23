# Card Classification Dataset

## Project Context

This directory is part of a forked KataCR dataset (originally from [wty-yy/Clash-Royale-Detection-Dataset](https://github.com/wty-yy/Clash-Royale-Detection-Dataset)) used by the CS175 "The Elixir Optimizers" project at UC Irvine, Winter 2026. The broader project builds a reinforcement learning agent that plays Clash Royale through screen capture, computer vision (YOLOv8 object detection + card classification + OCR), and imitation/reinforcement learning (behavior cloning then PPO).

The parent repository lives at:
```
/Users/alanguo/Codin/CS175/Froked-KataCR-Clash-Royale-Detection-Dataset/
```

The main project codebase that consumes this data lives at:
```
/Users/alanguo/Codin/CS175/Project/cr-object-detection/
```

## What This Directory Contains

227 cropped images of cards as they appear in the player's hand (the 4 card slots at the bottom of the Clash Royale game screen). The images are organized into 12 subdirectories by card class:

| Class | Count | Description |
|-------|-------|-------------|
| cannon | 20 | Cannon building card |
| empty | 59 | Empty card slot (no card available) |
| fireball | 15 | Fireball spell card |
| hog-rider | 19 | Hog Rider troop card |
| ice-golem | 18 | Ice Golem troop card |
| ice-spirit | 20 | Ice Spirit troop card |
| ice-spirit-evolution | 5 | Ice Spirit evolution variant |
| musketeer | 21 | Musketeer troop card |
| skeletons | 24 | Skeletons troop card |
| skeletons-evolution | 8 | Skeletons evolution variant |
| the-log | 18 | The Log spell card |
| _augmentation | 1 | Augmentation script or reference |

## CRITICAL: This Dataset Is Deck-Specific

This classification dataset covers ONLY KataCR's deck: **Hog 2.6 Cycle** (hog-rider, musketeer, ice-spirit, ice-golem, cannon, skeletons, the-log, fireball, plus their evolution variants).

**Our project uses a different deck.** This dataset is NOT directly useful for our card classification needs. It serves as a reference for the data format and directory structure, but the card classes do not match our deck.

## Image Format

- **File format:** `.jpg`
- **Source:** Crops extracted from 590x1280 (720p) gameplay video frames
- **Content:** Each image is a crop of a single card slot from the hand area at the bottom of the game screen
- **Naming convention:** `NNNNN_N.jpg` - the first number is the frame number, the second is the card slot position (0-3)
- **"empty" class:** Represents card slots where no card is currently available (e.g., the card was just played and is on cooldown, or elixir is still regenerating)

## How Card Classification Fits Into the Pipeline

In Clash Royale, the player always has 4 card slots visible at the bottom of the screen. At any moment, each slot either shows a playable card or is empty. A card classifier crops these 4 regions from each frame and predicts which card (or empty) is in each slot. This tells the agent which cards it can currently play, which is essential input for the decision-making policy.

The classification pipeline works as follows:
1. The screen capture system grabs a frame
2. Four fixed crop regions extract the card slot images
3. The card classifier predicts the class for each crop
4. The result feeds into the GameState, which the RL policy uses to decide actions

## What We Did Instead (For Our Deck)

Rather than expanding or relabeling this dataset, our project trained a custom **MiniResNet card classifier** (~25K parameters) for our specific 8-card deck. Key details:

- **Code location:** `Project/cr-object-detection/src/classification/card_classifier.py`
- **Training data:** Just 8 reference PNG images (one per card in our deck), with heavy data augmentation
- **Architecture:** MiniResNet - a small residual network sufficient for distinguishing 8 card classes
- **Status:** Trained and working, but not yet wired into the live StateBuilder pipeline

## If You Need Card Classification for a New Deck

Do NOT try to expand this dataset to cover additional decks. Instead:

1. Capture or crop approximately 8 reference card images (one per card in your deck)
2. Train a lightweight classifier with heavy augmentation (random brightness, contrast, rotation, noise, etc.)
3. The MiniResNet approach in `src/classification/` in the main codebase works well for this task
4. The card hand UI in Clash Royale is highly consistent, so a small number of reference images with augmentation is sufficient for reliable classification

## Related Files and Directories

- **Main codebase:** `/Users/alanguo/Codin/CS175/Project/cr-object-detection/`
- **Card classifier code:** `/Users/alanguo/Codin/CS175/Project/cr-object-detection/src/classification/card_classifier.py`
- **State builder (consumes card predictions):** `/Users/alanguo/Codin/CS175/Project/cr-object-detection/src/pipeline/state_builder.py`
- **Detection dataset (sibling directory):** `/Users/alanguo/Codin/CS175/Froked-KataCR-Clash-Royale-Detection-Dataset/images/part2/` (7,380 detection images)
- **Segment sprites (sibling directory):** `/Users/alanguo/Codin/CS175/Froked-KataCR-Clash-Royale-Detection-Dataset/images/segment/` (4,627 sprite cutouts)
- **Project-level context:** `/Users/alanguo/Codin/CS175/CLAUDE.md`
