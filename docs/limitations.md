# ⚠️ Limitations

## 🔒 Public-Safe Documentation Notice

This document is part of a generalized and anonymized portfolio version of an agricultural computer vision system.

It does **not** include private datasets, client or institutional names, real field coordinates, proprietary model weights, production credentials, unpublished experimental results, internal reports, or operational deployment details.

All project names, dataset names, paths, metric values, coordinates, and identifiers shown here are illustrative, anonymized, or generalized for technical documentation purposes.

## 🌟 Overview

This document describes the known technical, methodological, architectural, and operational limitations of the **AgriDrone Vision Evaluation Pipeline**.

The project is a strong research-grade computer vision and evaluation pipeline, but it should not be represented as a fully productionized distributed MLOps platform unless additional infrastructure, orchestration, testing, and observability layers are implemented.

---

## 📈 Current Maturity Level

**Current maturity classification:**

```text
Advanced prototype / Research-grade engineering pipeline
```

The system is more advanced than a basic proof of concept because it includes:

- ✅ YOLO inference
- ✅ SAHI sliced inference
- ✅ Prediction normalization
- ✅ COCO conversion
- ✅ COCO evaluation
- ✅ Global and per-class metrics
- ✅ Automated reports
- ✅ Geospatial exports
- ✅ Batch processing over image directories

However, it is not yet fully production-ready because it lacks:

- ❌ Formal orchestration
- ❌ Robust retry logic
- ❌ Distributed processing
- ⚠️ Experiment tracking exists through ClearML, but local manifests and tracking isolation still need hardening
- ❌ Automated testing coverage

---

## 🏗️ Architectural Limitations

### Filesystem-Based Coupling

The system relies heavily on local directory structures and file naming conventions.

**Risk:**

- Path changes can break the pipeline.
- Testing becomes harder.
- Deployment to cloud or distributed environments becomes more difficult.

**Recommended mitigation:**

- Introduce a formal configuration layer.
- Use a storage abstraction.
- Support object storage for larger workloads.

---

### Script-Driven Orchestration

The current workflow is primarily executed through local scripts.

**Risk:**

- Execution logic can become tightly coupled.
- Harder to reuse individual modules independently.
- Difficult to schedule, monitor, or scale.

**Recommended mitigation:**

- Refactor into explicit service modules.
- Add CLI commands with clear subcommands.
- Use a configuration-driven pipeline runner.

---

### No Formal Task Queue

There is no formal asynchronous processing layer.

**Missing components:**

- Task queue
- Message broker
- Workers
- Retry queue
- Failure queue
- Scheduled execution

**Risk:**

- Large datasets must be processed synchronously.
- Failures are harder to isolate.
- Long-running jobs are harder to manage.

**Recommended mitigation:**

- Add Celery, RQ, Dramatiq, or a similar task system if production batch processing is required.
- For local research execution, add multiprocessing first before over-engineering into distributed queues.

---

### Distributed Training Is Not a Distributed Production System

The project may use PyTorch DataParallel or Distributed Data Parallel for multi-GPU training. This means the training layer can use distributed GPU execution, but the overall architecture is still not a distributed production platform.

**Risk:**

- The system may be overstated as a distributed system when it is actually a distributed-training-enabled research pipeline.
- DDP subprocesses can introduce debugging and memory complexity without solving orchestration, retries, monitoring, or job scheduling.

**Recommended mitigation:**

- Describe the system as a distributed-training-enabled batch pipeline.
- Avoid claiming production-grade distributed architecture unless queues, workers, APIs, monitoring, and storage abstraction are implemented.

---

### Augmentation Reproducibility Risk

The first run may act as a reproducible baseline, while later runs may use dynamic augmentations.

**Risk:**

- Baseline and augmented runs may be compared incorrectly.
- High-level flags may not fully describe effective Ultralytics augmentation behavior.
- Albumentations and Ultralytics parameters may produce different experimental distributions.

**Recommended mitigation:**

- Persist the complete effective augmentation configuration.
- Mark each run as baseline, augmented, or custom.
- Record augmentation parameters in `summary.json` and run manifests.

---

### Ultralytics Output Folder Collision

Ultralytics may auto-increment output folders when paths collide.

**Risk:**

- Downstream components may read `results` while the actual run was saved to `results2`.
- Model selection and validation may use stale or incorrect artifacts.

**Recommended mitigation:**

- Resolve actual output directories from run metadata.
- Persist `ultralytics_output_dir`.
- Avoid assuming fixed folder names.

---

### CUDA Memory Fragmentation and Large-Model OOM

Large YOLOv11 variants, 2048/4K imagery, high batch sizes, and DDP can trigger CUDA OOM or memory fragmentation.

**Recommended mitigation:**

- Record memory configuration per run.
- Tune batch size and image size by model family.
- Use memory hygiene steps when needed:

```text
torch.cuda.empty_cache()
gc.collect()
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
```

---

### Batch Size Sensitivity

Very small batch sizes may produce unstable or artificially optimistic metrics in some configurations.

**Recommended mitigation:**

- Persist batch size in every metric summary.
- Avoid comparing experiments with different batch sizes without qualification.
- Treat unexpectedly high metrics as requiring qualitative and statistical review.

---

### Project Name and Path Sanitization

Project names and dataset names are used in filesystem paths.

**Risk:**

- Invalid characters can break path creation.
- Similar names can cause folder collision or artifact discovery errors.

**Recommended mitigation:**

- Sanitize project and dataset names before filesystem use.
- Store both raw and sanitized names in the run manifest.

## 📊 Scalability Limitations

### Sequential Batch Processing

The current processing model is primarily batch-oriented and sequential.

**Risk:**

- Runtime increases linearly with dataset size.
- 4K images and SAHI slicing can create significant computational cost.
- Processing large datasets can become slow.

**Recommended mitigation:**

- Parallelize image-level inference.
- Add GPU-aware batching.
- Use multiprocessing for CPU-bound preprocessing and metadata extraction.
- Keep GPU inference controlled to avoid memory exhaustion.

---

### SAHI Computational Overhead

SAHI improves small-object detection but requires multiple inference calls per image.

**Risk:**

- Runtime increases significantly.
- GPU memory and CPU overhead may increase.
- Large overlap ratios can multiply the number of slices.

**Recommended mitigation:**

- Benchmark different `slice_size` and `overlap_ratio` values.
- Record runtime per configuration.
- Compare recall gains against inference cost.

---

### Local Storage Bottlenecks

The system writes many artifacts to disk.

**Risk:**

- Large JSON, image, shapefile, and plot outputs can increase storage usage.
- Slow disks can affect throughput.
- Output directories may become difficult to manage.

**Recommended mitigation:**

- Add output retention policies.
- Compress large artifacts where appropriate.
- Store run outputs under unique `run_id` directories.

---

## 📏 Evaluation Limitations

### Dependency on Annotation Quality

Object detection metrics are only as reliable as the ground truth annotations.

**Risk:**

- Inconsistent bounding boxes distort AP metrics.
- Missing labels can make correct predictions appear as false positives.
- Ambiguous objects can reduce metric reliability.

**Recommended mitigation:**

- Perform annotation QA.
- Track annotation version.
- Review false positives and false negatives visually.

---

### Dataset Imbalance

Agricultural datasets often contain class imbalance.

**Risk:**

- Global metrics may be dominated by frequent classes.
- Rare classes may perform poorly but remain hidden in aggregate results.

**Recommended mitigation:**

- Always report per-class metrics.
- Include class distribution tables.
- Use stratified analysis by class, image condition, and location.

---

### AP50 Can Be Misleading

AP50 is less strict than AP50:95.

**Risk:**

- AP50 may suggest strong performance even when bounding box localization is weak.

**Recommended mitigation:**

- Report AP50 and AP50:95 together.
- Include qualitative examples.
- Review localization errors visually.

---

### maxDets Sensitivity

The system uses:

```text
maxDets = [100, 1000, 3000]
```

**Risk:**

- Different maxDets values can affect evaluation outcomes in dense scenes.
- Comparisons with external benchmarks may require documenting this configuration clearly.

**Recommended mitigation:**

- Always report maxDets values in evaluation summaries.
- Avoid comparing results to external studies unless configurations are compatible.

---

## 🧠 Model and Inference Limitations

### Small-Object Detection Difficulty

Drone images often contain small objects relative to the full image size.

**Risk:**

- Direct YOLO inference may miss small objects after resizing.
- SAHI may improve recall but increase false positives.

**Recommended mitigation:**

- Compare direct YOLO and SAHI runs.
- Tune slice size and overlap ratio.
- Perform class-specific error analysis.

---

### Model Drift

Agricultural environments vary over time.

**Potential sources of drift:**

- Lighting changes
- Crop growth stages
- Seasonal variation
- Camera altitude differences
- Sensor changes
- New field conditions
- Different geographic regions

**Risk:**

- Model performance may degrade outside the training distribution.

**Recommended mitigation:**

- Periodically evaluate on new field data.
- Track dataset versions.
- Monitor performance by date, field, and image condition.

---


### Coupled Inference and Geospatial Export

The operational inference pipeline currently connects detection, visualization, metadata extraction, GeoJSON export, and QGIS CSV generation in a single batch flow.

**Risk:**

- Harder to reuse inference without geospatial export.
- Harder to rerun only geospatial export after metadata fixes.
- Failures in EXIF/GPS extraction can complicate the inference workflow.

**Recommended mitigation:**

- Persist raw detection artifacts before geospatial enrichment.
- Separate inference, visualization, metadata extraction, and spatial export into clearer service boundaries.
- Make geospatial export re-runnable from saved prediction JSON.

---

### Output Idempotency and Timestamp Dependency

Many generated outputs depend on timestamps and dynamically created folders.

**Risk:**

- Reruns may create fragmented or hard-to-compare output folders.
- It may be difficult to determine whether an output already exists.
- Failed batch reprocessing is harder without stable run IDs.

**Recommended mitigation:**

- Use explicit `run_id` values.
- Persist run manifests.
- Store failed-image manifests.
- Support reprocessing from a saved failed-image list.

---

### Inference Resolution and Slice-Size Mismatch

The model may be trained at one resolution but inferred at another resolution or with a SAHI slice size that changes the object scale distribution.

**Risk:**

- Detection quality may degrade.
- Metrics may become difficult to compare across configurations.
- Small-object recall and false positives may change unexpectedly.

**Recommended mitigation:**

- Record training resolution, inference `img_size`, and SAHI `slice_size` in every run summary.
- Compare configurations only when parameters are documented.
- Benchmark direct YOLO versus SAHI under controlled settings.

---

### Confidence Threshold Sensitivity

Confidence thresholds affect the precision-recall trade-off.

**Risk:**

- High thresholds may reduce recall.
- Low thresholds may increase false positives.

**Recommended mitigation:**

- Run confidence threshold sensitivity analysis.
- Report chosen threshold with every experiment.
- Use PR curves for model comparison.

---

## 🌍 Geospatial Limitations

### Incomplete GPS Metadata

Some images may lack GPS or EXIF metadata.

**Risk:**

- Spatial outputs may be incomplete.
- Images without GPS cannot be mapped reliably.

**Recommended mitigation:**

- Generate a geospatial metadata completeness report.
- Mark images with missing metadata explicitly.

---

### Approximate Object Geolocation

Image-level GPS metadata does not automatically provide precise object-level ground coordinates.

**Risk:**

- Detection bounding boxes may be incorrectly interpreted as precise ground locations.

**Recommended mitigation:**

- Treat spatial outputs as approximate unless photogrammetric correction is applied.
- Document assumptions about image footprint and coordinate projection.

---

### AGL and FOV Estimation Uncertainty

AGL and field-of-view calculations depend on altitude, terrain, and camera metadata.

**Risk:**

- Incorrect altitude or missing terrain data can distort estimated coverage.

**Recommended mitigation:**

- Mark AGL and FOV as estimated when appropriate.
- Use DEM data when available.
- Avoid claiming survey-grade accuracy without validation.

---

### Raster Georeferencing and QGIS Integration Limitations

Styled JPEGs require explicit raster georeferencing to be positioned correctly in QGIS.

**Key risks:**

- EXIF GPS metadata is not equivalent to a raster geotransform.
- `.jgw` world files do not store CRS.
- Pixel scale derived from altitude/FOV can be approximate.
- `GPSImgDirection` may be missing or unreliable.
- QGIS may apply `.jgw` implicitly only when the basename matches the raster.
- GeoTIFF fallback depends on GDAL availability.

**Recommended mitigation:**

- Generate `.jgw` files for styled JPEG outputs.
- Persist CRS and world-file units in sidecar metadata.
- Prefer GeoTIFF when CRS ambiguity or archival robustness matters.
- Validate raster placement in QGIS with known reference layers.
- Treat raster placement as approximate unless camera calibration, terrain correction, and CRS handling are validated.

---

### External GIS Tool Dependency

The raster post-processing workflow may depend on system-level tools:

```text
ExifTool
GDAL / OGR
gdal_translate
QGIS / PyQGIS
```

**Risk:**

- Missing binaries can break metadata copying, GeoTIFF conversion, or batch GIS loading.
- These dependencies complicate Conda/Docker environment reproducibility.
- Subprocess calls can become CPU/I/O bottlenecks at large scale.

**Recommended mitigation:**

- Validate external tools before batch execution.
- Containerize GIS runtime dependencies when possible.
- Track per-stage timings for EXIF copy, JGW generation, GDAL conversion, and QGIS loading.
- Separate GPU inference from CPU/GIS post-processing for large production workloads.

---

### World File CRS Ambiguity

World files contain transform coefficients but no coordinate reference system.

**Risk:**

- A `.jgw` generated in degrees can be misinterpreted as meters.
- A UTM-based world file can be loaded under the wrong CRS.
- Raster overlays may appear plausible but spatially wrong.

**Recommended mitigation:**

- Store `crs`, `world_file_units`, `transform_origin`, and `rotation_source` in a sidecar manifest.
- Generate GeoTIFF outputs when unambiguous embedded CRS is required.

### Video Implementation Robustness Limitations

The video processor has additional implementation-level risks beyond tracker instability.

**Key risks:**

- Ultralytics boxes may not always expose the expected `xyxy` or `xywh` structure.
- JSON-loaded label dictionaries may use string keys while YOLO returns integer class IDs.
- Missing class colors or missing fonts can break or degrade rendering.
- NumPy, tensor, set, and `Path` objects can break JSON serialization.
- `VideoWriter` output can become corrupted if resources are not released.
- Excessive debug logging can slow long-video processing.
- Model selection can become ambiguous if the training-results hierarchy changes.

**Recommended mitigation:**

- Use `safe_extract_bbox_info_video` for defensive bbox parsing.
- Normalize label and color dictionaries before inference.
- Define fallback colors and fallback fonts.
- Convert outputs to JSON-safe Python types before persistence.
- Release `VideoCapture` and `VideoWriter` in a `finally` block.
- Throttle logs in normal execution mode.
- Persist model-selection lineage in every video summary.

---

### Video Configuration Drift

The video path depends on project configuration, class labels, colors, model paths, rendering options, and output directories.

**Risk:**

A small mismatch in JSON configuration can produce visually incorrect overlays, failed class lookup, wrong colors, or wrong checkpoint selection.

**Recommended mitigation:**

- validate project configuration before video processing
- normalize JSON keys
- persist raw and resolved configuration
- fail fast on missing required classes
- warn and fallback only for non-critical rendering settings

### Video Tracking and Object Counting Limitations

Video inference depends on temporal tracking behavior.

**Key risks:**

- Unique object counts depend on stable `box.id` values.
- Track IDs may switch during occlusion or re-entry.
- `box.id` may be missing in some frames.
- Per-frame detections can be confused with unique-object counts.
- Long videos can leave partial MP4/JSON/SRT outputs if processing fails mid-run.

**Recommended mitigation:**

- Persist frames with missing tracking IDs.
- Separate frame-level detections from unique tracked-object counts.
- Validate counts visually on sampled video segments.
- Use temporary output paths and promote artifacts only after successful completion.
- Record tracker configuration and Ultralytics version.

---

### Video Rendering and Color-Space Limitations

Video overlays are sensitive to color-space handling.

**Key risks:**

- OpenCV uses BGR frames while PIL commonly uses RGB.
- Native YOLO plotting may not preserve required class colors or custom overlay behavior.
- Incorrect conversion can degrade video color fidelity.

**Recommended mitigation:**

- Define a strict BGR/RGB contract.
- Use custom OpenCV rendering for configured class colors and counters.
- Add sample-frame regression checks.

---

### Video Performance Bottlenecks

Video processing is not only model-inference-bound.

**Potential bottlenecks:**

```text
frame decoding
YOLO tracking
ObjectCounter updates
OpenCV overlay rendering
video encoding
SRT generation
filesystem writes
```

**Recommended mitigation:**

- Track per-stage timings.
- Record effective processing FPS.
- Consider a worker/queue model only after local modularization is stable.

---

### HEIC / HEIF Metadata Extraction Risk

Some drone workflows may include HEIC/HEIF images, where metadata extraction behavior differs from JPG.

**Risk:**

- PIL may fail to read HEIC metadata.
- `pillow-heif` may expose metadata differently.
- ExifTool may be required for reliable GPS extraction.
- Dataset-specific GPS hemisphere defaults can become hidden assumptions.

**Recommended mitigation:**

- Use a format-aware fallback chain.
- Record metadata extraction strategy.
- Configure hemisphere defaults per project rather than hardcoding them.

## 🧪 Reproducibility Limitations

### No Formal Experiment Registry

The system generates metrics and artifacts but does not necessarily enforce experiment tracking.

**Risk:**

- Runs may be difficult to reproduce exactly.
- Model, dataset, and parameter versions may become ambiguous.

**Recommended mitigation:**

Add a `run_manifest.json` file containing:

```text
run_id
model_version
dataset_version
class_dictionary_version
inference_mode
img_size
confidence_threshold
slice_size
overlap_ratio
maxDets
timestamp
output_directory
```

---

### Dependency Version Coupling

The pipeline depends on libraries that may change APIs or behaviors.

**Examples:**

- Ultralytics YOLO
- SAHI
- pycocotools
- PyTorch
- OpenCV
- Rasterio

**Risk:**

- Results or execution behavior may change across environments.

**Recommended mitigation:**

- Pin dependency versions.
- Use `requirements.txt` or `pyproject.toml`.
- Add Docker for reproducible runtime environments.

---

## 📊 Observability Limitations

### Limited Structured Logging

The system includes logs and error handling but may not provide structured operational observability.

**Risk:**

- Harder to diagnose failures across large batch runs.
- Difficult to compute runtime and failure statistics.

**Recommended mitigation:**

Use structured logs such as:

```json
{
  "run_id": "experiment_001",
  "image_id": "image_001",
  "stage": "sahi_inference",
  "status": "success",
  "latency_seconds": 4.12,
  "detections": 58
}
```

---

### No Central Monitoring

There is no monitoring dashboard or alerting system.

**Risk:**

- Long-running failures may not be visible until execution finishes.

**Recommended mitigation:**

- Add progress summaries.
- Export run statistics.
- Track failed images.
- Add dashboard support if productionized.

---

## 🧪 Testing Limitations

The system would benefit from automated tests around critical transformations.

**High-priority test areas:**

- YOLO-to-COCO conversion
- Bounding box coordinate conversion
- Class mapping validation
- Empty prediction handling
- Missing label handling
- GPS metadata parsing
- CSV, GeoJSON, and shapefile export
- JGW world file validation
- GeoTIFF conversion validation
- PyQGIS batch loading validation
- Video tracking ID stability validation
- SRT synchronization validation
- OpenCV color-space rendering validation
- Defensive video bbox extraction validation
- Label/color normalization validation
- JSON-safe serialization validation
- Video resource finalization validation
- COCO evaluation artifact validation

**Recommended mitigation:**

- Add unit tests for conversion functions.
- Add small synthetic datasets for regression testing.
- Add integration tests for end-to-end evaluation.

---

## 🚀 Production Readiness Gaps

To become production-ready, the system would require:

- Configuration management
- Containerized environment
- Dependency pinning
- Automated tests
- Structured logging
- Experiment tracking
- Parallel or distributed processing
- Storage abstraction
- Error retry strategy
- Monitoring and alerting
- Security and access controls if deployed with sensitive data

---

## 🎯 Recommended Prioritization

### High Priority

- Add configuration file support.
- Add run manifest / experiment metadata.
- Add dependency pinning.
- Add validation for COCO artifacts.
- Add structured logging.

### Medium Priority

- Add multiprocessing.
- Add qualitative error galleries.
- Add geospatial completeness report.
- Add automated plots for SAHI vs direct YOLO comparison.

### Lower Priority

- Add distributed workers.
- Add cloud deployment.
- Add task queues.
- Add web dashboard.

**Important note:**

Distributed architecture should not be introduced before local modularization, configuration, and reproducibility are solved. Otherwise, the system risks becoming over-engineered without addressing the core maintainability problems.

---

## 🔬 Implementation-Level Gaps Identified During Architecture Audit

### Multi-GPU Execution Complexity

The project may use single-GPU execution, DataParallel, or DDP. Multi-GPU execution introduces additional complexity around metric collection, checkpoint ownership, seed handling, and CUDA memory behavior.

**Risk:** A training run can complete successfully while metrics are missing, incomplete, or associated with an unexpected output path.

**Recommended mitigation:** Persist execution mode, device list, distributed rank where applicable, resolved seed, checkpoint path, and metric source in each run summary.

---

### Training Metric Fallback Is Not Yet a Formal Contract

The pipeline may recover metrics by validating `best.pt` when `model.train()` returns incomplete or `None` metrics.

**Risk:** Different runs may derive metrics from different sources without that distinction being explicit.

**Recommended mitigation:** Add a `metric_source` field to every summary:

```text
training_return
results_csv
post_training_validation
coco_evaluation
```

---

### Checkpoint Lineage Risk

Multi-run training and dynamic folders increase the risk of validating or deploying the wrong `best.pt`.

**Risk:** Reported metrics may not correspond to the actual model used for inference.

**Recommended mitigation:** Store `training_run_id`, `checkpoint_path`, checkpoint hash or file size, `args.yaml` path, source `results.csv`, and selected metric row in validation and inference outputs.

---

### Dynamic Seed Strategy Requires Explicit Persistence

If seeds are composed from base seed, run index, timestamp, distributed rank, or random component, the run is only reproducible if the resolved seed is persisted.

**Risk:** Experiments may be traceable but not exactly reproducible.

**Recommended mitigation:** Persist both the seed-generation inputs and the final resolved seed.

---

### Model Weight Resolution and Download Behavior

If the system can recover or download missing model weights, that behavior should be documented as an explicit model resolution strategy.

**Risk:** Runs may silently use a fallback base model or downloaded checkpoint that differs from the intended trained artifact.

**Recommended mitigation:** Log weight source, download URL or registry source when applicable, checksum, and whether the model is a base model or trained checkpoint.

---

### Champion Model Selection by Configuration

A single global `best.pt` may not be optimal across image sizes, inference modes, datasets, crop stages, or deployment contexts.

**Risk:** A model selected globally may underperform in a specific operational scenario.

**Recommended mitigation:** Select champion models by configuration scope:

```text
dataset_version
model_family
img_size
inference_mode
slice_size
confidence_threshold
operational objective
```

---

## 📜 Summary

The AgriDrone Vision Evaluation Pipeline is technically strong as an applied ML and computer vision evaluation system. Its main value lies in combining YOLO inference, SAHI sliced inference, COCO metrics, geospatial processing, and automated reporting.

Its main limitations are not in the conceptual model, but in production engineering concerns:

- orchestration
- reproducibility
- scaling
- structured logging
- automated testing
- dependency management

Addressing these areas would move the project from a research-grade evaluation pipeline toward a production-ready ML engineering system.
