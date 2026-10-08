# cloud-mask-composite-images

cloud-mask-composite-images

## What it does

| Step | What it does |
|---|---|
| **Load File** | Load a DataFrame or GeoDataFrame from various file formats. |
| **Set Base Maps** |  |
| **Set Workflow Details** | Add information that will help to differentiate this workflow from another. |
| **Set Groupers** |  |
| **Set Time Range** | Choose the period of time to analyze. |
| **Connect to Google Earth Engine** |  |
| **Fix Invalid Geometries** | Repair invalid geometries (e.g. self-intersecting polygons) in place. |
| **Reproject Gdf** |  |
| **View State From Geodataframes** |  |
| **Export Cloudfree Composites** | Build one cloud-free composite over the ROI for the time range, write it as a GeoTIFF and return its path. |
| **Composite To Bitmap Layer** | Colour one band (or an RGB band combination) of a composite and return it as a map overlay. |
| **Merge Tile Layers** | Merges static base tile layers with a per-group overlay into a single list. |
| **Draw Map** | Creates a map based on the provided layer definitions and configuration. |
| **Persist Text** | Persist text to a file or cloud storage object. |
| **Create Map Widget Single View** | Create a map widget with a single view. |
| **Gather Dashboard** |  |

## Requirements

- Access to a **Google Earth Engine** data source, set up as a connection

---

## 1. Load the Workflow

In the workflow runner, add a new template using this repository's URL. Once added, select
**cloud-mask-composite-images** from the available templates list to load it.

---

## 2. Configure the Workflow

You'll be prompted to fill in the following parameters before running.

### Load File

| Field | Description | Default |
|---|---|---|
| File Path *(required)* | Path to the file to load. Supported formats: .parquet, .geoparquet, .geojson, .json, .gpkg, .csv, .shp | — |

### Set Workflow Details

| Field | Description | Default |
|---|---|---|
| Workflow Name *(required)* |  | — |

### Set Time Range

| Field | Description | Default |
|---|---|---|
| Since *(required)* | The start time | — |
| Until *(required)* | The end time | — |
| Timezone |  | — |

### Connect to Google Earth Engine

| Field | Description | Default |
|---|---|---|
| Data source *(required)* | Select one of your configured data sources. | — |

### Export Cloudfree Composites

| Field | Description | Default |
|---|---|---|
| Sensor | Satellite platform | `Sentinel-2` |
| Cloud Threshold | Maximum scene cloud cover (%) for a scene to be used | `60.0` |
| Reducer | Statistic used to aggregate clear pixels over time | `median` |
| Bands | Reflectance bands to include in the export | `["blue", "green", "red", "nir", "swir1", "swir2"]` |
| Indices | Spectral indices to add as bands | `["NDVI"]` |

### Composite To Bitmap Layer

| Field | Description | Default |
|---|---|---|
| Band | Band to show, e.g. 'NDVI', 'nir' or 'valid_count'; or 'true_color' (red/green/blue) / 'false_color' (nir/red/green). | `NDVI` |
| Palette | A named colormap or custom hex colours (single bands only). Defaults to a colormap suited to the band, e.g. RdYlGn for NDVI. | — |
| Opacity | Layer opacity | `0.8` |

---

## Developing

```bash
# recompile spec.yaml (first time: --install; after that: --update, which bumps VERSION.yaml)
./dev/recompile.sh --update

# run the test cases in test-cases.yaml
pixi run --manifest-path pixi.toml -e default bash dev/pytest-cli.sh cloud_mask_composite_images --all
```

Documentation: [docs/index.html](docs/index.html).
