# CLAUDE.md: GfS (Genes from Space tool, user-experience work)

Working repo for improving the user experience of the **Genes from Space (GfS) tool** (https://www.gfstool.com/). Completely separate from the `tonguesfromspace` project; nothing is shared except that we read its code for ideas (see `docs/DECISIONS.md`).

Several people work here. Everything that is decided, found or done is written down in this repo (this file, `docs/`), so others can follow it. Work cleanly, state the evidence, say what was not checked.

## Links

- Project page (UZH): https://www.geo.uzh.ch/en/units/sg/news/genes-from-space.html
- Project site: https://www.genesfromspace.com/
- The tool (Streamlit app, pages Methods / Input form / Results): https://www.gfstool.com/

## Aim (task from the project leader)

Original (German): *"Ich möchte, dass eine Population ausgewählt werden kann und dann extern diese Population (also diese Region) genauer anzuschauen. 'Quick and dirty' Link zu einem zusätzlichen Fenster."*

Formalised (English): On the **Results** page, let the user pick one population (one polygon). After picking, offer a link that opens an **additional window** in which that population's region can be looked at in more detail. "Quick and dirty" is explicitly acceptable for the first version.

Working interpretation (to be confirmed with the project leader, see `docs/DECISIONS.md` D-02): the extra window shows a **Sentinel-2 timelapse** of the selected polygon.

### Constraints

- GfS lives on GitHub; **no paid services**, no account or secret that must be kept (so no Copernicus Data Space credentials, no paid hosting tier).
- A **fully standalone** static page (`index.html`, no worker, no server) that takes a polygon in the URL and shows the timelapse. Imagery rules: one cloud-free Sentinel-2 L2A image per year, summer first, fallback composite (D-02, D-04).
- Owner's decisions 2026-11-03: timelapse (not a still), external page (not a change to the Streamlit app), public repo under the owner's current GitHub account.
- Licence of this repo is **not yet decided** (unknown); do not add a LICENSE file or claim one.

## How the tool works (from the screenshots in `screenshots/`, gitignored)

Streamlit app with a left menu (Methods, Input form, Results), language switch en/sp, "powered by Bon in a Box". Details: `docs/TOOL_WALKTHROUGH.md`.

1. **Input form, Step 1: determine populations.** Choose a data source: own observation points, GBIF (species name + year range slider + region by drawn box or countries), or own polygons. Observations show as blue circles on a Leaflet map; points can be removed, then *Confirm Observations*. Group by population: automatic (Buffer km = observation distance, Distance km = population distance; *Visualize Population polygons*, then *Confirm population polygons*) or manual (draw polygons). Example: *Nasalis larvatus*, Malaysia, buffer 10 km, distance 100 km gives 3 populations.
2. **Input form, Step 2: monitor habitat.** Baseline year, land-cover dataset (ESA CCI assisted/manual, or Global Forest Watch tree cover), *Process habitat maps* (minutes), *View results*.
3. **Results page.** Map of habitat gain/loss/no change with the population polygons (popup per polygon: name, Population_Density, nenc, pop_size, effective_size), two Plotly charts (habitat area and relative change per population over time), an editable table for effective population size (Ne) and defaults for density and Ne:Nc.

The user-experience gap: the results map shows the polygons but there is no way to pick one and look at that place in recent imagery.

## Rules

- Say what was verified and what was not. Test results are reported as they are, failures included.
- External facts (URLs, numbers, API behaviour) come from reading the source, not from memory; record the evidence in `docs/DECISIONS.md`.
- No paid or key-requiring service without the team agreeing first.
- Screenshots stay out of git (`screenshots/` is in `.gitignore`); they are numbered in the order given and described in `docs/TOOL_WALKTHROUGH.md`.
- Do not push, create the GitHub remote, or contact the GfS team without the owner's go-ahead.
- Every non-trivial choice gets an entry in `docs/DECISIONS.md` (id, date, choice, why, alternatives, evidence, status). Open questions go to the "Open questions" list there.

## Files

| Path | What |
|---|---|
| `CLAUDE.md` | this file: aim, constraints, rules |
| `mockup/results.html` | mockup of the Results page with the wire (invented polygons, D-05) |
| `docs/INTEGRATION.md` | proposal for wiring into the real tool |
| `index.html` | the standalone timelapse page (all code in one file; libraries proj4 2.9.0 and geotiff 2.1.3 from CDNs) |
| `docs/DECISIONS.md` | decision log and open questions |
| `docs/URL_CONTRACT.md` | how to link to the page with a polygon |
| `docs/TOOL_WALKTHROUGH.md` | what each screenshot shows |
| `screenshots/` | gitignored, 01 to 17 |

## Status log

- 2026-11-03: repo created, aim written down, feasibility of a worker-free Sentinel-2 timelapse checked (D-03).
- 2026-11-03: prototype `index.html` built and tested in headless Chromium (Borneo bbox 2019 to 2021 and Alps bbox 2021 to 2023): scene choice, composite fallback, tropics season rule (D-04). 

## How to run locally

`python3 -m http.server 8123` in the repo root, then open `http://localhost:8123/index.html#bbox=118.05,5.75,118.25,5.95&name=Example`. There is no build step and no test suite yet; the checks so far were manual runs (see D-04).
- 2026-11-03: owner created the public repo `braakhekke/GfS`; README, `.gitignore`, `docs/INTEGRATION.md` written. Owner accepted the tropics rule and the composite window for now (Q-06, Q-07). No Streamlit access: Results page mockup built and the wire tested (D-05); search retries, paging and a fixed tropics season found and fixed while testing (D-06). Not done: GitHub Pages, terms of use (Q-08), the wire in the real tool (Q-09), tests in Safari/Firefox/phones, export of the timelapse.
- 2026-11-03: GitHub Pages switched on (https://braakhekke.github.io/GfS/); export as video / GIF / PNG added to `index.html` (D-07). Still open: terms of use (Q-08), the wire in the real tool (Q-09), Safari/Firefox/phone tests, playback test of the MP4 in other players.
- 2026-11-03: faster scoring (about 1.5 to 2 times, D-08), loading display with per-year chips and counter, per-browser saved pictures (IndexedDB), selectable years, Alps test polygon in the mockup. Raise `CACHE_VERSION` in `index.html` when scoring or rendering changes.
- 2026-11-03: scene table can delete a year from the timelapse and switch a composite year to its best single scene (D-09); `CACHE_VERSION` is `v2`.
- 2026-11-03: first year is looked up per polygon from Earth Search (D-10), no longer a constant.
- 2026-11-03: mockup gets a Turkey polygon and centre markers; export progress bar; cleaner page styling; scenes with s3:// asset links are skipped (D-12).
- 2026-11-03: exports have a Quality menu; High re-reads the scenes at 10 m (D-13).
- 2026-11-06: NIR false-colour view, RGB / Both / NIR switch, shared zoom and pan, measuring (line, area, erase) in one control row with Play/Speed (D-15); `CACHE_VERSION` is `v3`.
- 2026-11-03: measured where load time goes and wrote the caching/free-polygon proposal (D-14, docs/CACHING_OPTIONS.md); nothing built.
