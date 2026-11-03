# What the screenshots show (gfstool.com, example run)

Screenshots are in `screenshots/` (gitignored), numbered in the order they were supplied (taken 2026-11-03). Example species: *Nasalis larvatus* (van Wurmb, 1787); country Malaysia (Borneo).

| # | Page / state | What it shows |
|---|---|---|
| 01 | Input form | Population polygons drawn (3, two red near Sandakan, one blue near Bandar Victoria), "number of Populations: 3", help figure "Polygon creation methods" (observation distance 1 km, population distance 10 km), buttons *Visualize Population polygons*, *Confirm population polygons*, *add polygons to map* |
| 02 | Input form, Step 1 | Data source dropdown (GBIF), species name, GBIF time slider 1900 to 2026 (2010 to 2020), *Select GBIF Range*, observation points as blue circles, *Confirm Observations*, "advanced options" |
| 03 | Input form | Region selection by countries (Malaysia), *Get observations from GBIF*, status "Job submitted... Data loaded successfully!" |
| 04 | Input form | After the GBIF load: *Download Curated Points*, "Group observation by population" dropdown (empty) |
| 05 | Input form | Automated calculation: Buffer in km 0.50, Distance in km 0.00, "What do these distances represent?", *Visualize Population polygons* |
| 06 | Input form | Buffer 10.00, Distance entered 100 (not yet applied) |
| 07 | Input form | Polygons drawn on the map (red = grouped populations, blue = single), buttons *Confirm population polygons*, *add polygons to map* |
| 08 | Input form, Step 2 | "Monitor the population habitats", Baseline year field (empty) |
| 09 | Input form, Step 2 | Baseline year 2010 |
| 10 | Input form, Step 2 | Land Cover Type dropdown: ESA CCI assisted [RECOMMENDED], ESA CCI manual simplified, Tree Cover (Global Forest Watch) |
| 11 | Input form, Step 2 | *Process habitat maps* running (spinner "this can take a few minutes, do not refresh the page...") |
| 12 | Input form, Step 2 | Explanation of datasets (GFW 30 m, 2000 to 2023; ESA CCI 300 m, 1992 to 2021), *Process habitat maps*, *View results* |
| 13 | Input form, Step 2 | Same as 12 (duplicate state) |
| 14 | Results | Habitat map over Borneo (green = tree cover, red = loss), population polygons in purple, two Plotly charts: habitat area per population over time and relative change |
| 15 | Results | Charts (2010 to 2022) and "Estimate effective population size": table (Name, Population_Density, nenc, pop_size, effective_size, habitat_area: 794.69, 763.8, 194.72), default density and Ne:Nc 0.10, *Apply to all populations* |
| 16 | Results | Zoomed map, layer control (base map URL, NC / GAIN / LOSS checkboxes) |
| 17 | Results | Zoomed to the populations; popup on a polygon (name, Population_Density, nenc, pop_size, effective_size) |

Observation: on the Results map the polygons are clickable (popup with the table fields) but nothing links out. That click is the natural place to hang "open this population in a new window" (aim in `CLAUDE.md`).
