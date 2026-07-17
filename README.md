# India Census 2011 Interactive Choropleth Map (Folium)

A Python workflow that joins India's state boundaries with Census 2011 data and renders an interactive, self-contained HTML choropleth map using [`folium`](https://python-visualization.github.io/folium/latest/) — no web server or GIS software required to view the output.

## Overview

This notebook demonstrates an end-to-end pipeline for turning tabular census data into an interactive map:

1. **Load state boundaries** from an open-source India shapefile repository (with a note that Survey of India's own boundary data is recommended over third-party sources for production/official work)
2. **Inspect and validate the CRS**, reprojecting to `EPSG:4326` if needed
3. **Render a baseline interactive map** of state boundaries with hover tooltips and click popups
4. **Pull Census 2011 data** two ways: (a) directly via the [data.gov.in API](https://www.data.gov.in), or (b) from a pre-cleaned Excel extract sourced from the [Wikipedia 2011 Census summary](https://en.wikipedia.org/wiki/2011_census_of_India)
5. **Merge census data onto the boundary GeoDataFrame** by state name, with explicit unmatched-name diagnostics (state-name strings rarely match cleanly across independently-sourced datasets)
6. **Render the final interactive choropleth/popup map** and export it as a standalone HTML file

## Data Sources

| Data | Source |
|---|---|
| State boundaries | [datta07/INDIAN-SHAPEFILES](https://github.com/datta07/INDIAN-SHAPEFILES) (`INDIA/INDIA_STATES.geojson`) — open-source community dataset; Survey of India boundaries recommended for official/production use |
| Census 2011 data | [data.gov.in API](https://www.data.gov.in) *or* a cleaned extract from [Wikipedia's 2011 Census of India summary](https://en.wikipedia.org/wiki/2011_census_of_India) |

## Method Notes

- **State-name matching is the main friction point.** Boundary datasets and census tables rarely use identical state-name strings (casing, `&` vs `and`, UT naming conventions, etc.), so the merge step explicitly reports which states failed to match rather than silently dropping them — this diagnostic step is essential before trusting any downstream choropleth.
- **CRS handling is defensive**: the notebook checks whether a CRS is set at all before assuming `EPSG:4326`, and reprojects only if needed.
- Map centering avoids the `UserWarning` that comes from computing `.centroid` directly on a geographic (lat/lon) CRS, by projecting to a planar CRS (`EPSG:3857`) for the centroid calculation only, then converting back.
- The final map is a single **self-contained HTML file** — `folium`'s `.save()` bundles the base tiles, GeoJSON layer, tooltips, and popups into one file that opens directly in any browser, making it easy to share or embed without a live Python/GIS backend.

## Tech Stack

- `geopandas` — vector boundary handling, CRS management
- `pandas` — census data loading/merging
- `folium` — interactive web map rendering
- `requests` (implicit via `pandas.read_csv`) — data.gov.in API access

## Repository Structure

```
.
├── Creating_an_HTML_page_census_data.ipynb   # Main analysis notebook
├── census.xlsx                                # Cleaned Census 2011 data extract
└── README.md
```

## Outputs

- `india_states_map.html` — baseline interactive state boundary map
- `india_states_map_census.html` — final choropleth/popup map with Census 2011 attributes joined in

## Getting Started

```bash
pip install geopandas pandas folium openpyxl
```

Then open the notebook and run all cells. If using the data.gov.in API option, register for a free personal API key at [data.gov.in](https://www.data.gov.in) rather than relying on the public demo key, which is rate-limited.

## Possible Extensions

- Add proper choropleth fill (colormap per census variable) with a legend, rather than a flat popup-only style
- Support toggling between multiple census variables as separate map layers
- Extend to district-level census data and boundaries for finer-grained analysis (e.g., Uttarakhand/Kerala specific views)
- Cross-reference with other administrative datasets (e.g., LGD codes) for joining against government resource inventories
