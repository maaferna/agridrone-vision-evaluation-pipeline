# 🔍 YOLO / SAHI Inference and Geospatial Export Pipeline

> **Purpose:** Execute direct YOLO or SAHI inference on high-resolution agricultural drone imagery, generate styled detection outputs, enrich predictions with EXIF/GPS metadata, and export JSON, CSV, GeoJSON, and QGIS-compatible artifacts.

---

## 🌟 Overview

The **YOLO / SAHI Inference and Geospatial Export Pipeline** is a core runtime component of the `agridrone-vision-evaluation-pipeline` project.

Its purpose is to transform agricultural drone images into operational detection and geospatial artifacts. It executes model inference using either direct YOLO inference or SAHI sliced inference, reconstructs detections into full-image coordinates, generates annotated visualizations, extracts geospatial metadata, and exports structured outputs for GIS analysis and downstream evaluation.

This component sits between the CLI orchestration layer and the geospatial / evaluation layers.

```text
CLI Orchestrator
        │
        ▼
YOLO / SAHI Inference and Geospatial Export Pipeline
        │
        ├── Styled Images
        ├── Per-image JSON
        ├── EXIF/GPS Metadata
        ├── UTM Coordinates
        ├── GeoJSON
        ├── QGIS CSV
        └── Batch Summaries
```

> SAHI is used to enable efficient inference on high-resolution imagery with dense object distributions, improving detection performance without exceeding GPU memory constraints.

---

## 🛠️ Component Classification

This component can be classified as:

- **Data processor**
- **Service layer**
- **Batch processing component**
- **Core inference runtime module**
- **Geospatial export bridge**

It is part of the core runtime workflow because it directly processes images, executes model inference, and produces structured detection outputs.

---

## 🎯 Main Responsibility

The pipeline is responsible for:

- selecting the trained `best.pt` checkpoint
- loading YOLO or SAHI-compatible inference configuration
- processing single images, directories, or video inputs
- executing direct YOLO inference
- executing SAHI sliced inference for high-resolution imagery
- reconstructing detections from slices into full-image coordinates
- consolidating detections by class
- generating styled images with bounding boxes and legends
- extracting EXIF/GPS metadata
- converting GPS coordinates to UTM when possible
- exporting per-image JSON metadata and detections
- generating GeoJSON outputs
- generating QGIS-compatible CSV summaries
- generating batch-level JSON and CSV summaries
- optionally saving cropped detected objects

---

## 📥 Inputs

The pipeline consumes:

```text
individual images
image directories
trained YOLO model best.pt
img_size
slice_size
overlap_ratio
confidence_threshold
class dictionary
class color map
output directories
EXIF/GPS metadata embedded in images
```

Optional inputs:

```text
video files
model selection configuration
ClearML context
DEM / terrain elevation source
```

---

## 📤 Outputs

The pipeline generates:

```text
styled images with bounding boxes
per-image JSON metadata
per-image prediction JSON
GeoJSON per image or batch
QGIS-compatible CSV summary
batch summary JSON
batch summary CSV
optional object crops
georeferenced styled rasters
JGW world files
optional GeoTIFF outputs
```

Example output layout:

```text
outputs/
├── styled-images/
├── JSON_metadata/
├── predictions/
├── geospatial/
│   ├── detections.geojson
│   ├── qgis_summary.csv
│   └── shapefile_outputs/
└── batch_summary.json
```

---

## 🔄 High-Level Workflow

```text
Input Image / Directory
        │
        ▼
Load best.pt Model
        │
        ▼
Select Direct YOLO or SAHI Mode
        │
        ▼
Run Inference
        │
        ▼
Reconstruct / Normalize Detections
        │
        ▼
Generate Styled Image
        │
        ▼
Extract EXIF/GPS Metadata
        │
        ▼
Convert GPS to UTM
        │
        ▼
Export JSON / CSV / GeoJSON
        │
        ▼
Batch Summary Outputs
```

---

## 🔍 Detailed Workflow

### Step 1: Model Selection

The pipeline selects the model checkpoint according to the current configuration.

Typical model artifact:

```text
best.pt
```

This model may be selected manually or through a model-selection utility that evaluates prior training results.

---

### Step 2: Inference Mode Selection

The pipeline supports two inference modes.

#### Direct YOLO Inference

Best suited for:

- smaller images
- faster execution
- larger or visually clear objects
- simple inference workflows

#### SAHI Sliced Inference

Best suited for:

- high-resolution drone imagery
- small-object detection
- dense scenes
- 4K images where resizing may remove important detail

SAHI introduces additional cost because each image is split into multiple slices and each slice triggers inference.

---

### Step 3: SAHI Slicing and Reconstruction

When SAHI mode is enabled:

1. the image is divided into overlapping slices
2. YOLO inference is executed per slice
3. slice-level detections are transformed back into full-image coordinates
4. overlapping detections are consolidated
5. final predictions are normalized

Technical concern:

```text
slice_size and overlap_ratio strongly affect recall, duplicate detections, runtime, and GPU memory usage.
```

---

### Step 4: Detection Consolidation

The system consolidates detections by:

- class ID
- class name
- confidence
- bounding box coordinates
- per-image counts
- optional class-level summaries

This enables both visual inspection and downstream tabular analysis.

---

### Step 5: Styled Image Generation

The pipeline can generate annotated visual outputs containing:

- bounding boxes
- class labels
- confidence scores
- class-specific colors
- image-level summary legends

These outputs are useful for qualitative inspection, validation, reporting, and stakeholder communication.

---

### Step 6: EXIF/GPS Metadata Extraction

The pipeline extracts image metadata where available.

Possible fields:

```text
latitude
longitude
altitude
timestamp
camera model
image width
image height
orientation
```

If EXIF/GPS metadata is missing or corrupted, the system should still preserve non-spatial inference outputs and explicitly mark spatial fields as unavailable.

---

### Step 7: UTM Conversion

When valid GPS coordinates are available, the system converts latitude and longitude into projected UTM coordinates.

Possible fields:

```text
utm_easting
utm_northing
utm_zone
utm_hemisphere
```

UTM coordinates are useful for GIS workflows because they provide meter-based spatial references.

---

### Step 8: JSON Export

Per-image JSON outputs may include:

```json
{
  "image_name": "image_001.jpg",
  "model": {
    "weights": "best.pt",
    "img_size": 1280,
    "confidence_threshold": 0.25
  },
  "detections": [
    {
      "class_id": 0,
      "class_name": "object",
      "confidence": 0.91,
      "bbox": [120, 230, 340, 460]
    }
  ],
  "metadata": {
    "latitude": -33.12345,
    "longitude": -70.12345,
    "utm_easting": 394000.12,
    "utm_northing": 6330000.45
  }
}
```

---

### Step 9: GeoJSON and QGIS CSV Export

GeoJSON outputs support spatial visualization and downstream GIS workflows.

QGIS-compatible CSV summaries may include:

```text
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
```

These outputs allow detections to be inspected outside the ML pipeline in spatial analysis tools.

---

### Step 10: Batch Summary Generation

For directory-level execution, the pipeline generates consolidated summaries.

Batch summaries may include:

- number of processed images
- number of failed images
- total detections
- detections by class
- images with GPS metadata
- images without GPS metadata
- generated output paths
- processing duration

---

### Step 10B: Video Inference Outputs

When video inference is enabled, the pipeline may generate:

```text
processed video files
frame-level detection metadata
.srt subtitle files
```

The `.srt` output can preserve temporal detection summaries or frame-level annotations for review. These outputs should include model, run, and timestamp lineage because they are not equivalent to static image predictions.

### Step 10D: Dedicated Video Tracking Path

Video inference is handled through a dedicated temporal processing path rather than as a simple static-image extension.

Recommended detailed document:

```text
docs/yolo-video-inference-object-tracking-processor.md
```

The video path uses:

```text
OpenCV VideoCapture
Ultralytics model.track()
tracking IDs from box.id
ObjectCounter
OpenCV VideoWriter
optional SRT frame summaries
```

Important distinction:

```text
Frame-level detections are not the same as unique tracked-object counts.
```

Unique counts depend on tracker ID stability. Missing or unstable `box.id` values should be recorded in the output summary.

Recommended video outputs:

```text
annotated_video.mp4
tracking_summary.json
frame_level_detections.srt
processing_log.json
```

### Step 10E: Video Implementation Contracts

The dedicated video path should include defensive processing contracts:

```text
safe_extract_bbox_info_video
    handles xyxy / xywh differences
    records invalid or missing bounding boxes

label/color normalization
    converts JSON string keys to integer class IDs
    applies fallback colors when required

adaptive rendering
    scales font size and line thickness by frame resolution
    preserves BGR/RGB consistency

JSON-safe persistence
    converts NumPy/tensor/path/set values before writing JSON

resource finalization
    releases VideoCapture and VideoWriter explicitly
```

These contracts prevent common failures in long-running video inference jobs.

### Step 10C: Raster Georeferencing and QGIS Automation

When styled JPEG outputs need to be loaded as georeferenced rasters in QGIS, the pipeline can generate a raster georeferencing artifact pair:

```text
image-styled.jpg
image-styled.jgw
```

This workflow includes:

1. Copy full EXIF/XMP metadata from the original image to the styled JPEG.
2. Generate a `.jgw` world file with six affine transform parameters.
3. Use pixel scale derived from altitude/FOV when available, or a documented fallback.
4. Use `GPSImgDirection` for rotation only when present and trusted.
5. Persist CRS assumptions because `.jgw` files do not store CRS.
6. Optionally convert the styled JPEG to GeoTIFF using GDAL.
7. Use PyQGIS to batch-load `.jpg` files while QGIS applies the `.jgw` implicitly.

Important caution:

```text
EXIF GPS metadata alone is not reliable raster georeferencing for QGIS. Styled JPEGs require a world file or GeoTIFF to be positioned as raster layers.
```

Recommended detailed document:

```text
docs/raster-georeferencing-qgis-automation-pipeline.md
```

## 🧩 Participating Components

```text
CLI Orchestrator
Model Selection Utility
YOLO Inference Service
SAHI Inference Service
Video Tracking Processor
Object Counter
Frame Annotation Renderer
SRT Artifact Generator
Visualization Utilities
Geospatial Processing Layer
Filesystem Artifact Store
ClearML Tracking Integration
```

Representative modules:

```text
scripts/main.py
scripts/predict_yolo.py
scripts/utils.py
scripts/geo_data_utils.py
scripts/clearml_utils.py
```

---

## 🧰 Dependencies

Core dependencies:

- Python
- Ultralytics YOLO
- SAHI
- PyTorch
- CUDA / GPU
- OpenCV
- Pillow
- NumPy
- Pandas
- PyProj / UTM
- JSON / CSV
- ExifTool CLI
- GDAL / OGR
- PyQGIS / QGIS runtime
- local filesystem

External consumers / supporting tools:

- QGIS
- ClearML
- Roboflow / CVAT as dataset preparation tools

---

## 🚨 Technical Risks

### SAHI Performance Bottleneck

SAHI increases runtime because each image is split into multiple inference slices.

Mitigation:

- benchmark multiple `slice_size` and `overlap_ratio` values
- compare recall gains against runtime cost
- log runtime per image and per batch

### GPU Memory Pressure

High-resolution images, large YOLO models, and SAHI can increase GPU memory usage.

Mitigation:

- tune `img_size`, `batch_size`, and model size
- monitor CUDA memory usage
- clear cache between large processing stages when appropriate

### Configuration Mismatch

Using a model trained at one resolution and inferred at another may degrade performance.

Mitigation:

- record training and inference resolution
- validate `img_size` and `slice_size` against experiment metadata
- compare performance across resolution configurations

### EXIF/GPS Dependency

Spatial outputs depend on image metadata availability.

Mitigation:

- continue non-spatial inference when metadata is missing
- generate metadata completeness reports
- mark spatial fields as unavailable instead of silently failing

### Coupling Between Inference and Geospatial Export

Inference and geospatial export are currently closely connected.

Mitigation:

- define clear interfaces between prediction generation and geospatial enrichment
- persist raw predictions before geospatial transformation
- allow geospatial export to be rerun independently

### Limited Idempotency

Outputs may depend on timestamps and dynamically generated folders.

Mitigation:

- introduce stable `run_id`
- write run manifests
- make output overwrite behavior explicit

### Video Tracking ID Instability

Video unique-object counts depend on stable YOLO tracking IDs.

Risk:

- `box.id` may be missing in some frames.
- Track IDs may switch after occlusion.
- The same object may receive multiple IDs.
- Unique counts may be inflated or undercounted.

Mitigation:

- separate frame-level detections from unique tracked-object counts
- record frames with missing IDs
- validate tracker behavior on sampled clips
- persist tracker configuration and Ultralytics version

### Video Rendering and Color-Space Risk

The video processor uses custom OpenCV rendering to preserve configured class colors and avoid visual degradation.

Risk:

- OpenCV uses BGR frames while PIL commonly uses RGB.
- Incorrect conversion can alter video colors.
- Native YOLO plotting may not support the required custom overlays.

Mitigation:

- define an explicit BGR/RGB contract
- render overlays manually when necessary
- validate sample frames visually

### Partial Video Artifact Risk

Long videos can fail after partially writing outputs.

Mitigation:

- write MP4, JSON, and SRT to temporary paths
- promote outputs only after successful completion
- persist job status as `complete`, `partial_failed`, or `failed`

### Raster Georeferencing Assumptions

Styled JPEG rasters can be georeferenced through `.jgw` world files, but world files do not encode CRS.

Mitigation:

- persist CRS metadata in sidecar JSON
- record whether world-file units are degrees or meters
- prefer GeoTIFF when CRS ambiguity is unacceptable
- validate raster placement in QGIS

### External GIS Tool Dependency

EXIF/XMP copying, GeoTIFF fallback, and QGIS automation depend on external tools such as ExifTool, GDAL/OGR, and PyQGIS.

Mitigation:

- validate binaries and Python bindings before batch execution
- document installation requirements
- isolate subprocess failures from core inference outputs
- persist failed post-processing stages in a manifest

### Project Name and Output Path Sanitization

Because output folders are generated from project names, dataset names, image size, model family, timestamps, and run identifiers, names should be sanitized before filesystem use.

Mitigation:

- store raw project name and sanitized project name
- avoid invalid path characters
- use stable `run_id`
- persist actual output directory

### No Formal Retry Logic

Failed images are logged or skipped, but no formal retry queue exists.

Mitigation:

- persist failed image lists
- support reprocessing failed inputs
- add retry only for non-deterministic I/O failures

---

## 🧠 Additional Engineering Notes

### Resolution and Slice-Size Compatibility

The pipeline should record both training and inference scale parameters:

```text
train_img_size
inference_img_size
sahi_slice_size
overlap_ratio
```

This matters because a model trained at one resolution may behave differently when direct inference or SAHI inference changes the effective object scale.

### Raw Prediction Persistence Before Geospatial Enrichment

The pipeline should persist raw or normalized prediction artifacts before EXIF/GPS enrichment. This makes geospatial export re-runnable if metadata parsing, CRS assumptions, or shapefile generation must be corrected later.

### Failed Image Manifest

For batch inference, the system should persist a manifest such as:

```json
{
  "run_id": "inference_001",
  "failed_images": [
    {
      "image": "image_042.jpg",
      "stage": "exif_parsing",
      "error": "missing GPS metadata"
    }
  ]
}
```

This is especially important because the current workflow has no formal retry queue.

### Model Selection Context

If the pipeline resolves `best.pt` automatically, it should persist why that checkpoint was selected, including the source metric, source run, and selection score.

---

## 🧪 Recommended Validations

Before execution:

- `best.pt` exists
- image path or directory exists
- image extensions are supported
- CUDA device is available when selected
- class dictionary matches model classes
- output directories are writable
- SAHI parameters are valid
- EXIF/GPS metadata handling is configured
- CRS assumptions are documented

After execution:

- styled images were generated
- JSON outputs are parseable
- GeoJSON is valid
- CSV contains expected columns
- UTM fields exist only when GPS is valid
- batch summaries match processed image counts

---

## 📊 Maturity Assessment

Current maturity:

```text
High-maturity research inference and geospatial export component
```

Strengths:

- supports direct YOLO and SAHI inference
- handles high-resolution imagery
- generates visual, tabular, JSON, and geospatial outputs
- integrates with broader validation and evaluation workflows
- supports GIS-compatible analysis

Limitations:

- synchronous batch execution
- filesystem-coupled
- no formal retry queue
- tight coupling between inference and geospatial export
- metadata-dependent spatial outputs
- limited production observability

---

## 📌 Portfolio Summary

This component demonstrates the implementation of an applied inference and geospatial export pipeline for agricultural computer vision.

It connects YOLO/SAHI object detection with metadata extraction, UTM coordinate conversion, styled visualizations, JSON outputs, GeoJSON generation, QGIS-compatible CSV summaries, and batch-level reporting.

The component highlights applied experience in:

- YOLO inference
- SAHI sliced inference
- high-resolution image processing
- GPU-aware batch execution
- geospatial metadata extraction
- GIS-compatible output generation
- agricultural computer vision workflows

---

## 🔒 Privacy and Confidentiality Notice

This documentation describes the technical architecture and methodology of an inference and geospatial export pipeline in a generalized form.

It does not expose private datasets, client names, sensitive field locations, production credentials, private model weights, internal filesystem paths, or unpublished experimental results.

Any examples included in a public repository should be anonymized, synthetic, or non-sensitive.
