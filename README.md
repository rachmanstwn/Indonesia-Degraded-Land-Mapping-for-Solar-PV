# Indonesia Degraded Land Mapping for Solar PV

A Google Earth Engine (GEE) analysis to identify spatially degraded and marginal land across Indonesia that is suitable for utility-scale Solar PV deployment — avoiding productive land, forests, and protected areas.

---

## Overview

Indonesia has vast areas of degraded land resulting from historical deforestation, mining, and agricultural abandonment. This project maps those areas and screens them against exclusion criteria to produce a clean, analysis-ready layer of **Solar PV-suitable degraded land**.

The output is an interactive map and a 100 m GeoTIFF raster that can be used in QGIS or downstream analysis.

---

## Repository Contents

| File | Description |
|---|---|
| `GIS_degraded_land.ipynb` | Main analysis notebook (GEE + Python) |
| `indonesia_degraded_land.html` | Interactive Folium/Leaflet map output |

---

## Methodology

### Degraded Land Classification

Four land cover types are identified as degraded:

| Layer | Source | Logic |
|---|---|---|
| Shrubland (degraded forest) | ESA WorldCover 2021 | Class 20 |
| Grassland / Imperata | ESA WorldCover 2021 | Class 30 |
| Bare / sparse vegetation | ESA WorldCover 2021 | Class 60 |
| Deforested, no recovery | Hansen GFC 2023 | Forest loss (2001–2023) AND current cover ≠ forest |

### Exclusion Zones

The following areas are excluded from the final suitable layer:

- **Protected areas** — WDPA (World Database of Protected Areas), filtered to Indonesia
- **Steep terrain** — Slope > 5° derived from SRTM DEM
- **Active land uses** — Cropland, built-up areas
- **Sensitive ecosystems** — Water bodies, herbaceous wetland, mangroves

### Final Layer

`Suitable Degraded Land = (Degraded Land) AND NOT (Any Exclusion Zone)`

---

## Data Sources

| Dataset | Description | Resolution |
|---|---|---|
| [ESA WorldCover 2021 v200](https://esa-worldcover.org) | Global land cover classification | 10 m |
| [Hansen Global Forest Change v1.11](https://glad.earthengine.app/view/global-forest-change) | Deforestation 2001–2023 | 30 m |
| [WDPA](https://www.protectedplanet.net) | Protected area polygons | Vector |
| [SRTM GL1](https://www2.jpl.nasa.gov/srtm/) | Elevation & slope | ~30 m |
| [USDOS LSIB](https://developers.google.com/earth-engine/datasets/catalog/USDOS_LSIB_SIMPLE_2017) | Indonesia country boundary | Vector |

---

## Requirements

```bash
pip install geemap earthengine-api folium pandas
```

A Google Earth Engine account is required. On first run, authenticate with:

```python
ee.Authenticate()
ee.Initialize(project='your-gcp-project-id')
```

---

## How to Run

1. Clone the repository and open `GIS_degraded_land.ipynb` in Jupyter or VS Code.
2. Replace `'rachman-research'` with your own GCP project ID in the `ee.Initialize()` call.
3. Run all cells in order (Steps 0–11).
4. The interactive map renders inline (Step 8) and is also exported as `indonesia_degraded_land.html` (Step 8b).
5. The GeoTIFF is exported to your Google Drive under `GEE_Exports/` (Step 11).

---

## Outputs

| Output | Description |
|---|---|
| `final_degraded` | GEE image — 1 = degraded land suitable for Solar PV |
| `degradation_type` | GEE image — type-coded (1=shrub, 2=grass, 3=bare, 4=deforested) |
| `df_national` | DataFrame — area by degradation category (km² and Mha) |
| `df_islands` | DataFrame — area by major island group |
| GeoTIFF (Google Drive) | 100 m raster, EPSG:4326, ready for QGIS / further analysis |

---

## Interactive Map

`indonesia_degraded_land.html` can be opened in any browser. It includes toggleable layers:

- **ESA WorldCover 2021** — full land cover reference
- **Hansen Forest Loss 2001–2023** — historical deforestation
- **Exclusion Zones** — all excluded areas combined
- **Degraded Land by Type** — pre-exclusion type breakdown
- **★ Suitable Degraded Land (Solar PV)** — final output layer (on by default)

**Map legend:**

| Color | Class |
|---|---|
| 🟠 `#FF4500` | Suitable Degraded Land (final) |
| 🟡 `#e8a838` | Shrubland |
| 🟢 `#78c679` | Grassland / Imperata |
| 🔴 `#d9534f` | Bare Land |
| 🟫 `#8c510a` | Deforested (no recovery) |
| ⬜ `#808080` | Exclusion Zones |

---

## Solar PV Potential Estimates (Preliminary)

The notebook includes rough national-level estimates using the following assumptions:

- **30%** usable area (accounts for equipment spacing, roads, shade clearance)
- **650 Wp** panel size (2.108 m × 1.048 m, typical monocrystalline)
- **4.5 PSH/day** — Indonesia average peak sun hours
- **$0.80/Wp** capital cost (rough estimate)
- CO2 avoidance based on coal plant displacement at 600 t CO2/GWh

> ⚠️ These are order-of-magnitude estimates. Step 2 (planned) will refine these using GHI data from ERA5 or PVGIS.

---

## Next Steps (Step 2)

- Load **GHI (Global Horizontal Irradiance)** from ERA5 or PVGIS
- Overlay with `final_degraded` to compute irradiance on degraded pixels only
- Calculate **technical potential** (MW / GWh) using panel efficiency, GCR, and performance ratio
- Produce province-level potential maps and a final summary table

---

## License

This project is for research purposes. Data sources retain their respective licenses (ESA WorldCover CC-BY 4.0, Hansen GFC open access, WDPA open access).
