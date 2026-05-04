# ⚠️ Limitations

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
- ⚠️ Experiment tracking is partially supported through ClearML, but local run manifests and failure isolation should still be improved
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


### Temporary Validation YAML Fragility

The validation / benchmarking service may generate temporary YAML files to redirect evaluation toward a specific split such as `test/images`.

**Risk:**

- Incorrect path rewriting can generate invalid paths such as `test/images/test/images`.
- Reusing a generic `temp_val.yaml` can reduce idempotency.
- The validation run may silently evaluate the wrong split if the generated YAML is not checked.

**Recommended mitigation:**

- Generate temporary YAML files using a unique `run_id`.
- Validate temporary YAML paths before calling `model.val()`.
- Add unit tests for split redirection logic.
- Store the final resolved YAML path in the validation summary JSON.

---

### Benchmarking Metric Fragility

YOLO validation metrics may be extracted as arrays, especially for per-class metrics.

**Risk:**

- Class order changes can misalign metric values and class names.
- Reports may attach metrics to the wrong class.
- Downstream comparison scripts become fragile.

**Recommended mitigation:**

- Persist per-class metrics as explicit objects with `class_id`, `class_name`, and metric fields.
- Validate array length against `names` and `nc`.
- Include the class dictionary version in validation summaries.

---
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
mode
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
temporary_validation_yaml
clearml_task_id
validation_split
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
