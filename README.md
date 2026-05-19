# Park Access Equity in San Antonio

A spatial analysis of park access across San Antonio neighborhoods using open data from the City of San Antonio, the U.S. Census Bureau, and Python geospatial libraries. This project was built as part of [CivicData SA](https://civicdatasa.github.io), a community data initiative focused on civic transparency and equity in San Antonio.

This README is written for students and analysts interested in replicating or building on this work. All data sources are free and publicly available.

---

## What This Analysis Does

This project measures what share of residents in each Census block group live within 0.5 miles of a public park. That distance is a commonly used proxy for walkable park access. Results are visualized as a choropleth map and examined through an equity lens, specifically, whether park access varies by neighborhood income level.

**Key finding:** Lower-income block groups in San Antonio had *higher* average park access (~78%) than higher-income block groups (~40%). This counterintuitive result reflects distinct development patterns: higher-income suburban areas on the north and northwest sides were built with private amenities rather than public parks, while many older, denser, lower-income neighborhoods were developed when public park infrastructure was standard.

---

## Data Sources

All data in this project is free and publically available. No API keys are required except the Census Bureau API (free to obtain).

| Dataset | Source | Format | Notes |
|---|---|---|---|
| City of San Antonio Parks | [San Antonio Open Data Portal](https://data.sanantonio.gov) | Shapefile / GeoJSON | Search "parks" — download the parks shapefile |
| Census Block Group Boundaries | [Census TIGER/Line Shapefiles](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html) | Shapefile | Use `tl_2023_48_bg` (Texas block groups statewide) |
| ACS Demographics (income, population) | [Census Bureau API](https://www.census.gov/data/developers/data-sets/acs-5year.html) | API / CSV | Uses ACS 5-Year estimates; free API key at [api.census.gov](https://api.census.gov/data/key_signup.html) |
| Highway Reference (for map overlay) | [Census TIGER Roads](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html) | Shapefile | Primary and Secondary Roads files for Bexar County |


---

## Methods

### 1. Load and reproject park polygons
Parks data from the City of San Antonio open data portal comes in Texas State Plane South Central (EPSG:2278), with distances measured in US Survey Feet. Buffer distances must be specified in feet to match this CRS.

### 2. Download Census block group data
Use the `censusdata` Python package to pull ACS 5-Year estimates for Bexar County (state FIPS: 48, county FIPS: 029). JOIN to TIGER/Line block group boundaries on GEOID.

### 3. Buffer parks by 0.5 miles (2,640 feet)
Apply a 0.5-mile buffer to each park polygon using `.buffer(2640)` (in feet, matching EPSG:2278). Dissolve overlapping buffers into a single unified access zone using `.dissolve()`.

### 4. Calculate park access per block group
Use `gpd.overlay()` to intersect the dissolved buffer with block group boundaries. For each block group, calculate what share of its area falls within the park access buffer:

```
pct_park_access = intersection_area / total_block_group_area
```

Cap results at  1.0 to handle minor geometric artifacts from reprojection.

### 5. Visualize and interpret
Build a choropleth using GeoPandas + matplotlib. Add a CartoDB Positron basemap via `contextily`. Overlay Loop 410 and Loop 1604 as reference geography. Gray out low-density / rural block groups (below 1,000 people per sq mi) to focus on urban neighborhoods.

---

## Tech Stack

- Python
- `geopandas` - spatial joins, overlay, buffering
- `censusdata` - Census API wrapper
- `contextily` - basemap tiles
- `matplotlib` - choropleth visualization
- `pandas` - data wrangling

Install dependencies:

```bash
pip install geopandas censusdata contextily matplotlib pandas
```

## How to Replicate
 
1. Download the parks shapefile from the [SA Open Data Portal](https://data.sanantonio.gov)
2. Download the Texas block group shapefile from [Census TIGER/Line](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html)
3. Sign up for a free Census API key at [api.census.gov](https://api.census.gov/data/key_signup.html)
4. Run the notebook `Park_Access.ipynb`

---

## Limitations and Caveats
 
- **0.5 miles is an approximation.** Walkability depends on street network connectivity, sidewalk quality, and other factors this analysis does not capture. A block group with 80% coverage may still have poor *actual* walkability if the park is separated by a major highway.
- **This measures proximity, not quality.** A small neighborhood pocket park and a large regional park are treated identically.
- **Block group centroids are not used** — this method measures area coverage, not population-weighted access. A block group could score 60% even if most residents are clustered in the low-access half.
- **Data currency:** Parks data and Census estimates may not reflect very recent development.

---
 
## About CivicData SA
 
CivicData SA is an independent open data project publishing analysis on equity, transit, land use, and economic development in San Antonio. All code and data are open. Feedback and contributions welcome.
