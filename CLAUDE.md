# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, Dutch-language family babysitting schedule ("Oppasplanning") for the children Otto & Rosie, deployed via GitHub Pages at **babysit.ottorosie.com** (see `CNAME`). There is no build system, no package manager, no tests, and no dependencies beyond Google Fonts loaded from a CDN.

- `index.html` — the entire app: inline CSS and vanilla JS. Renders the schedule in two views (cards and table) with a per-person filter.
- `data.json` — the schedule data, fetched at runtime with a cache-busting query string (`data.json?v=Date.now()`).

## Development workflow

There are no build, lint, or test commands. To preview locally, serve the directory over HTTP (the `fetch('data.json')` call fails on `file://`):

```bash
python3 -m http.server 8000
```

Deployment is automatic: pushing to `main` publishes via GitHub Pages.

## Data model (`data.json`)

Top-level: `{ "rows": [...], "camps": [...] }`.

Each entry in `rows` is one day, sorted chronologically:

- `date` (`YYYY-MM-DD`), `day` (capitalized Dutch day name, e.g. `"Woensdag"` — weekend detection compares against `"Zaterdag"`/`"Zondag"` exactly), `dd`, `mm`
- Three slot objects `ochtend`, `dag`, `avond`, each `{ "v": <display text or null>, "c": <color key> }`. Valid `c` keys: `lieze`, `pieter`, `renilde`, `linda`, `kamp`, `weekend`, `none`.
- Per-person task fields (string or `null`): `lieze`, `pieter`, `omaopa` (tasks for Linda & Dirk), `omieopie` (tasks for Renilde & Ronny), `liezewerk` (Lieze's work hours), `opm` (a warning note, rendered in red).
- `camps` entries are `{ "date", "otto" }` and add a "Kamp" note to matching days.

## Key conventions in `index.html`

- The `PEOPLE` object maps person keys to labels, CSS variables, and initials: `lieze` (mama), `pieter` (papa), `renilde` (Renilde & Ronny, omie & opie), `linda` (Linda & Dirk, oma & opa). Each person key has a matching pair of CSS custom properties (`--<key>` and `--<key>-ink`) used by both the card and table views.
- `TASK_OWNER` links persona filters to their data.json task fields — note the asymmetry: filter `linda` owns field `omaopa`, filter `renilde` owns field `omieopie`, filter `lieze` owns both `lieze` and `liezewerk`.
- The month is hardcoded in several places: the `<title>`, the card view's "juli" label, and the header. When creating a new month's schedule, update these along with `data.json`.
- All UI text and data values are in Dutch; keep it that way.
- The page auto-scrolls to today's card/row on first paint (or the first upcoming date if today is outside the schedule range).
