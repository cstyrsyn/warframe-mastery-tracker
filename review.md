# WF_TRACK_V2 — Code Review Notes

> Read-only audit. No changes have been made.

---

## 1. `xpPerLevel` is always a tab-level constant

**~796 redundant values in data.js.**

Every item stores `xpPerLevel` as position `[4]` in its array, but the value is always identical for every item within a tab — determined by tab, not by item:

- `200` → Warframes, Companions, Vehicles (archwings, K-drives, Plexus, Necramechs)
- `100` → All weapons (Primary, Secondary, Melee, Comp. Weapons, Arch-Weapons, Amps)
- `1500` → Intrinsics

Could be replaced with a single lookup:
```javascript
const TAB_XP_PER_LEVEL = { warframes:200, primary:100, secondary:100, melee:100, companions:200, compWeapons:100, vehicles:200, archWeapons:100, amps:100, intrinsics:1500 };
```

Removing it from every item entry would shorten `data.js` by ~800 values. Would also require updating all `item[4]` references in the renderer.

---

## 2. `buildCard` and `buildListRow` are ~95% duplicate code

**Lines 1685–1802 in `warframe-mastery-tracker.html`.**

Both functions take the same signature and share identical logic for: rank calculation, XP, incarnon state, vaulted tag, comp tag, circuit tag, checklist state, recipe check. The ~25 `const` declarations at the top are copy-pasted verbatim.

The only real differences:
- `buildCard` renders the tile image; `buildListRow` does not
- Outer wrapper adds `list-row` class in list mode
- Badge container uses different class names (`card-top` vs `list-name-col`/`list-badges`)
- `buildCard` wraps buttons in `card-foot`; `buildListRow` does not

Merge into one function with a `mode` parameter (`'card'` | `'list'`).

---

## 3. `parseCSVLine` duplicated 5 times across build tools

Defined independently in:
- `dev/Build/build.js`
- `dev/Build/build_mods.js`
- `dev/Build/build_arcanes.js`
- `dev/Build/build_warframes.js`
- `dev/update.js`

All are functionally identical. Extract to a shared `dev/lib/csv.js` module.

---

## 4. `cleanStr` / `jsEsc` also duplicated between build scripts

`dev/Build/build_mods.js` and `dev/Build/build_arcanes.js` each define their own string escaping helpers. Same candidate for `dev/lib/` as item 3.

---

## 5. 7 near-identical key builder functions

```javascript
function itemKey(tab, name)     { return (PFX[tab] || tab+':') + name; }
function aqKey(tab, name)       { return 'aq:' + itemKey(tab, name); }
function incarnonKey(tab, name) { return 'inc:' + itemKey(tab, name); }
function modKey(name)           { return 'md:' + name; }
function modAqKey(name)         { return 'aq:md:' + name; }
function arcKey(name)           { return 'arc:' + name; }
function scKey(type, name)      { return type + name; }
```

The first three already chain correctly. The mod/arcane ones follow the same pattern but are disconnected. Could be unified into one parametrized builder.

---

## 6. `dev/Import/` appears fully superseded

`dev/Import/` contains a node_modules tree for an xlsx/Excel-based importer. The entire build pipeline has been CSV-based for a long time. This looks like early-era tooling that was replaced and never removed. Confirm nothing calls into it, then delete.

---

## 7. Category string normalisation happens late — and inconsistently

`dev/Build/build.js` has a hardcoded comment noting that `"Arch-Guns"` in the CSV must be manually adjusted to `"Arch-Gun"` in data.js. `dev/update.js` handles this more gracefully. A shared `CATEGORY_MAP` at CSV-read time would remove the manual step.

---

## 8. `maxRank` is mostly predictable but has enough exceptions to keep explicit

Standard items = 30, Kuva/Tenet/Coda/Necramechs/Paracesis = 40, Intrinsics = 10. There are ~5 exceptions (Paracesis being the main one). Unlike `xpPerLevel`, the inference rule is messier — probably not worth removing unless item 1 is done at the same time.

---

## 9. `COMPANION_IMG_PLAIN` defined in HTML but echoed in dev tooling

The set `['Venari', 'Venari Prime']` controlling companion image suffix exists in `warframe-mastery-tracker.html` but similar logic appears in at least one dev script. Low risk given the set is tiny, but it could drift if a new companion is added.

---

## Summary

| # | Finding | Benefit |
|---|---------|---------|
| 1 | Remove `xpPerLevel` from item arrays | ~800 values removed from data.js |
| 2 | Merge `buildCard` / `buildListRow` | ~60 lines of duplicated setup removed |
| 3 | Shared `parseCSVLine` in `dev/lib/` | 5 independent copies → 1 |
| 4 | Shared `cleanStr` / `jsEsc` in `dev/lib/` | 2 independent copies → 1 |
| 5 | Unified key builder | 7 functions → 1 parametrized |
| 6 | Delete `dev/Import/` | Dead directory removed |
| 7 | `CATEGORY_MAP` in CSV parser | Removes manual post-edit step |
| 8 | `maxRank` inference | Low value — keep explicit |
| 9 | Centralise `COMPANION_IMG_PLAIN` | Prevents future drift |
