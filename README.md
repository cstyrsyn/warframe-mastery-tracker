# Warframe Mastery Tracker

A portable, offline web app for tracking Mastery Rank progress in Warframe.

## Features

- **Full item coverage** — Warframes, Primary, Secondary, Melee, Companions, Companion Weapons, Vehicles, Arch Weapons, Amps, Intrinsics, Mods, and Arcanes
- **Star Chart** — track node completion for both standard and Steel Path, with per-planet XP and junction counts
- **Intrinsics** — separate tracking for Railjack and Drifter skill trees
- **Mods** — track mod collection with rank sliders, polarity, rarity, and category filters
- **Arcanes** — track arcane collection and rank progress
- **Acquisition tracking** — mark items as owned before they're levelled, so you know what's in your inventory
- **Per-item rank sliders** — set exact rank for partially levelled items
- **Projected MR** — live XP total and Mastery Rank projection as you update ranks
- **XP overrides** — manually adjust XP for star chart planets if your node count differs
- **Summary tab** — at-a-glance owned/completed counts and XP totals across all categories
- **Category filters** — narrow any tab down to a specific subcategory
- **Visibility toggles** — hide maxed, acquired, or unowned items per tab
- **Wiki links** — every item name links directly to its page on the Warframe wiki
- **Tradable badges** — items that can be traded between players are flagged on their card, with a direct link to the item's listing on warframe.market
- **Component tags** — items used as crafting ingredients for other weapons show what they build into
- **Auto-backup** — optionally sync progress to a local file via the File System Access API
- **Persistent state** — progress, active tab, and UI preferences are saved to localStorage

## Getting Started

1. Download `warframe-mastery-tracker-base.html`, `Import/xlsx.full.min.js`, and `data.js` and place them in the same folder (maintaining the `Import/` subfolder).
2. Open `warframe-mastery-tracker-base.html` in your browser (Chrome or Edge recommended for full auto-backup support).
3. That's it. No internet connection required after download.

> **Card images:** The base version does not include card artwork. A full version with tile images can be built locally — download the images using the scripts in `dev/Download/` and open `warframe-mastery-tracker.html` (not tracked in git).

## Usage

### Setting Item Ranks
Click any item card to open a rank slider. Drag to your current rank and click **Save**. The card updates immediately and the projected MR recalculates.

### Marking Items as Acquired
On weapon, companion, and vehicle tabs, each card has an **Acq** button. Use this to flag items you own but haven't levelled yet. Acquired cards are visually distinguished from unowned and partially levelled ones.

### Star Chart
Each planet card shows total XP available across its nodes. Toggle between **Regular** and **Steel Path** using the section headers. Click a planet card to set your completion percentage via a slider, or use the XP override field if your node count differs from the default.

### Intrinsics
Rank sliders work the same as item cards. Railjack and Drifter trees are shown in separate sections.

### Auto-Backup
Click the floppy disk icon in the header to link a local save file. Once linked, progress is automatically written to that file whenever you make a change. To restore from a backup, use the import button in the same area.

### Filtering and Visibility
- **Category buttons** (above the card grid on most tabs) filter to a specific subcategory.
- **Visibility buttons** (Show All / Hide Maxed / etc.) toggle which completion states are shown.

## Repository Structure

| Path | Purpose |
|------|---------|
| `warframe-mastery-tracker-base.html` | The app — HTML, CSS, and JavaScript in one file (no card images) |
| `data.js` | All item data: names, categories, obtain methods, ranks, and XP values |
| `Import/xlsx.full.min.js` | Vendored xlsx bundle used by the spreadsheet import feature |
| `dev/Source/` | Source CSVs that define the content of `data.js` |
| `dev/update.js` | Script for adding new items from CSV to `data.js` — see `dev/UPDATE_README.md` |
| `dev/Build/` | Legacy one-shot build scripts used to generate the initial `data.js` sections |
| `dev/Download/` | Scripts to download card images from the wiki into `Images/` (gitignored) |
| `dev/Templates/` | CSV templates for each section — copy and fill in to add new items |
| `dev/UPDATE_README.md` | Full documentation for the update workflow |

## Adding or Updating Items

The recommended workflow uses `dev/update.js` — a validated, dry-run-by-default script that appends new items to `data.js` from CSV files.

```
node dev/update.js                          # preview all sections
node dev/update.js --section primary        # preview one section
node dev/update.js --section primary --apply  # write changes
```

Copy the relevant template from `dev/Templates/`, fill in your new items, then run the script. See `dev/UPDATE_README.md` for the full workflow, CSV formats, and validation rules.

### Manual edits

Items can also be added directly to `data.js`. Each item is an array:

```js
["Name", "Category", "How to obtain", maxRank, xpPerLevel, tradable, compFor]
```

| Field | Type | Notes |
|-------|------|-------|
| `Name` | string | Display name |
| `Category` | string | Subcategory shown on the card |
| `How to obtain` | string | Source description shown on the card |
| `maxRank` | number | `30` standard, `40` for Kuva/Tenet/Coda/Necramechs/Paracesis |
| `xpPerLevel` | number | `200` for frames/companions/archwings, `100` for all weapons |
| `tradable` | `1` or `0` | Optional. `1` shows a Tradable badge and market link |
| `compFor` | string | Optional. Semicolon-separated list of weapons this item crafts into |

## Data Notes

- Star chart XP values are approximate averages based on typical node counts
- Steel Path mirrors the standard star chart structure and awards the same mastery XP
- Junctions each award 1,000 XP
- Intrinsics award 1,500 XP per rank, up to rank 10 each

## Compatibility

Works in any modern browser. The **File System Access API** (used for auto-backup) requires Chrome 86+ or Edge 86+. All other features work in Firefox and Safari via localStorage only.
