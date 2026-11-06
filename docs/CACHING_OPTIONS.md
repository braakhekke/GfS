# Free drawing of polygons: how to keep it fast (proposal, nothing of this is built yet)

Status: **proposal for discussion** (owner question 2026-11-03: "total freedom to draw own polygons; requests to Earth Search are slow; think of cache or storage options"). Numbers below are measurements of the current page unless marked *estimate*.

## 1. The problem

With free polygons a cache keyed by polygon (what the page has now, per browser) almost never hits for a new user or a new drawing. The work per population is the same each time: find the scenes, judge clouds, read pixels.

## 2. Where the time goes (measured 2026-11-03, headless Chromium, empty cache, 3 years each, `from`/`to` fixed)

| Case | Wall time | Earth Search (STAC) requests | Range requests to the image files | Downloaded |
|---|---|---|---|---|
| Borneo box 16 km, composites needed | 24.8 s | 7 | 195 | 93 MB |
| Turkey box 40 km, single clear scenes | 21.4 s | 7 | 130 | 52 MB |

Per call (averages, calls run in parallel so the sums are larger than the wall time): STAC search 1.2 to 1.6 s (3 calls, about 4 s in total); cloud-mask scoring of one acquisition day 1.2 s (Borneo, 36 calls) to 4.4 s (Turkey, 12 calls); reading one day's picture at 10 m 6 to 12 s (24 and 3 calls).

Reading: **the catalogue search is not the bottleneck. The time is in reading pixels from the image files (about 17 to 31 MB per year, 40 to 65 requests per year) and, second, in scoring days by cloud.** Composites are the heaviest case (8 days read per year). A cache therefore has to avoid pixel reads and scoring, not just searches.

## 3. Options, cheapest first

| # | Option | Helps | Needs | Verdict |
|---|---|---|---|---|
| A | **Browser cache** (built): finished years per polygon, 30 days | the same user, same polygon | nothing | keep |
| B | **Snap to a fixed grid, cache by cell, not by polygon**: cut the area into fixed cells (e.g. 5 km, aligned to a lattice) and cache `cell + year + season rule` in the browser; a slightly different polygon in the same place re-uses cells | same user, redrawn or edited polygons | page code only | **do next** |
| C | **Progressive loading**: show every year first from a coarse overview of the image files (80 to 160 m, a fraction of the bytes), refine to 10 m afterwards or only for exports | first picture in a few seconds for everyone | page code only | **do next** |
| D | **A shared "cloud index", built ahead and served as static files**: for each Sentinel-2 tile and day a coarse cloud-fraction grid (say 5 km cells) from the SCL mask, written to JSON by a GitHub Action and published on GitHub Pages. A polygon then picks its best days by looking up the index (no scoring reads), and only the chosen scenes' pixels are read | everyone, removes the scoring step (about a third of the time) and decides composites without reading | a scheduled GitHub Action (free for public repos), repo or Pages space | **main step if a shared speed-up is wanted** |
| E | **A shared cache of finished pictures** (tiles of rendered yearly images) | everyone, removes pixel reads too: the biggest saving | somewhere that visitors can *write*: GitHub cannot take anonymous writes; a free tier of an object store plus a tiny function would. Free-plan limits are tight (the TSF project's records say it moved to a paid Cloudflare plan after hitting the free-plan KV write limit); **terms and limits must be read before choosing** | possible, needs a team decision and credentials held somewhere |
| F | **Pre-render by the team**: a GitHub Action renders the yearly tiles for the regions GfS cares about (known species ranges, countries) and publishes them as static files; the page uses them when the polygon lies inside, otherwise it computes | instant for covered regions | Action minutes, repo size limits (large for many regions) | good for a fixed set of regions, not for "anywhere" |

Dead ends, with reasons: server-side rendering on a free function (CPU limits are far below what reading and compositing pixels needs; the TSF project's relay only forwards requests, it does not process pixels); Microsoft Planetary Computer (its data API sends no CORS headers, per the TSF reference, so a browser page cannot call it).

## 4. Recommended path

1. **Now, no infrastructure:** B (cell-keyed browser cache) and C (progressive loading). Together they make a first picture appear fast and make edits cheap.
2. **If speed for first-time users still matters:** D, the cloud index. Size *estimate*: 22 x 22 cells of 1 byte x about 30 days is roughly 15 kB per tile and year, so roughly 150 MB per 1,000 tiles for ten years; whole-world coverage is large, so build it for the regions the team names and grow it. It must be refreshed (a weekly Action) because new scenes arrive every few days. Whether the Action can read the image files fast enough for a large area has **not been tested**.
3. **Only with a decision by the GfS team:** E or F for finished pictures.

## 5. Other things free polygons need (not specific to caching)

- **Drawing:** built in the mockup on 2026-11-06 (Leaflet-Geoman, polygon and rectangle, one area at a time, D-21).
- **Limits:** the mockup allows at most 60 × 60 km for a drawn area (D-21); the earlier note: a maximum area (a 100 x 100 km polygon at 10 m is 100 million pixels; the page already falls back to coarser pixels, but reading gets heavy), a vertex limit (simplify to a few hundred), and a clear message when a polygon is mostly sea (Earth Search has no scenes over open ocean, tested).
- **Hand-over:** a long polygon makes a long link; for drawn polygons pass them through `sessionStorage` or `postMessage` between the Results page and the timelapse window (same site only) or keep the link for short ones.
- **Politeness to the public services:** many users reading the same public bucket is what it is for, but the Earth Search catalogue is a shared service; keep requests few (the page makes about 7 per run) and retry with backoff (done).
