# timezone-site

A static, single-page weekly availability grid for coordinating between **Melbourne** and **Toronto**.

Live at: https://mk2134226.github.io/timezone-site/

## What it does

- Two tables — Melbourne and Toronto — rows Mon–Sun, 24 hourly columns each.
- **Two independent schedules on one grid.** A block only one city painted fills the whole cell; where both cities painted the same hour the cell splits, Melbourne on top and Toronto below. Both grids show both schedules — only the timezone framing differs, so the overlap is visible from either side.
- **Separate category sets per city**, each with its own names and colours, plus a running hour count. Both sets are freely editable — there is no login.
- **A Blocked category renders as a cross-hatch** rather than a flat colour, so unavailable hours read as struck out instead of being just another shade. Any category can be switched to that style with the `⨯` toggle on its chip — it's a `"pattern": true` flag in the JSON, not a hardcoded category.
- **Clicking a colour selects which layer you paint.** Click a Melbourne category and clicks land in the Melbourne layer; click a Toronto one and they land in Toronto's. The panel you're painting into is highlighted, and the choice is remembered.
- **Clear all blocks** wipes both weeks at once so you can start over; categories are kept and Undo brings the blocks back. Each panel also has a per-city clear.
- **Click or drag to paint, double-click to erase a block, ⌘/Ctrl+Z to undo.** Undo covers painting, erasing, category renames and recolours, add and delete, clearing a layer, and import or load-from-repo. A drag counts as one step rather than one per cell, and the depth is capped at 60.
- Each city's category panel sits to the right of its own table, so the palette you're editing is next to the grid it applies to. Clicking anywhere on a category chip selects it — including the name field, which stays editable.
- Saturday and Sunday rows are shaded grey so the weekend reads at a glance.
- **Copy share link** packs both schedules and both category sets into the URL fragment, so whoever opens the link sees the full picture. Painting over a shared grid makes your own local copy; the sender's link is unaffected.
- Reloading shows the committed `melbourne.json` / `toronto.json`. Local edits live in `localStorage` as a working buffer and are offered back via a banner rather than overriding what's deployed.
- **The current week loads automatically.** Each table labels its rows with its own local calendar dates, recomputed per timezone, and the page reloads if the day rolls over while it's open.
- **A red line marks "now"** in each table, positioned to the minute in that city's own local time, refreshed every 30 seconds. Today's row is highlighted.
- **The viewer's own city is shown first.** The page compares your browser's UTC offset against both cities and puts the closer one on top, with a link to swap if it guesses wrong.

Note the Toronto table's Sunday row spans two calendar dates (e.g. `26 Jul / 2 Aug`). That is not an error — folding a Melbourne week into Toronto time wraps, so that row holds the tail of one Sunday and the head of the next.

## melbourne.json / toronto.json

Shared state lives in **one file per city**, each holding that city's categories and painted blocks:

```json
{
  "version": 1,
  "categories": [ { "id": "m1", "name": "Free", "color": "#22c55e", "pattern": false } ],
  "blocks":     { "0-9": "m1" }
}
```

Block keys are `day-hour` where day is `0`=Mon … `6`=Sun, **always in Melbourne local time** — including in `toronto.json`. One canonical clock means a Toronto block painted at Wed 2PM is stored as `3-4` (Melbourne Thu 4AM). The value is the `id` of a category in the same file.

### Why two files rather than one

With a single combined file, every export rewrote the whole thing — including a snapshot of the other person's data as you last loaded it. Export after they'd committed something newer and you'd silently overwrite their work. Splitting makes that impossible: your export only ever contains your own city. It also means two people committing at the same time touch different files, so git stops producing conflicts.

The limit: two people editing the *same* city from different browsers is still last-writer-wins.

### Round trip

Since a static site has no backend, the page reads these files but cannot write them. Each panel has its own buttons:

1. **Export `<city>.json`** — downloads the file and copies it to the clipboard.
2. Commit it to this repo.
3. **Load from repo** pulls the committed version back into that city.
4. **Import `<city>.json`** takes a file directly, for passing state around without a commit.

Import and Load both replace one city only, and Import validates the shape before touching anything.

**The committed files win on load.** What is deployed is what everyone sees, including whoever deployed it — otherwise the browser you export from is the one browser that never shows the result of your own commit, which is exactly the trap the earlier localStorage-first order set.

localStorage is a working buffer, not a competing copy. If a browser holds edits that were never committed, they aren't dropped silently: the committed state loads and a banner offers **Restore my unsaved version** or **Discard**. Nothing is written to localStorage until you actually change something, so that offer survives repeated reloads.

If the page is opened directly off disk (`file://`), `fetch` is blocked by CORS and it falls back to built-in defaults. A legacy combined `data.json` is still read if the per-city files are missing.

## Who can commit

The repo being public means anyone can read and fork it — **not** commit. Only accounts with write access can push. For a second person to save their own `toronto.json` they either need to be added as a collaborator, or send the exported file to someone who can commit it.

## Timezone handling

Offsets are computed at runtime from the `Intl` timezone database against the current week's dates, not hardcoded — so the Melbourne/Toronto gap stays correct as it drifts between 14 and 16 hours across the two DST calendars.

On the two changeover weeks each year the hour mapping is briefly not 1:1 (an hour repeats or vanishes); where two Melbourne blocks land on one Toronto hour, the later one wins.

## Development

No build step, no dependencies. Serve the folder over HTTP (`python3 -m http.server`) so the JSON files can be fetched.
