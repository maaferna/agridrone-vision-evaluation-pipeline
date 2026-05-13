# 🗺️ Raster Georeferencing and QGIS Automation Pipeline

## 🔒 Public-Safe Documentation Notice

This document is part of a generalized and anonymized portfolio version of an agricultural computer vision system.

It does **not** include private datasets, client or institutional names, real field coordinates, proprietary model weights, production credentials, unpublished experimental results, internal reports, or operational deployment details.

All project names, dataset names, paths, metric values, coordinates, and identifiers shown here are illustrative, anonymized, or generalized for technical documentation purposes.

> **Purpose:** Convert styled YOLO/SAHI detection images into georeferenced raster artifacts that can be loaded automatically in QGIS, while preserving EXIF/XMP metadata and generating world files or GeoTIFF fallbacks.

---

## 🌟 Overview

The **Raster Georeferencing and QGIS Automation Pipeline** is a post-processing layer of the AgriDrone Vision Evaluation Pipeline.

It extends the object detection and geospatial export workflow by making the generated styled images usable as georeferenced raster layers in QGIS.

This is a separate concern from vector exports such as GeoJSON, CSV, and Shapefile. Vector exports describe detections as spatial features, while this module focuses on the raster image artifact itself:

```text
original drone image
        │
        ▼
YOLO / SAHI inference
        │
        ▼
styled detection image
        │
        ├── copy full EXIF/XMP metadata
        ├── generate .jgw world file
        ├── optionally convert to GeoTIFF
        └── batch-load into QGIS
```

The practical objective is to remove manual GIS ingestion steps after inference.

---

## 🛠️ Component Classification

This component can be classified as:

- **Post-processing service**
- **Raster georeferencing utility**
- **GIS automation module**
- **Filesystem-based batch processor**
- **External-tool integration layer**

It is not a model evaluation component and not a training component. It belongs to the operational post-processing and GIS integration layer.

---

## 🎯 Main Responsibility

The module is responsible for:

- preserving EXIF/XMP metadata in derived styled images
- generating `.jgw` world files for styled JPEG outputs
- computing an affine transform for raster placement
- supporting optional GeoTIFF fallback through GDAL
- enabling batch raster loading in QGIS via PyQGIS
- preserving GIS interoperability after YOLO/SAHI inference
- avoiding manual ImportPhotos or raster registration steps

---

## 📥 Inputs

Typical inputs:

```text
original drone JPEG / HEIC image
styled detection JPEG
EXIF / XMP metadata from original image
GPS latitude / longitude
GPS altitude
GPSImgDirection when available
image width and height
camera/FOV metadata when available
UTM or WGS84 coordinate assumptions
output directory
```

Optional inputs:

```text
DEM / terrain elevation
FOV configuration
GDAL conversion configuration
QGIS project path
```

---

## 📤 Outputs

Generated outputs may include:

```text
*-styled.jpg
*-styled.jgw
*-styled.tif
per-image metadata JSON
QGIS_summary.csv
GeoJSON
Shapefile
QGIS project layers
```

The most important raster artifact pair is:

```text
image-styled.jpg
image-styled.jgw
```

QGIS can load the `.jpg` and apply the `.jgw` implicitly when both files share the same basename.

---

## 🧩 Participating Components

| Component | Role |
|---|---|
| `predict_yolo.py` | Orchestrates inference and post-processing |
| `geo_data_utils.py` | Extracts GPS, converts coordinates, creates geospatial outputs |
| `copy_exif_metadata()` | Copies EXIF/XMP metadata from source image to styled image |
| `generate_jgw_file()` | Generates the world file affine transform |
| ExifTool CLI | Copies complete EXIF/XMP payload |
| GDAL/OGR | Supports GeoTIFF conversion and GIS formats |
| PyQGIS script | Batch-loads georeferenced rasters into QGIS |

---

## 🔄 High-Level Workflow

```text
SAHI / YOLO Inference
        │
        ▼
Styled Image Generation
        │
        ▼
Copy EXIF/XMP Metadata
        │
        ▼
Generate JGW World File
        │
        ├── pixel scale
        ├── rotation
        └── top-left translation
        │
        ▼
Optional GeoTIFF Fallback
        │
        ▼
PyQGIS Batch Loading
```

---

## 🔍 Detailed Workflow

### Step 1: Generate Styled Detection Image

After YOLO or SAHI inference, the pipeline generates a styled JPEG containing:

- bounding boxes
- class labels
- confidence scores
- legends or class summaries

Example:

```text
image_001-styled.jpg
```

This image is useful for visual inspection, but by default it is not a georeferenced raster.

---

### Step 2: Copy Full EXIF/XMP Metadata

The pipeline copies complete EXIF/XMP metadata from the original image to the styled image.

Representative command:

```bash
exiftool -all:all -overwrite_original -TagsFromFile original.jpg styled.jpg
```

Purpose:

- preserve GPS metadata
- preserve camera metadata
- preserve timestamp metadata
- preserve orientation metadata
- improve downstream interoperability

Important distinction:

```text
Metadata preservation is not the same as raster georeferencing.
```

QGIS may not position a JPEG raster using EXIF GPS alone. A world file or GeoTIFF is required for reliable raster placement.

---

### Step 3: Generate `.jgw` World File

The world file stores the affine transform used to position the styled JPEG in map space.

A `.jgw` file contains six lines:

```text
A  pixel size in x direction
D  rotation about y axis
B  rotation about x axis
E  pixel size in y direction, usually negative
C  x coordinate of center of upper-left pixel
F  y coordinate of center of upper-left pixel
```

Conceptual transform:

```text
map_x = A * pixel_x + B * pixel_y + C
map_y = D * pixel_x + E * pixel_y + F
```

The implementation may estimate:

- pixel scale from altitude and field of view
- rotation from `GPSImgDirection` when available
- translation from the coordinate associated with the top-left pixel or image footprint estimate

---

### Step 4: Rotation Handling

If `GPSImgDirection` is available and trusted, it can be used to populate rotation terms.

Recommended policy:

```text
GPSImgDirection available and validated
        → generate rotated world file

GPSImgDirection missing or unreliable
        → generate north-up world file and mark rotation_source = none
```

Rotation should be treated carefully because incorrect heading metadata can misalign the raster.

---

### Step 5: CRS Assumption

World files do not store CRS information.

This is a critical hidden assumption:

```text
A .jgw stores pixel-to-map transform values, but not EPSG code, CRS name, or units.
```

Therefore, every world-file output should be accompanied by explicit CRS metadata in a manifest or sidecar file.

Recommended fields:

```text
world_file_units
crs
source_coordinate_type
target_coordinate_type
pixel_size_x
pixel_size_y
rotation_source
transform_origin
```

If a `.jgw` is generated in degrees but the user assumes UTM meters, the image can be incorrectly positioned.

---

### Step 6: Optional GeoTIFF Fallback

When JPEG + JGW is insufficient or unreliable, the pipeline can convert the styled image to GeoTIFF using GDAL.

Representative tool:

```text
gdal_translate
```

GeoTIFF is often more robust because georeferencing can be embedded in the raster file itself.

Recommended use cases:

- QGIS does not apply the world file as expected
- VRT generation fails
- long-term GIS archival is required
- the workflow needs a single self-contained raster artifact

---

### Step 7: PyQGIS Batch Loading

A PyQGIS script can batch-load all styled JPEGs into QGIS.

Important behavior:

```text
The script should add .jpg files, not .jgw files.
QGIS applies the .jgw automatically when it shares the same basename.
```

Conceptual logic:

```python
for file in output_dir:
    if file.endswith(".jpg"):
        QgsRasterLayer(file, layer_name)
```

The `.jgw` remains a sidecar file and is not loaded as a layer.

---

## ⚠️ Known GIS Integration Failure Modes

| Failure | Cause | Resolution |
|---|---|---|
| ImportPhotos reports unsupported data source | Generated styled image path or format is not accepted by that workflow | Use PyQGIS raster loading |
| `gdalbuildvrt` ignores JPEG files | JPEG lacks usable raster georeferencing | Generate `.jgw` or convert to GeoTIFF |
| EXIF GPS copied but raster does not move in QGIS | EXIF GPS is metadata, not a raster geotransform | Generate `.jgw` or GeoTIFF |
| Raster appears shifted | Pixel scale, CRS, or top-left transform is approximate | Validate CRS, FOV, altitude, and transform origin |
| Raster appears rotated incorrectly | `GPSImgDirection` is missing, noisy, or misinterpreted | Use north-up fallback or validate heading |
| QGIS loads `.jgw` as invalid layer | User/script attempts to load world file directly | Load only `.jpg` or `.tif` raster files |

---

## 📊 Performance Considerations

This stage can be CPU/I/O-bound, not only GPU-bound.

Potential bottlenecks:

```text
styled image rendering
EXIF/XMP copying through ExifTool subprocess
JGW generation
GDAL conversion
GeoJSON/Shapefile writing
PyQGIS layer loading
filesystem I/O
```

Recommended timing fields:

```text
styled_render_time
exif_copy_time
jgw_generation_time
geotiff_conversion_time
geojson_export_time
shapefile_export_time
qgis_load_time
total_postprocess_time
```

These metrics help distinguish YOLO/SAHI inference bottlenecks from GIS post-processing bottlenecks.

---

## 🧪 Recommended Validations

Before or during execution:

- original image exists
- styled image was created
- ExifTool is installed
- EXIF copy completed successfully
- GPS fields are present when spatial export is required
- `.jgw` file exists
- `.jgw` has exactly six numeric lines
- CRS assumptions are persisted
- rotation source is recorded
- GeoTIFF conversion succeeds when enabled
- PyQGIS script loads only raster images

Recommended metadata completeness fields:

```text
GPSLatitude
GPSLongitude
GPSLatitudeRef
GPSLongitudeRef
GPSAltitude
GPSImgDirection
GPSVersionID
GPSStatus
```

---

## 🧾 Recommended Manifest Fields

Each georeferenced raster output should include sidecar metadata:

```json
{
  "run_id": "inference_001",
  "image_name": "image_001.jpg",
  "styled_image": "image_001-styled.jpg",
  "world_file": "image_001-styled.jgw",
  "geotiff": "image_001-styled.tif",
  "metadata_copy": {
    "tool": "exiftool",
    "status": "success"
  },
  "world_file_transform": {
    "pixel_size_x": 0.045,
    "pixel_size_y": -0.045,
    "rotation_x": 0.0,
    "rotation_y": 0.0,
    "top_left_x": "ANONYMIZED_TOP_LEFT_X",
    "top_left_y": "ANONYMIZED_TOP_LEFT_Y",
    "units": "meters",
    "crs": "EPSG:32719",
    "rotation_source": "GPSImgDirection"
  },
  "qgis": {
    "load_strategy": "load_jpg_only_apply_jgw_implicitly",
    "loaded": true
  }
}
```

---

## 🔐 Geospatial Confidentiality Controls

Geospatial outputs can reveal sensitive field locations even when images or code are not published.

The public version must not include:

```text
real coordinates
real shapefiles
real GeoJSON files
real JGW world files
real GeoTIFF rasters
private field names
flight paths
farm or station identifiers
```

When examples are required, use anonymized placeholders or synthetic coordinates that cannot identify a real location.

## 🚨 Risks

### World Files Do Not Encode CRS

Risk:

- Users may interpret degree-based transforms as meter-based UTM transforms.

Mitigation:

- store CRS metadata in sidecar JSON
- generate `.prj` or documentation when applicable
- prefer GeoTIFF when CRS ambiguity is unacceptable

### Approximate Pixel Scale

If pixel scale is estimated from altitude and FOV, raster placement can be approximate.

Mitigation:

- mark transform as estimated
- record altitude/FOV source
- use calibrated camera or orthomosaic when high accuracy is required

### External Tool Dependency

ExifTool, GDAL, and PyQGIS are system-level dependencies.

Mitigation:

- validate tools at startup
- document installation requirements
- containerize GIS runtime when productionized

### Sequential Subprocess Bottleneck

Running ExifTool and GDAL per image can become slow for very large batches.

Mitigation:

- benchmark per-stage timing
- parallelize CPU/I/O stages when safe
- separate GPU inference from post-processing queue in production

---

## 📌 Portfolio Summary

This component demonstrates advanced post-processing and GIS interoperability engineering.

It extends YOLO/SAHI inference beyond vector exports by generating georeferenced styled rasters. The module preserves EXIF/XMP metadata, generates `.jgw` world files, supports optional GeoTIFF fallback, and automates QGIS loading through PyQGIS.

This highlights applied experience in:

- raster georeferencing
- EXIF/XMP metadata preservation
- QGIS automation
- GDAL/OGR integration
- filesystem-based GIS artifact generation
- applied computer vision post-processing for precision agriculture

---

## 🔒 Privacy and Confidentiality Notice

This document describes generalized raster georeferencing and GIS automation patterns.

It should not expose private field coordinates, client names, production paths, proprietary model weights, or sensitive geospatial locations.
