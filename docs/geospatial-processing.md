# 🌍 Geospatial Processing

## 🌟 Overview

The **AgriDrone Vision Evaluation Pipeline** includes a geospatial processing layer that enriches computer vision results with spatial metadata extracted from drone imagery. This layer allows detection outputs to be used in GIS workflows, spatial analysis, field inspection, and precision agriculture reporting.

The geospatial component bridges object detection and geographic analysis by connecting detections, image metadata, GPS coordinates, UTM coordinates, and spatial export formats.

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

## 📈 Related Diagram


![AgriDrone Geospatial Processing](../assets/images/agridrone-vision-evaluation-pipeline-4-shapefile-generation.png)
