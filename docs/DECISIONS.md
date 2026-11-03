# Decisions and open questions

Format: id, date, choice, why, alternatives, evidence, status (proposed / agreed / dropped).

## D-01 Repo location (2026-11-03, agreed by default)
Local repo at `/Users/jbraak/GfS`, a sibling of nothing else: separate from `tonguesfromspace` on purpose. `git init` only; no GitHub remote yet (see Q-01).

## D-02 First deliverable: Sentinel-2 timelapse of the picked polygon (agreed by owner, 2026-11-03)
The leader's request: pick a population on the Results page, then open it in an additional window for a closer look, "quick and dirty". Owner's answers: it must be a **timelapse** (no still, no Copernicus Browser link as a stop-gap); we provide an **external page** (not a change to the Streamlit app); the GitHub repo is **public, under the owner's current account** (`braakhekke`, from the tonguesfromspace remote; to be confirmed when created).

**Imagery rules (owner, 2026-11-03):** one image per year, cloud free; priority is the summer footage of each region; Sentinel-2 L2A; if no clear scene exists, fall back to a quarterly mosaic of the region for the relevant summer season.

## D-03 Standalone timelapse without a worker or paid source (agreed in principle; prototype works)
**Question asked by the owner:** can the timelapse run without a Cloudflare Worker and without paid sources?

**How tonguesfromspace does it** (read in `worker/worker.js`, `js/03-imagery.js`): the page calls a Worker that holds Copernicus Data Space OAuth client id/secret (up to 15 accounts), forwards Sentinel Hub Statistical API requests (scene scoring by SCL cloud/snow) and Process API requests (rendering), and caches in KV. Needs secrets, quota and a paid Cloudflare plan. **Not reusable for GfS.**

**Alternative checked 2026-11-03:** Element 84 *Earth Search* (public STAC, Sentinel-2 L2A as Cloud Optimized GeoTIFFs on the public AWS bucket `sentinel-cogs`).
- `POST https://earth-search.aws.element84.com/v1/search` answers 200 with `access-control-allow-origin: *` (curl, Origin header set).
- A COG asset (`.../TCI.tif`) answers byte-range requests (206) with `Access-Control-Allow-Origin: *`, `Accept-Ranges: bytes`.
- Result: a **static page in the browser** can search scenes and read only the needed pixels of a polygon (e.g. with geotiff.js), no key, no server. This could be one self-contained HTML file hosted on GitHub Pages, opened from the Results page with the polygon passed in the URL.

**Prototype result (2026-11-03):** `index.html` works in headless Chromium: Borneo (5.85 N) and Alps (46.5 N) polygons produce real frames, with scene choice and composites as in D-04. Pages, scenes and the Earth Search STAC were read live; nothing is cached or stored.

**Not yet verified:** terms of use / licence of Earth Search and the bucket (to read, Q-03); the Streamlit side (how to pass a polygon from the Results map to a link, Q-04); performance of reading many scenes in-browser; whether Sentinel-2 imagery is enough for small polygons (population polygons in the example are about 5 to 20 km across, fine at 10 m).

**Alternatives:** (a) Copernicus Browser deep link per polygon (no code, but no timelapse of our own, user must use its UI); (b) Sentinel Hub EO Browser link (same); (c) Google Earth Engine timelapse (needs an account, not "no key"); (d) own worker with CDSE credentials (needs secrets and quota: rejected by the constraints).

Stop-gap link to the Copernicus Browser: rejected by the owner (D-02).

## D-04 Scene choice per year (implemented in `index.html`, 2026-11-03; to be reviewed)
1. **Season** (the "relevant summer"): northern hemisphere June to September; southern December to March (season named after the year it ends in); **tropics (|latitude| < 23.5): there is no summer**, so the three months with the lowest mean tile cloud cover (`eo:cloud_cover` of all catalogue scenes over the polygon, all years) are used, the same months every year so frames are comparable. User can override the months on the page or with `from`/`to` in the link. *Assumption to confirm with the leader: Q-06. The tile cloud cover is a heuristic, not a climate record.*
2. **Single scene first:** scenes of the same day (several MGRS tiles) are merged; up to 10 days per year, lowest tile cloud first, are scored by the **SCL** band inside the polygon only; bad = no data, defective/saturated (1), cloud shadow (3), cloud (8, 9), thin cirrus (10); snow/ice (11) counts as clear. The scene with the least bad pixels is taken if **at most 2 %** are bad. (First try 5 %: visible clouds remained in the Borneo test, so 2 %.)
3. **Fallback composite:** otherwise the **per-pixel median** of the clear pixels of up to 12 best days in the same season window; pixels with no clear value stay dark grey and the share is reported. This is the "quarterly mosaic" of the owner's rule; for N/S hemisphere the window is the 4-month summer, not exactly a calendar quarter. *Question whether that is acceptable: Q-07.*
4. Pictures are the delivered **TCI** (true colour) product, cut to the polygon area, never recoloured. Resolution: 10 m pixel at best, at most 800 px per side (coarser overviews are read for large polygons).
5. First year **2017**: Earth Search has no L2A over Borneo in 2016 (search returned 0 scenes on 2026-11-03); 2017 is sparse there (37 scenes in the whole year).
6. The year range stops at the previous year unless the current season has ended.

Known limits of the prototype: cloud shadow outside SCL's detection can remain; a polygon spanning two UTM zones is not handled (each scene is read in its own CRS, it works, but not tested); heavy polygons (> 100 km across) are read from coarse overviews and are slow; no download/export yet; tested only in headless Chromium, not Safari/Firefox/phone.

## Open questions (for the owner / project leader)

- **Q-01** (answered: public, current account) GitHub repo not created yet: `gh` is not installed on this machine and no credentials are available to the agent. Owner to create `braakhekke/GfS` (public) or install and log in `gh`; then push.
- **Q-02** (answered: timelapse.)
- **Q-03** Licence of the GfS code (unknown) and who owns the tool's repository: do we change the Streamlit app itself (pull request to the GfS team) or only provide an external page they link to?
- **Q-04** Where is the Results page code (Streamlit, Leaflet via streamlit-folium?) and can we see it? Needed to add "select a polygon, then open link".
- **Q-05** (answered: no.)
- **Q-06** Tropical season rule (D-04.1): is "the three least cloudy months" acceptable, or does the leader prefer the species' dry season, or one fixed rule for all populations?
- **Q-07** Is the 4-month summer window for the composite acceptable as the "quarterly mosaic", or should it be exactly one calendar quarter?
- **Q-08** Earth Search / AWS Open Data / Copernicus terms of use: read and quote them before the page is promoted (not yet read).
- **Q-09** How does the Results page get the polygon into the link? Needs the Streamlit/folium code (Q-04); the URL format is in `docs/URL_CONTRACT.md`.
