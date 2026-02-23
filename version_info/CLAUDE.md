# Version Info - Dataset Annotation History

## What This Directory Contains

Changelog and tracking files for KataCR's dataset annotation process. The dataset author (wty-yy) maintained version history as detection annotations and sprite cutouts were incrementally added over time.

### File Types

| Pattern | Count | Description |
|---------|-------|-------------|
| `annotation_v0.X.csv` | 14 files (v0.1 - v0.14) | CSV tracking bounding box annotation counts per class |
| `annotation_v0.X_update.txt` | 13 files (v0.2 - v0.14) | Changelogs describing what was added in each annotation version |
| `segment_v0.X.csv` | 15 files (v0.1 - v0.15) | CSV tracking sprite cutout counts per class |
| `segment_v0.X_update.txt` | 14 files (v0.2 - v0.15) | Changelogs for sprite cutout additions |
| `label_build.log` | 1 file | Log from the label conversion tool (Labelme JSON to YOLO txt format) |

Total: 57 files.

## CSV Format

Each CSV file has three columns:

```
,0,1
class-name,ally_count,enemy_count
```

- Column `0`: Count of ally-side (belong=0) annotations or sprites
- Column `1`: Count of enemy-side (belong=1) annotations or sprites
- First row is the header

Example from `annotation_v0.1.csv`:
```
,0,1
tower-bar,3832,6420
text,1450,0
king-tower,3228,3234
```

## Update File Format

Each `_update.txt` file shows what changed between consecutive versions:

```
                           Old (v0.1) ->       New (v0.2)    Count
tower-bar                 [3832 6420] ->      [4028 6806]        1
ice-spirit                  [375   0] ->        [405  17]        6
```

The `[ally enemy]` brackets show counts before and after the update. New classes added in that version appear with `[0 0]` as the old value.

## How the Dataset Grew

The dataset was built incrementally over approximately 5 months:

- **Early versions (v0.1 - v0.5):** Established the annotation format with a small set of decks (Hog 2.6 Cycle, coffin deck). Focused on core classes like towers, bars, and common units.
- **Mid versions (v0.6 - v0.10):** Expanded unit coverage by annotating new gameplay videos with different decks and opponents.
- **Late versions (v0.11 - v0.14/v0.15):** Filled gaps in enemy unit sprites, added evolution card variants (barbarian-evolution, bat-evolution, royal-recruit-evolution, etc.), and polished coverage.
- **Final state:** 150+ unit types covered. All units except "mirror" are represented (mirror can be handled by game logic since it copies the last played card).

Segments went one version further than annotations (v0.15 vs v0.14) because sprite cutouts were extracted independently from the bounding box annotation process.

## label_build.log

This log records runs of the label conversion tool that converts Labelme JSON annotations into YOLO-format `.txt` files for training. Each entry includes:

- Build timestamp
- Machine name (wty-Yoga-14s-Ubuntu)
- Dataset size (number of images)
- Maximum bounding box count per image
- Output paths

The earliest build recorded is `2024-02-08` and shows the dataset growing from 3,427 to 7,380+ images across multiple rebuilds.

## For Our Project

These version files are historical records from the upstream KataCR project. We do not need to update them when adding our own sprites to the fork. Changes to our forked dataset are tracked via git commits instead.

## Related Files

- **Root dataset CLAUDE.md:** `Froked-KataCR-Clash-Royale-Detection-Dataset/CLAUDE.md`
- **Sprite cutouts tracked by segment CSVs:** `images/segment/`
- **Bounding box annotations tracked by annotation CSVs:** `images/part2/`
- **Upstream repo:** https://github.com/wty-yy/Clash-Royale-Detection-Dataset
