# Cloud-Free Composite Images Workflow

Builds one cloud-free Sentinel-2 or Landsat image of your area for a chosen period on Google Earth Engine. Clouds and their shadows are masked pixel by pixel. The clear observations are combined into a single composite with optional spectral indices such as NDVI, and the result is saved as a GeoTIFF and shown on an interactive map.

Full documentation: [docs/index.html](docs/index.html) (User Guide, [Technical Guide](docs/technical-guide.html) and Troubleshooting).

## What it produces

- A **cloud-free composite GeoTIFF** of the area: the selected reflectance bands, the selected spectral indices, and a `valid_count` band giving the number of clear observations behind each pixel. 10 m for Sentinel-2, 30 m for Landsat.
- An **interactive map** of one band (NDVI by default) or a true/false-colour view, over Esri topographic and satellite base maps, with a legend.
- A **dashboard** showing the map, with the workflow name and time range.

## How it works

1. **Area.** Reads the area of interest from a vector file, repairs invalid geometries and merges all features into one shape.
2. **Scenes.** On Earth Engine, finds the Sentinel-2 (`COPERNICUS/S2_SR_HARMONIZED`) or Landsat 5/7/8/9 Collection 2 Level-2 scenes over the area in the time range. Scenes cloudier than the threshold are left out.
3. **Cloud mask.** Masks cloud, cloud shadow and cirrus pixels in each scene. Sentinel-2 uses the Scene Classification band (`SCL`). Landsat uses `QA_PIXEL`, and also masks saturated pixels.
4. **Composite.** Reduces the clear pixels over time to one value per pixel (median by default). Adds the chosen spectral indices and a `valid_count` band.
5. **Export.** Downloads the composite tile by tile with `computePixels` into one int16 GeoTIFF in the area's UTM zone.
6. **Map.** Reprojects and downsamples the chosen band, colours it, and lays it over the base maps on the dashboard.

The [Technical Guide](docs/technical-guide.html) covers each stage: masking rules, index formulas, raster encoding, tiling and rendering.

## Requirements

- Access to a **Google Earth Engine** data source, set up as a connection
- A vector file of the **area of interest**, with polygons (GeoParquet, GeoJSON, GeoPackage, Shapefile, …)

> Larger areas take longer: a Sentinel-2 composite of a 100 × 100 km area is about 64 downloads and over a gigabyte before compression. For a first run, try an area of a few tens of kilometres and a range of one to three months.

---

## 1. Load the Workflow

In the workflow runner, add a new template using this repository's URL. Once added, select
**cloud-mask-composite-images** from the available templates list to load it.

---

## 2. Configure the Workflow

The form shows these sections in this order. Fields with a default can be left as they are.

### Set Workflow Details

| Field | Description | Default |
|---|---|---|
| Workflow Name *(required)* | The name shown on the dashboard | — |

### Connect to Google Earth Engine

| Field | Description | Default |
|---|---|---|
| Data source *(required)* | Select one of your configured Earth Engine data sources | — |

### Set Time Range

| Field | Description | Default |
|---|---|---|
| Since *(required)* | Start of the period to composite | — |
| Until *(required)* | End of the period (excluded) | — |
| Timezone | Timezone the two times are in | — |

### Load File

| Field | Description | Default |
|---|---|---|
| File Path *(required)* | Path to the area-of-interest file. Supported formats: .parquet, .geoparquet, .geojson, .json, .gpkg, .csv, .shp | — |

### Export Cloudfree Composites

| Field | Description | Default |
|---|---|---|
| Sensor | `Sentinel-2` (10 m, 2017 onward) or `Landsat` (30 m, Landsat 5/7/8/9 combined, 1984 onward) | `Sentinel-2` |
| Cloud Threshold | Maximum cloud cover (%) for a whole scene to be used. Cloudy pixels in kept scenes are masked anyway | `60.0` |
| Reducer | How each pixel's clear observations are combined: `median`, `mean`, `min` or `max` | `median` |
| Bands | Reflectance bands written to the GeoTIFF | `["blue", "green", "red", "nir", "swir1", "swir2"]` |
| Indices | Spectral indices added as bands: NDVI, EVI, SAVI, MSAVI, NDMI, NDWI, NBR, NDBI, NDSI, BSI | `["NDVI"]` |

### Composite To Bitmap Layer

Controls what is drawn on the map. The GeoTIFF always contains every selected band and index.

| Field | Description | Default |
|---|---|---|
| Band | Band to show, e.g. `NDVI`, `nir` or `valid_count`; or `true_color` (red/green/blue) / `false_color` (nir/red/green). Must be a band selected above | `NDVI` |
| Palette | A named colormap or custom hex colours (single bands only). Defaults to a colormap suited to the band, e.g. RdYlGn for NDVI | — |
| Opacity | Layer opacity, 0–1 | `0.8` |

---

## 3. Outputs

Files are written to the workflow's results folder (`ECOSCOPE_WORKFLOWS_RESULTS`).

| File | Contents |
|---|---|
| `composite_<sensor>_<roi-hash>_<since>_<until>.tif` | The composite. Bands in order: selected bands, selected indices, `valid_count`. Reflectance and index values are stored × 10 000 as int16. No data is `-32768` |
| Map `.html` | The interactive map shown on the dashboard |

The map is a downsampled preview (at most 2048 px on its long side). Use the GeoTIFF for full-resolution values.

---

## 4. Troubleshooting

| Symptom | Likely cause |
|---|---|
| `No Sentinel-2 scenes under …% cloud cover` | No scene passed the threshold. Raise **Cloud Threshold**, widen the time range, or use Landsat before 2017 |
| Holes inside the area | No clear observation there (`valid_count` = 0). Widen the time range or raise the threshold |
| Stripes on a Landsat composite | Landsat 7 scan-line gaps. Widen the range or use 2013 onward |
| `Band(s) [...] not in ...` | The map **Band** wasn't selected under Bands or Indices |
| The run is slow or times out | The area's bounding box is large. Use a smaller area, or Landsat |

More in [docs/troubleshooting.html](docs/troubleshooting.html).

---

## Developing

```bash
# recompile spec.yaml (first time: --install; after that: --update, which bumps VERSION.yaml)
./dev/recompile.sh --update

# run the test cases in test-cases.yaml
pixi run --manifest-path pixi.toml -e default bash dev/pytest-cli.sh cloud_mask_composite_images --all
```

The composite and map tasks (`export_cloudfree_composites`, `composite_to_bitmap_layer`) come from `ecoscope-workflows-ext-mep` 1.0.7.
