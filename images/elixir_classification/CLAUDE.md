# Elixir Classification Dataset

## What This Directory Contains

526 JPEG images of the elixir bar area from Clash Royale gameplay, used to classify relative elixir state. All images are 32x32 pixel crops (already resized from original gameplay screenshots). They are organized into 6 subdirectories by relative elixir count:

| Directory | Image Count | Meaning |
|-----------|-------------|---------|
| `-4` | 159 | 4 elixir below reference |
| `-3` | 23 | 3 elixir below reference |
| `-2` | 88 | 2 elixir below reference |
| `-1` | 80 | 1 elixir below reference |
| `0` | 3 | At reference elixir level |
| `1` | 173 | 1 elixir above reference |

The negative and positive numbers represent relative elixir states. The exact semantics relate to KataCR's specific approach for tracking elixir advantage/disadvantage rather than absolute elixir counts (0-10). The class distribution is imbalanced, with class `0` having only 3 images.

## Image Format

- **File type:** JPEG (`.jpg`)
- **Resolution:** 32x32 pixels
- **Color:** RGB, 3 channels
- **Naming:** `NNN.jpg` (sequential numbers within each class directory, not necessarily contiguous)

Example paths:
```
images/elixir_classification/-4/10.jpg
images/elixir_classification/1/103.jpg
images/elixir_classification/0/0.jpg
```

## Origin

This dataset is part of the KataCR project by wty-yy (https://github.com/wty-yy/Clash-Royale-Detection-Dataset). The images were cropped from the elixir bar region of 590x1280 gameplay screenshots and resized to 32x32 for classification. KataCR used an image classification approach to determine the player's current elixir state from these small crops.

## Project Context

This directory is part of a forked KataCR dataset used by the CS175 "The Elixir Optimizers" project (UC Irvine, Winter 2026). Our project uses PaddleOCR to read the elixir number directly from the screen instead of a classification approach, so this subdataset is **not actively used** in our pipeline. It remains in the repository as a reference and potential fallback approach.

### Why Classification vs OCR

KataCR trained a small classifier on these 32x32 crops to determine relative elixir state. Our project instead uses PaddleOCR to read the elixir number directly from the game screen, which:

- Does not require labeled training data per elixir state
- Reads absolute elixir count (0-10) rather than relative values
- Generalizes across different decks and game contexts without retraining

The classification approach has the advantage of being faster and potentially more robust to visual noise, but requires curated training examples like the ones in this directory.

## Related Files

- **Root dataset CLAUDE.md:** `Froked-KataCR-Clash-Royale-Detection-Dataset/CLAUDE.md`
- **Card classification images:** `images/card_classification/` (228 images, Hog 2.6 deck only)
- **Our OCR module:** `CS175/Project/cr-object-detection/src/ocr/`
- **Our pipeline code:** `CS175/Project/cr-object-detection/src/pipeline/`
