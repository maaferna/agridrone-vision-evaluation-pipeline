# 🌍 Geospatial Processing

## 🌟 Overview

The **AgriDrone Vision Evaluation Pipeline** includes a geospatial processing layer that enriches computer vision results with spatial metadata extracted from drone imagery. This layer allows detection outputs to be used in GIS workflows, spatial analysis, field inspection, and precision agriculture reporting.

The geospatial component bridges object detection and geographic analysis by connecting detections, image metadata, GPS coordinates, UTM coordinates, and spatial export formats.

---


## 🔗 Relationship to the YOLO / SAHI Inference Export Pipeline

The geospatial processing layer is commonly invoked after direct YOLO or SAHI inference has produced detection results.

The inference/export pipeline provides:

```text
styled images
per-image prediction JSON
class summaries
EXIF/GPS metadata
UTM coordinates when available
GeoJSON records
QGIS-compatible CSV summaries
batch summaries
```

This document focuses on the geospatial enrichment and export behavior, while the operational inference flow is documented in:

```text
docs/yolo-sahi-inference-geospatial-export-pipeline.md
```

Important distinction:

```text
The inference pipeline determines what was detected in image space.
The geospatial layer determines how those detections are associated with spatial metadata and GIS-compatible outputs.
```

---

## 🎯 Objectives

The geospatial processing module is designed to:

- 📍 Extract GPS and EXIF metadata from drone images.
- 🌐 Convert geographic coordinates into projected coordinate systems such as UTM.
- 🔗 Associate detection outputs with image-level spatial metadata.
- 🗺️ Generate GIS-compatible output formats.
- 📊 Support downstream spatial analytics and map-based reporting.
- 🧾 Preserve metadata traceability for each processed image.

---

## 📥 Geospatial Inputs

The module may consume the following inputs:

```text
Drone images
EXIF metadata
GPS latitude and longitude
Altitude metadata
Digital elevation model data, when available
Camera metadata
Detection predictions
Image dimensions
Class dictionary
```

**Typical input image formats:**

```text
JPG
PNG
TIF
TIFF
```

The availability of metadata depends on the drone platform, camera configuration, and image export process.

---

## 📊 Metadata Extracted

The system attempts to extract or derive metadata such as:

- Image file name
- Image width
- Image height
- GPS latitude
- GPS longitude
- Altitude
- Camera orientation, when available
- EXIF timestamp, when available
- UTM coordinates
- AGL, when terrain elevation is available
- Estimated field-of-view coverage
- Detection count per image
- Class-level detection summaries

---

## ⚙️ Processing Flow

```text
Drone Image
   │
   ▼
EXIF / GPS Metadata Extraction
   │
   ▼
Coordinate Validation
   │
   ▼
Coordinate Transformation
   │
   ▼
Detection-to-Image Metadata Association
   │
   ▼
Geospatial Export Generation
   │
   ├── Metadata JSON
   ├── CSV
   ├── GeoJSON
   └── Shapefile
```

---

## 🛠️ Step 1: EXIF and GPS Metadata Extraction

The first step reads metadata embedded in drone images.

**Possible metadata sources:**

- EXIF tags
- GPS tags
- Camera metadata
- Drone flight metadata

**Required fields for spatial positioning:**

```text
latitude
longitude
```

**Optional fields:**

```text
altitude
timestamp
camera model
orientation
gimbal angle
focal length
```

**Technical concern:**

Not all drone images preserve complete metadata after export or compression. The system should explicitly handle missing metadata rather than silently producing incorrect spatial outputs.

---

## ✔️ Step 2: Coordinate Validation

Extracted coordinates should be validated before use.

**Validation checks:**

- Latitude is between `-90` and `90`.
- Longitude is between `-180` and `180`.
- GPS values are not null.
- Coordinate format is numeric.
- Metadata belongs to the correct image.

Invalid coordinates should be logged and excluded from strict geospatial outputs, or marked with fallback metadata depending on the workflow configuration.

---

## 🔄 Step 3: UTM Coordinate Conversion

Geographic coordinates may be converted into UTM coordinates for spatial analysis.

**Input:**

```text
latitude, longitude
```

**Output:**

```text
utm_easting
utm_northing
utm_zone
utm_hemisphere
```

UTM coordinates are useful because they represent location in meters, making them more appropriate for distance and area-related calculations than latitude and longitude.

---

## ⛰️ Step 4: AGL and Terrain-Aware Metadata

When altitude and terrain elevation are available, the system can estimate AGL.

```text
AGL = drone_altitude - terrain_elevation
```

AGL means:

```text
Above Ground Level
```

This is useful for estimating image footprint and field-of-view coverage.

**Technical concern:**

AGL calculation depends on reliable altitude metadata and terrain elevation data. If either is missing or inaccurate, AGL should be treated as estimated or unavailable.

---

## 📐 Step 5: Field-of-View Estimation

The system may estimate field-of-view coverage using image metadata, camera parameters, altitude, and derived spatial information.

**Possible outputs:**

- Estimated image footprint
- Approximate coverage area
- Spatial center point
- Detection location approximation

**Important limitation:**

Unless camera calibration, orientation, altitude, and terrain correction are handled rigorously, FOV-based spatial outputs should be interpreted as approximate rather than survey-grade measurements.

---

## 🔗 Step 6: Detection and Metadata Association

Detection outputs are associated with image-level metadata.

**Typical association fields:**

```text
image_id
image_name
class_id
class_name
confidence
bbox
latitude
longitude
utm_easting
utm_northing
altitude
AGL
```

This allows downstream users to connect detections with spatial context.

---

## 📂 Geospatial Output Formats

### 🧾 Metadata JSON

Per-image metadata JSON stores structured metadata for traceability.

**Example structure:**

```json
{
  "image_name": "image_001.jpg",
  "width": 3840,
  "height": 2160,
  "gps": {
    "latitude": -33.12345,
    "longitude": -70.12345
  },
  "utm": {
    "easting": 394000.12,
    "northing": 6330000.45,
    "zone": "19S"
  },
  "detections_count": 42
}
```

---

### 📊 CSV Export

CSV outputs are useful for tabular analysis and integration with data science tools.

**Possible columns:**

```text
image_name
class_id
class_name
confidence
x_center
y_center
width
height
latitude
longitude
utm_easting
utm_northing
altitude
agl
```

---

### 🗺️ GeoJSON Export

GeoJSON supports map-based visualization and GIS workflows.

**Typical feature structure:**

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [-70.12345, -33.12345]
  },
  "properties": {
    "image_name": "image_001.jpg",
    "class_name": "object_class",
    "confidence": 0.91
  }
}
```

---

### 📁 Shapefile Export

Shapefiles support compatibility with traditional GIS tools.

**Generated shapefile components may include:**

```text
.shp
.shx
.dbf
.prj
```

Shapefile export is useful for external analysis in GIS platforms but has stricter field naming and data type limitations than GeoJSON.

---

## 📏 Coordinate System Considerations

**Recommended coordinate systems:**

- WGS84 for latitude/longitude outputs.
- UTM for projected meter-based spatial outputs.

**Important considerations:**

- Always document the coordinate reference system.
- Include CRS metadata in exported spatial files where possible.
- Avoid mixing coordinate systems without explicit transformation.
- Ensure longitude and latitude order is correct in GeoJSON.

**GeoJSON convention:**

```text
[longitude, latitude]
```

---

## 🚨 Error Handling

The geospatial module should handle the following issues:

### Missing GPS Metadata

**Action:**

- Log warning.
- Continue processing image-level detections.
- Exclude from strict spatial export or mark as missing location.

### Invalid Coordinates

**Action:**

- Validate coordinate range.
- Log error.
- Skip spatial feature generation for the affected image.

### Missing DEM or Terrain Elevation

**Action:**

- Use fallback altitude handling.
- Mark AGL as unavailable or estimated.

### Shapefile Export Error

**Action:**

- Log the failed export.
- Preserve CSV and GeoJSON outputs if possible.

---

## ⚠️ Data Quality Risks

### Incomplete Metadata

Drone images may lose EXIF metadata during compression, conversion, or transfer.

### Coordinate Precision

GPS coordinates may not be precise enough for object-level location without additional correction.

### Image Footprint Approximation

Estimating spatial footprint from image metadata can be approximate if camera orientation and terrain are not fully modeled.

### Detection-to-Ground Projection

Bounding boxes in image coordinates do not automatically correspond to precise ground coordinates. Additional photogrammetric processing would be required for high-accuracy object geolocation.

---

## 📂 Recommended Output Directory

```text
outputs/
  metadata/
    image_001.json
    image_002.json

  geospatial/
    detections.csv
    detections.geojson
    detections.shp
    detections.shx
    detections.dbf
    detections.prj

  rasters/
    image_001-styled.jpg
    image_001-styled.jgw
    image_001-styled.tif
```

---

## 📜 Recommended Metadata Manifest

A run-level geospatial manifest can improve traceability.

**Example:**

```json
{
  "run_id": "experiment_001",
  "geospatial_processing": {
    "enabled": true,
    "coordinate_system_input": "WGS84",
    "coordinate_system_projected": "UTM",
    "geojson_export": true,
    "csv_export": true,
    "shapefile_export": true
  },
  "summary": {
    "images_processed": 120,
    "images_with_gps": 114,
    "images_without_gps": 6
  }
}
```

---

## 🧾 HEIC / HEIF Metadata Extraction

Some drone image workflows may include HEIC or HEIF images. Metadata extraction for these formats can require a fallback chain because support differs across libraries.

Recommended extraction strategy:

```text
1. ExifTool, when installed
2. pillow-heif metadata access
3. PIL / ExifTags fallback when supported
```

Recommended output fields:

```text
metadata_extraction_strategy
metadata_extraction_status
metadata_extraction_error
```

Important caution:

```text
Dataset-specific hemisphere defaults such as S/W should be treated as configuration, not global assumptions.
```

If GPS reference fields are missing, the pipeline should avoid silently applying geographic defaults unless they are explicitly configured for the project.

---

## 🗺️ Raster Georeferencing for Styled Images

In addition to vector outputs such as GeoJSON, CSV, and Shapefile, the pipeline may generate georeferenced styled raster images.

This is necessary because a styled JPEG with copied EXIF GPS metadata is not automatically interpreted by QGIS as a georeferenced raster.

Recommended artifact pair:

```text
image-styled.jpg
image-styled.jgw
```

The `.jgw` file stores the affine transform used by GIS software to position the JPEG.

World-file parameters:

```text
A = pixel size in x direction
D = rotation about y axis
B = rotation about x axis
E = pixel size in y direction, usually negative
C = x coordinate of center of upper-left pixel
F = y coordinate of center of upper-left pixel
```

Important constraint:

```text
World files do not store CRS.
```

Therefore, each raster-georeferenced output should include sidecar metadata documenting:

```text
crs
world_file_units
pixel_size_x
pixel_size_y
rotation_source
transform_origin
altitude_source
fov_source
```

Recommended detailed document:

```text
docs/raster-georeferencing-qgis-automation-pipeline.md
```

---

## 🧾 EXIF/XMP Preservation

The pipeline may preserve metadata in styled images by copying complete EXIF/XMP metadata from the source image using ExifTool.

Representative behavior:

```text
copy_exif_metadata()
exiftool -all:all source.jpg styled.jpg
```

This preserves original camera, GPS, timestamp, and orientation metadata in the derived visual artifact.

Important distinction:

```text
Copying EXIF/XMP preserves metadata, but it does not replace raster georeferencing.
```

QGIS raster placement should rely on `.jgw` or GeoTIFF georeferencing.

---

## 🧭 GPS Metadata Completeness

GPS parsing should validate not only latitude and longitude, but also supporting fields when available.

Recommended fields:

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

The pipeline may extract GPS in DMS format, convert it to decimal degrees, and then convert decimal coordinates to UTM.

---

## 🛰️ Optional GeoTIFF Fallback

If JPEG + JGW is insufficient for GIS interoperability, the pipeline may generate GeoTIFF outputs using GDAL.

Recommended use cases:

- QGIS does not apply `.jgw` as expected.
- `gdalbuildvrt` ignores JPEG files due to missing georeferencing.
- A single self-contained raster artifact is preferred.
- Long-term GIS archival is required.

---

## 🧰 PyQGIS Batch Loading

A PyQGIS script can automate loading georeferenced styled rasters into QGIS.

Important behavior:

```text
Load only .jpg or .tif raster files.
Do not load .jgw files directly.
QGIS applies .jgw implicitly when it shares the raster basename.
```

This eliminates manual loading steps after inference and post-processing.

---

## ⚠️ Known GIS Integration Failure Modes

| Failure | Cause | Resolution |
|---|---|---|
| ImportPhotos reports unsupported data source | Generated styled image is not accepted by that workflow | Use PyQGIS raster loading |
| `gdalbuildvrt` ignores JPEG | Raster lacks geotransform | Generate `.jgw` or GeoTIFF |
| EXIF GPS copied but raster does not move in QGIS | EXIF GPS is metadata, not geotransform | Generate world file or GeoTIFF |
| Raster appears shifted | CRS, FOV, altitude, or transform origin is approximate | Persist assumptions and validate placement |
| `.jgw` loaded as invalid layer | World file was loaded directly | Load the `.jpg` or `.tif` instead |

## ⬆️ Recommended Improvements

### 1. Explicit CRS Handling

Include CRS metadata in all spatial outputs and documentation.

### 2. Geospatial Validation Report

Generate a report showing:

- Images with valid GPS.
- Images missing GPS.
- Coordinate range summary.
- Exported spatial feature counts.

### 3. Better Object-Level Geolocation

For more precise object geolocation, consider:

- Camera calibration.
- Ground control points.
- Orthomosaic generation.
- Photogrammetric correction.
- Drone attitude metadata.

### 4. Integration with GIS Tools

Potential integrations:

- QGIS
- PostGIS
- GeoPandas
- Rasterio
- GDAL

### 5. Spatial Database Support

For larger datasets, consider storing spatial outputs in:

```text
PostgreSQL + PostGIS
```

---

## 📝 Interpretation Note

The geospatial outputs generated by this pipeline should be treated as spatially enriched detection artifacts, not necessarily as survey-grade geolocation outputs. Accuracy depends on drone metadata quality, camera parameters, terrain model availability, and spatial transformation assumptions.

---

## 🧠 Implementation Notes for Re-Runnable Spatial Export

The geospatial export should ideally be re-runnable from saved prediction JSON rather than requiring model inference to be executed again.

Recommended separation:

```text
raw detections / normalized predictions
        ↓
metadata enrichment
        ↓
coordinate transformation
        ↓
GeoJSON / CSV / Shapefile export
```

This separation is important because EXIF parsing, CRS assumptions, DEM lookup, and shapefile schema issues may need to be fixed independently from inference.

Recommended metadata fields for traceability:

```text
run_id
image_id
prediction_source
model_checkpoint
inference_mode
gps_available
crs_source
crs_target
spatial_export_status
```

---

## 📈 Related Diagram


![AgriDrone Geospatial Processing](../assets/images/agridrone-vision-evaluation-pipeline-4-shapefile-generation.png)
