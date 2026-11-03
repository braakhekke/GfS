# Link format of the timelapse page

The page reads everything from the URL **hash** (after `#`), so nothing is sent to a server.

| Parameter | Meaning |
|---|---|
| `geojson` | URL-encoded GeoJSON (Polygon, MultiPolygon, Feature or FeatureCollection), WGS84 longitude/latitude |
| `bbox` | `west,south,east,north` in degrees, used when `geojson` is absent |
| `name` | label shown in the header (e.g. `pop_1`) |
| `y0`, `y1` | first and last year (default 2017 and the last year whose season has ended) |
| `from`, `to` | season months 1 to 12 (default: automatic, see DECISIONS D-04.1) |

Example: `index.html#bbox=118.05,5.75,118.25,5.95&name=Example%20population`

From JavaScript (e.g. a Leaflet popup on the Results map): `base + '#geojson=' + encodeURIComponent(JSON.stringify(layer.toGeoJSON())) + '&name=' + encodeURIComponent(name)` and open it with `target="_blank"`. Long polygons make long URLs (browsers cope with some 100 kB; simplify polygons with more than a few thousand vertices).
