# 📊 Evaluation

## 🌟 Overview

This document describes the evaluation strategy used in the **AgriDrone Vision Evaluation Pipeline**. The system evaluates object detection models on high-resolution drone imagery using standardized COCO metrics through `pycocotools`.

The evaluation process converts both ground truth annotations and model predictions from YOLO format into COCO-compatible JSON files, then computes global and per-class detection metrics.

---

## 🎯 Evaluation Objectives

The evaluation module is designed to:

- 📏 Measure object detection performance on agricultural drone imagery.
- 🔄 Compare direct YOLO inference against SAHI sliced inference.
- 📈 Generate reproducible COCO metrics.
- 📂 Report both global and per-class performance.
- 📝 Export metrics in JSON, CSV, and visual formats.
- 📚 Support scientific analysis and technical documentation.

---

## 📥 Evaluation Inputs

The evaluation process requires the following inputs:

```text
Drone images
YOLO ground truth labels
YOLO prediction files
Class dictionary
Image metadata
Evaluation configuration
```

**Typical input structure:**

```text
data/
  images/
    image_001.jpg
    image_002.jpg

  labels/
    image_001.txt
    image_002.txt

outputs/
  predictions/
    image_001.txt
    image_002.txt

config/
  classes.json
```

---

## 🏷️ YOLO Ground Truth Format

Ground truth annotations are expected in normalized YOLO format:

```text
<class_id> <x_center> <y_center> <width> <height>
```

Where:

- `class_id` is the numeric class identifier.
- `x_center` is the normalized horizontal center of the bounding box.
- `y_center` is the normalized vertical center of the bounding box.
- `width` is the normalized box width.
- `height` is the normalized box height.

All coordinate values are relative to the image dimensions and should be between `0` and `1`.

---

## 🔮 YOLO Prediction Format

Prediction files include confidence scores:

```text
<class_id> <x_center> <y_center> <width> <height> <confidence>
```

Where:

- `confidence` is the model confidence score for the detection.

The confidence score is required for COCO detection evaluation because detections are ranked by confidence.

---

## 📂 COCO Conversion

The evaluation module converts YOLO files into COCO JSON files.

**Generated files:**

```text
gt_coco.json
pred_coco.json
```

### 📁 Ground Truth COCO Structure

The ground truth COCO file includes:

- `images`
- `annotations`
- `categories`

Required fields include:

```text
image_id
file_name
width
height
category_id
bbox
area
iscrowd
```

### 📂 Prediction COCO Structure

The prediction COCO file includes a list of detection records with:

```text
image_id
category_id
bbox
score
```

COCO bounding box format:

```text
[x_min, y_min, width, height]
```

This differs from YOLO normalized center-based coordinates, so conversion must be carefully validated.

---

## ⚙️ Evaluation Engine

The pipeline uses:

```text
pycocotools
```

The evaluation follows COCO-style object detection methodology based on IoU thresholds and ranked detections.

Configured `maxDets`:

```text
[100, 1000, 3000]
```

This configuration is important for high-resolution drone imagery, where a single image may contain many objects. Standard COCO defaults can be too restrictive for dense agricultural scenes.

---

## 📊 Main Metrics

### AP50

AP50 measures Average Precision at an IoU threshold of `0.50`.

It is useful for understanding whether the model is detecting objects with moderately correct localization.

**Interpretation:**

- Higher AP50 indicates stronger detection quality.
- AP50 is less strict than AP50:95.
- It may look optimistic when bounding boxes are not highly precise.

---

### AP50:95

AP50:95 measures Average Precision across multiple IoU thresholds from `0.50` to `0.95`.

It is stricter than AP50 and better reflects localization quality.

**Interpretation:**

- Lower than AP50 in most practical systems.
- Sensitive to bounding box quality.
- Useful for comparing models under stricter localization requirements.

---

### Precision

Precision measures the proportion of predicted detections that are correct.

```text
Precision = True Positives / (True Positives + False Positives)
```

High precision means the model generates fewer false detections.

---

### Recall

Recall measures the proportion of ground truth objects that were detected.

```text
Recall = True Positives / (True Positives + False Negatives)
```

High recall means the model misses fewer objects.

---

### F1-Score

F1-score combines precision and recall into a single metric.

```text
F1 = 2 * (Precision * Recall) / (Precision + Recall)
```

F1 is useful when the trade-off between false positives and false negatives matters.

---

## Per-Class Evaluation

The pipeline computes metrics per class to identify uneven model performance.

Per-class metrics include:

- AP50
- AP50:95
- Precision
- Recall
- F1-score

Per-class evaluation is critical in agricultural imagery because class imbalance is common. A model may perform well on frequent classes while failing on rare or visually ambiguous classes.

---

## Global Evaluation

Global metrics aggregate performance across classes.

Global outputs include:

```text
global_metrics.json
metrics.csv
summary plots
```

Global metrics are useful for comparing experimental runs, but they should not be interpreted without reviewing per-class results.

> In addition to standard validation splits, models are evaluated on independently labeled high-resolution (4K) imagery to assess generalization under real-world deployment conditions, where image resolution and object density differ from the training data.

---

## SAHI vs Direct YOLO Evaluation

The pipeline can evaluate and compare two inference strategies:

### Direct YOLO

**Expected characteristics:**

- Lower runtime.
- Simpler output structure.
- Fewer duplicate detections.
- Potentially weaker small-object recall.

### SAHI

**Expected characteristics:**

- Higher runtime.
- Better small-object sensitivity.
- Improved recall in some scenarios.
- More duplicate detection risk.
- More complex post-processing.

**Recommended comparison metrics:**

```text
AP50
AP50:95
Precision
Recall
F1-score
Per-class AP
Runtime per image
Number of predictions
False positive rate
False negative rate
```

---

## 📂 Evaluation Outputs

The evaluation module generates structured outputs for analysis and reporting.

**Expected outputs:**

```text
outputs/
  coco/
    gt_coco.json
    pred_coco.json

  metrics/
    eval_metrics_coco.json
    global_metrics.json
    per_class_metrics.json
    metrics.csv

  plots/
    precision.png
    recall.png
    f1_score.png
    ap50.png
    ap5095.png
```

---

## Recommended Metrics JSON Structure

**Example global metrics structure:**

```json
{
  "run_id": "experiment_001",
  "inference_mode": "sahi",
  "model": "trained_model.pt",
  "dataset": "drone_dataset_v1",
  "metrics": {
    "AP50": 0.82,
    "AP50_95": 0.57,
    "precision": 0.79,
    "recall": 0.75,
    "f1_score": 0.77
  }
}
```

**Example per-class metrics structure:**

```json
{
  "class_name": {
    "AP50": 0.84,
    "AP50_95": 0.61,
    "precision": 0.80,
    "recall": 0.78,
    "f1_score": 0.79
  }
}
```

---

## Validation Checks Before Evaluation

Before running COCO evaluation, the system should validate:

- All images have valid dimensions.
- All ground truth class IDs exist in the class dictionary.
- Prediction files are correctly formatted.
- Bounding boxes are within valid coordinate ranges.
- COCO JSON files contain valid `images`, `annotations`, and `categories` sections.
- Prediction JSON contains valid `score` values.
- Image IDs match between ground truth and prediction files.

---

## Common Evaluation Failure Modes

### ❌ Invalid COCO JSON

**Cause:**

- Missing required fields.
- Incorrect data types.
- Invalid image IDs or category IDs.

**Impact:**

- `pycocotools` may fail or produce invalid results.

---

### 🚫 Class Mapping Mismatch

**Cause:**

- Model predictions use class IDs not present in the evaluation dictionary.

**Impact:**

- Incorrect per-class metrics.
- Missed detections in evaluation.

---

### 🔄 Bounding Box Conversion Error

**Cause:**

- Incorrect conversion between YOLO normalized coordinates and COCO absolute coordinates.

**Impact:**

- Severely distorted AP metrics.

---

### ⚠️ Excessive Duplicate Detections

**Cause:**

- SAHI overlap or NMS settings are not tuned.

**Impact:**

- Precision decreases.
- False positives increase.

---

### 🧐 Annotation Ambiguity

**Cause:**

- Objects are partially visible, overlapping, or inconsistently labeled.

**Impact:**

- Metrics may not reflect true model capability.

---

## Interpretation Guidelines

### AP50 is not enough

AP50 can be useful, but it may overstate performance when localization quality is weak. AP50:95 should be reviewed for stricter evaluation.

### Global metrics can hide class-level failures

Always review per-class metrics, especially for imbalanced datasets.

### Recall matters for detection coverage

In agricultural monitoring, missing relevant objects may be more problematic than generating some false positives, depending on the use case.

### Precision matters for operational trust

If detections trigger downstream decisions, false positives can reduce trust in the system.

### SAHI should be evaluated with runtime

Improved recall is not automatically better if runtime becomes operationally impractical.

---

## Recommended Evaluation Report Sections

A complete evaluation report should include:

1. Dataset summary
2. Model version
3. Inference mode
4. Inference parameters
5. COCO configuration
6. Global metrics
7. Per-class metrics
8. SAHI vs direct YOLO comparison
9. Error analysis
10. Qualitative examples
11. Limitations
12. Recommendations

---


## YOLO Dataset Validation and Benchmarking Service

In addition to COCO-based evaluation, the project includes a YOLO-native validation and benchmarking component based on Ultralytics `model.val()`.

This service is used to validate trained YOLO models directly against agricultural datasets while controlling runtime and reproducibility factors such as:

- model checkpoint selection (`best.pt`)
- dataset split redirection through temporary YAML generation
- training metadata recovery from `args.yaml`
- GPU warm-up before timing measurements
- CUDA cache cleanup between runs
- repeated validation runs
- global and per-class metric extraction from `results.box`
- average time per image
- standard deviation across runs
- ClearML logging
- structured JSON summary persistence

Recommended detailed document:

```text
docs/yolo-dataset-validation-benchmarking-service.md
```

### Relationship to COCO Evaluation

| YOLO Validation / Benchmarking | COCO Evaluation |
|---|---|
| Uses Ultralytics `model.val()` | Uses `pycocotools` |
| Useful for YOLO-native validation | Useful for standardized COCO-style evaluation |
| Extracts metrics from `results.box` | Computes metrics from converted COCO JSON artifacts |
| Can include GPU timing measurements | Focuses primarily on detection quality metrics |
| Integrates with ClearML tracking | Integrates with JSON/CSV/plot reporting |

Both evaluation paths are useful, but they should not be treated as identical unless their configurations, datasets, thresholds, and metric definitions are documented.

---
## Implementation-Level Evaluation Considerations

### YOLO `max_det` vs COCO `maxDets`

The evaluation documentation must distinguish two similar but different controls:

| Parameter | Stage | Meaning |
|---|---|---|
| `max_det` | YOLO validation / inference | Maximum detections produced or retained per image by the YOLO runtime |
| `maxDets` | COCO evaluation | Detection limits used by `pycocotools` when computing AP/AR |

Both can affect results. Reporting only one of them is not sufficient for reproducible comparison.

### Batch Size and Metric Stability

Batch size should be recorded with every validation and evaluation result. Very small batch sizes, especially batch size 1, may create unstable training dynamics or unexpectedly optimistic validation outcomes in some configurations.

Recommended fields:

```text
batch_size
effective_batch_size
multi_gpu_mode
model_family
img_size
```

Unexpectedly high metrics should be reviewed with qualitative examples and class-level analysis.


### Model Selection Score Is a Project-Specific Heuristic

The best-model selection utility may use a weighted score such as:

```text
score = 0.7 * mAP50-95 + 0.3 * F1
```

This is a practical heuristic, not a universal definition of the best model. If operational requirements prioritize recall, precision, or specific classes, the weighting should be configurable and documented per experiment.

### Post-Training Metric Recovery

When training does not produce complete metric objects, evaluation may be executed immediately after training on the generated `best.pt` checkpoint. Evaluation reports should state whether metrics came from:

```text
training return object
results.csv
YOLO-native post-training validation
COCO conversion and pycocotools evaluation
```

### Resolution-Mismatch Reporting

Evaluation summaries should record:

```text
train_img_size
validation_img_size
inference_img_size
sahi_slice_size
resolution_mismatch
```

This is necessary because high-resolution agricultural imagery can expose scale effects that are not visible in standard resized validation splits.

---

### Video Tracking Outputs Are Not COCO Evaluation Outputs

Video tracking artifacts such as annotated MP4 files, unique object counts, and `.srt` frame summaries are operational inference outputs. They should not be confused with COCO evaluation artifacts unless a separate frame-level ground truth and evaluation protocol is defined.

Important distinction:

```text
COCO evaluation
    image-level ground truth and prediction comparison

video tracking summary
    temporal inference artifact based on tracker IDs
```

If video metrics are required, the project should define a separate evaluation protocol for tracking quality, ID switches, frame-level precision/recall, or object counting error.

### Video Counting and Tracking Evaluation Caveat

Video tracking summaries should not be interpreted as detection evaluation metrics unless a separate video ground truth and tracking evaluation protocol exists.

Important distinction:

```text
unique object count
    derived from YOLO tracking IDs across frames

detection metric
    derived from comparison against labeled ground truth
```

If the project evaluates video tracking quality, recommended metrics may include:

```text
ID switches
missing track IDs
track fragmentation
counting error
frame-level precision / recall
SRT synchronization error
```

The current video JSON and SRT outputs are operational inference artifacts, not COCO evaluation artifacts.

## Recommended Future Improvements

- Add experiment tracking with `run_id`.
- Store model and dataset versions.
- Add confusion matrix generation.
- Add qualitative false positive / false negative galleries.
- Add IoU distribution plots.
- Add runtime benchmarking.
- Add validation benchmarking summaries from `model.val()`.
- Persist class metrics with explicit `class_id` and `class_name`.
- Add confidence-threshold sensitivity analysis.
- Add automated metric regression tests.

---

![AgriDrone Evaluation Workflow](../assets/images/agridrone-vision-evaluation-pipeline-3-evaluation.png)
