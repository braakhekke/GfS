# Decisions and open questions

Format: id, date, choice, why, alternatives, evidence, status (proposed / agreed / dropped).

## D-01 Repo location (2026-11-03, agreed by default)
Local repo at `/Users/jbraak/GfS`, a sibling of nothing else: separate from `tonguesfromspace` on purpose. Remote: https://github.com/braakhekke/GfS (public, created by the owner 2026-11-03; remote `origin` added the same day).

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

## D-05 Results page mockup as the stand-in for the real page (owner, 2026-11-03)
No access to the Streamlit source, so the wire is built on `mockup/results.html`, a hand-made imitation of https://www.gfstool.com/Results from screenshots 14 to 17 (Leaflet map, polygon popups with the real fields, effective-population-size table). **It holds no real data:** three polygons are invented shapes near the locations of the example run; only the `habitat_area` values (794.69, 763.8, 194.72) and the field names are copied from screenshot 15 / 17. A yellow banner on the page says so.
The wire on the mockup: a drop-down "Look at one population closely" plus a button, and an "Open timelapse" button in each polygon's popup; both build `index.html#geojson=<polygon>&name=<name>` (the URL contract) and open it in a **new window**. One function, `timelapseUrl`, does it. Tested 2026-11-03 in headless Chromium: select pop_3, click, a new window opens, parses the polygon and starts scoring scenes.

## D-06 Fixes found while testing the wire (2026-11-03)
- **Search robustness:** the first wire test failed with "Failed to fetch" (CORS error in the console) on the tropics catalogue search. Earth Search answers some requests in a burst with an error that carries no CORS header; `stacPost` now retries up to 4 times with a growing pause. The cause was not established beyond that (curl bursts of 12 requests all returned 200), so it is a workaround, not a diagnosis.
- **Paging:** a year can have more than 200 scenes over a polygon (2018 over Sabah: 243); the page follows the API's `next` links (max 10 pages), de-duplicates by scene id and stops when a page brings nothing new. The API's page tokens were seen to step backwards in date between pages; results were still complete by id.
- **Tropics catalogue search** asks only for id, date and cloud cover (`fields`), to keep the answers small.
- **Season no longer depends on the years shown:** the tropical clearest three months were first computed from the years on screen, so the same polygon gave Mar to May for 2019 to 2021 and Jul to Sep for 2022 to 2023. It now always uses every full year since 2017 (D-04.1 amended). Also seen: for the mock `pop_3`, 2022 and 2023 had no single clear scene and used composites of 10 and 9 days (0.1 % and 0.0 % of the polygon left without a clear pixel).

## D-07 GitHub Pages and export (2026-11-03, owner: "switch on", "build export option, video or gif")
- **Hosting:** GitHub Pages switched on for `main`, folder `/` (legacy "deploy from branch" build, done through the GitHub API with the owner's stored git credential at the owner's request). Addresses: https://braakhekke.github.io/GfS/ (timelapse page) and https://braakhekke.github.io/GfS/mockup/results.html (mockup). The mockup links to `../index.html`, so the wire works on the hosted site too.
- **Export** (buttons under the player, enabled when all years are done): **Video**, **GIF**, **This frame (PNG)**. Years without a picture are left out. Every export frame is the screen frame plus a band with name, year, scene date or composite description and the credit "Contains modified Copernicus Sentinel data · Sentinel-2 L2A". Frame time follows the Speed menu; the last frame is held twice as long.
- **Video:** recorded in the browser with `MediaRecorder` from a canvas, **in real time** (a 10-year film takes about 10 s to record). MP4 when the browser supports it (Chromium 2026 gave `video/mp4`), else WebM; a browser without `MediaRecorder` is told to use the GIF.
- **GIF:** `gifenc` 1.0.3 loaded from jsdelivr when first used, one 256-colour palette per frame (a shared palette bands badly on satellite colours). Files are large (1.2 MB for 3 frames at about 700 x 850 px).
- **Tested** 2026-11-03 in headless Chromium on the Alps polygon (3 years): GIF with 3 frames (`GIF89a`), MP4 372 kB, PNG with the credit band. **Not tested:** Safari, Firefox, phones; playing the MP4 in QuickTime / social apps (no `ffprobe` on this machine to inspect it).

## Open questions (for the owner / project leader)

- **Q-01** (answered 2026-11-03: public repo `braakhekke/GfS`, created by the owner.)
- **Q-02** (answered: timelapse.)
- **Q-03** Licence of the GfS code (unknown) and who owns the tool's repository: do we change the Streamlit app itself (pull request to the GfS team) or only provide an external page they link to?
- **Q-04** (answered 2026-11-03: no access to the Streamlit code. Interim: a mockup of the Results page, D-05.) The real code is still needed to deliver the wire in the tool itself.
- **Q-05** (answered: no.)
- **Q-06** (answered 2026-11-03 by the owner: the tropics rule of D-04.1 is fine **for now**; revisit with the project leader, who may prefer the species' dry season or one fixed rule.)
- **Q-07** (answered 2026-11-03 by the owner: the composite window of D-04.3 is fine **for now**; revisit with the project leader.)
- **Q-08** Earth Search / AWS Open Data / Copernicus terms of use: read and quote them before the page is promoted (not yet read).
- **Q-09** How does the real Results page get the polygon into the link? Needs the Streamlit code (Q-04). Proposed ways for the GfS team: `docs/INTEGRATION.md`.
