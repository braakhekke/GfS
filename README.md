# GfS: population timelapse for the Genes from Space tool

An external page for the [Genes from Space tool](https://www.gfstool.com/) (project: [UZH](https://www.geo.uzh.ch/en/units/sg/news/genes-from-space.html), [genesfromspace.com](https://www.genesfromspace.com/)). It takes one population polygon and plays a Sentinel-2 timelapse of it: one cloud-free image per year, summer first, with a composite when no single scene is clear. It runs entirely in the browser: no server, no key, no paid service.

**Status: prototype.** Hosted on GitHub Pages: https://braakhekke.github.io/GfS/ (mockup: https://braakhekke.github.io/GfS/mockup/results.html). Not yet connected to the real Results page. Export as video, GIF or PNG is built in.

## Try it locally

```
python3 -m http.server 8123
```
Then open
- `http://localhost:8123/mockup/results.html`: mockup of the tool's Results page; pick a population and open its timelapse, or
- `http://localhost:8123/index.html#bbox=118.05,5.75,118.25,5.95&name=Example`: the page directly with a bounding box.

An internet connection is needed (scene catalogue, imagery and two script libraries from public CDNs).

## What is where

| Path | What |
|---|---|
| `index.html` | the timelapse page (one file) |
| `mockup/results.html` | mockup of the Results page with the "open timelapse" wire (invented polygons) |
| `docs/URL_CONTRACT.md` | link format of the timelapse page |
| `docs/INTEGRATION.md` | proposal for wiring it into the real tool |
| `docs/DECISIONS.md` | every decision, assumption and open question, with evidence |
| `docs/TOOL_WALKTHROUGH.md` | what the tool's pages show (from screenshots) |
| `CLAUDE.md` | instructions and status for the AI assistant and the team |

## Data and credits

Contains modified Copernicus Sentinel data, read from the public AWS Open Data copy through Earth Search (Element 84). The terms of use of these services have not been read yet (DECISIONS Q-08). Mockup map: OpenStreetMap contributors, Leaflet.

## Licence

Not decided yet (the licence of the Genes from Space tool is unknown). Until a licence file is added, no permission to reuse is granted.
