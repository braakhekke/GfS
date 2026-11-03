# How the real Results page can offer "look at this population"

Status: **proposal for the GfS team; not tested against the real tool** (we have no access to its Streamlit code, DECISIONS Q-04). The mockup `mockup/results.html` shows the intended behaviour and is tested.

The timelapse page needs only a link: `<page>#geojson=<URL-encoded GeoJSON of the polygon>&name=<population name>` (see `URL_CONTRACT.md`). Hosted address of the page: to be fixed when GitHub Pages is switched on (not yet).

Two ways, depending on how the Results map is built (the popup in the screenshots looks like a Leaflet/folium popup, which is an assumption):

1. **Select box plus link button (Python side).** Add a `st.selectbox` of population names under the map and a link button built from the stored polygon of that population:
   ```python
   import json, urllib.parse
   def timelapse_url(base, polygon_geojson, name):
       return base + "#geojson=" + urllib.parse.quote(json.dumps(polygon_geojson), safe="") + "&name=" + urllib.parse.quote(name, safe="")
   ```
   Show it with `st.link_button` (available in recent Streamlit versions; the version used by the tool has not been checked) or a markdown link with `target="_blank"` via HTML.
2. **Link inside the polygon popup (map side).** Put an `<a target="_blank">` in the popup HTML of each population with the same URL. The mockup does this (JavaScript `timelapseUrl`).

Notes: the polygon must be in longitude/latitude (WGS84, EPSG:4326). Very detailed polygons make long URLs; simplify to a few hundred vertices. The page runs entirely in the visitor's browser and sends nothing to a GfS server.
