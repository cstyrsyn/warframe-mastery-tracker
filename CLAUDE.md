# WF_TRACK_V2 — Codebase Guide

Warframe mastery tracker, local-only version.

## ⚠ Source File Structure (Important)

**`warframe-mastery-tracker-base.html`** is the **tracked source file** committed to git. It is a monolithic single-file app with all CSS, HTML, and JavaScript inline inside one `<script>` block.

**`warframe-mastery-tracker.html`** is **gitignored** (`# Full image version — tracked locally only`). It is a refactored working copy that loads JS from a separate `app.js` file. Changes made here are NOT committed.

**`app.js`** (root level) is also **untracked/gitignored**. It is the extracted JS from the working copy.

**Rule: always edit `warframe-mastery-tracker-base.html`** when making changes that need to be committed. The working `warframe-mastery-tracker.html` + `app.js` need to be kept in sync manually.

## Files

| File | Tracked? | Purpose |
|------|----------|---------|
| `warframe-mastery-tracker-base.html` | ✅ Yes | Source — all CSS, HTML, JS inline |
| `warframe-mastery-tracker.html` | ❌ Gitignored | Working copy — loads external app.js |
| `app.js` | ❌ Untracked | Extracted JS for the working copy |
| `data.js` | ✅ Yes | Item data arrays (WARFRAMES, WEAPONS, MODS, ARCANES, etc.) |
| `relics.js` | ✅ Yes | Relic drop data |
| `dev/Build/build.js` | ✅ Yes | Builds data.js from CSV files (not an HTML build step) |

## HTML Structure (ctrl bar)

```
#ctrl
  #ctrl-row1   (flex, wraps)
    #search            — text input, oninput="render()"
    #cat-btns          — display:contents; filled by populateCatFilter() / buildModDropdowns() / buildArcaneDropdowns()
    #status-dd         — filled by buildStatusDropdown(); hidden on special tabs
    #fb-incarnon       — toggle button, shown only on primary/secondary/melee tabs
    #circuit-week-ind  — text span, shown on incarnon tabs
    #circuit-wf-week-ind — text span, shown on warframes tab
    #fb-conclave       — toggle button, shown on mods tab only
    #fb-flawed         — toggle button, shown on mods tab only
    #tab-stat          — XP/completion stat (margin-left:auto)
  #ctrl-row2   (flex; hidden on special tabs)
    #fb-tile / #fb-list   — view toggle buttons
    #fb-grp               — group toggle button
    #fb-wftile            — tile art toggle (CARD_IMAGE_TABS only)
    #fb-wfbg              — bg art toggle (intrinsics only)
```

## State Variables

```javascript
let progress = {};           // all saved data — itemKey / aqKey / arcKey / modKey
let activeTab = 'summary';
let filters = { status: '', incarnon: false };  // status: ''|'unowned'|'notStarted'|'inProgress'|'maxed'
let activeCategory = '';
let activeType = '';         // mods only
let activeUse = '';          // mods only
let activeArcaneType = '';
let activeArcaneRarity = '';
let activeArcaneCategory = '';
let groupedView = false;
let listView = false;
let modShowConclave = false;
let modShowFlawed = false;
```

## Tabs

| Tab key | AQ_TABS? | Notes |
|---------|----------|-------|
| `warframes` | ✅ | |
| `companions` | ✅ | |
| `primary` | ✅ | incarnon filter shown |
| `secondary` | ✅ | incarnon filter shown |
| `melee` | ✅ | incarnon filter shown |
| `vehicles` | ✅ | |
| `compWeapons` | ✅ | |
| `archWeapons` | ✅ | |
| `amps` | ✅ | |
| `mods` | ❌ | special: Category/Type/Use dropdowns, Conclave/Flawed toggles |
| `arcanes` | ❌ | special: Type/Rarity/Category dropdowns |
| `relics` | ❌ | |
| `intrinsics` | ❌ | no group view |
| `conclave` | ❌ | |
| `starChart` | — | special: hides all filters |
| `summary` | — | special: hides all filters |
| `checklist` | — | special: hides all filters |

AQ_TABS items have a separate "acquired" flag distinct from rank.

## Data Model

| Key pattern | Meaning |
|-------------|---------|
| `PFX[tab] + name` | Item rank (0–maxRank) |
| `'aq:' + itemKey(tab, name)` | Acquired flag (AQ_TABS only) |
| `'arc:' + name` | Arcane copy count |
| `itemKey('mods', name)` | Mod rank |
| `aqKey('mods', name)` | Mod owned flag |

### Item status (AQ_TABS)

| Status | Condition | Card class |
|--------|-----------|-----------|
| Unowned | rank=0, !aq | (none) |
| Not Started | rank=0, aq=true | `acquired` (blue) |
| In Progress | 0 < rank < maxRank | `partial` (gold) |
| Maxed | rank === maxRank | `maxed` (green) |

## XP Rates (`TAB_XP_PER_LEVEL`)

| Category | XP/rank |
|----------|---------|
| warframes, companions, vehicles | 200 |
| all weapons, amps | 100 |
| intrinsics | 1500 |
| mods, arcanes | not in table (no MR XP) |

## Key Functions

| Function | Purpose |
|----------|---------|
| `switchTab(tabEl)` | Resets filters, restores prefs, wires DOM visibility |
| `populateCatFilter()` | Injects category dropdown/buttons into #cat-btns |
| `setCatFilter(val)` | Sets activeCategory, rebuilds filter, renders |
| `buildModDropdowns()` | Mods: Category + Type + Use dropdowns |
| `buildArcaneDropdowns()` | Arcanes: Type + Rarity + Category dropdowns |
| `makeDd(id, label, opts, activeVal, onSelect, noSearch)` | Builds custom .sdd dropdown widget |
| `buildStatusDropdown()` | Injects Status dropdown into #status-dd |
| `setStatusFilter(val)` | Sets filters.status, saves to localStorage, renders |
| `restoreStatus()` | Loads filters.status from localStorage, rebuilds dropdown |
| `toggleFilt(key, btn)` | Toggle for incarnon filter (button) |
| `getVisibleItems()` | Filters items for bulk-action bar |
| `getVisibleMods()` | Filters mods (used by bulk bar) |
| `getVisibleArcanes()` | Filters arcanes (used by bulk bar) |
| `render()` | Main render dispatcher |
| `buildItem(tab, name, ...)` | Builds a card element |
| `totalXP()` | Current earned XP across all tabs + star chart |
| `potentialXP()` | totalXP + remaining XP on owned AQ_TABS items |
| `getCurrentMR(xp)` / `getNextMR(xp)` | Look up rank from MASTERY array |
| `updateHeader()` | Updates MR badge, potential badge, XP bar |
| `saveProgress()` / `deferSave()` | Persist progress to localStorage |

## Status Filter Values

```javascript
const STATUS_KEYS   = { 'Unowned': 'unowned', 'Not Started': 'notStarted', 'In Progress': 'inProgress', 'Maxed': 'maxed' };
const STATUS_LABELS = { 'unowned': 'Unowned', 'notStarted': 'Not Started', 'inProgress': 'In Progress', 'maxed': 'Maxed' };
```

"Not Started" option only shown for AQ_TABS, mods, and arcanes.

## Potential MR Logic

`potentialXP()` = `totalXP()` + remaining XP for every **AQ_TABS** item where `rank < maxRank && (rank > 0 || acquired)`. Mods, arcanes, intrinsics, and star chart uncompleted nodes are NOT included in potential.
