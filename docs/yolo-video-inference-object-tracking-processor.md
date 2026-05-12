# 🎥 YOLO Video Inference & Object Tracking Processor

> **Purpose:** Process videos with YOLO object tracking, preserve temporal object identity, generate annotated video outputs, compute unique object counts by class, and export frame-level or run-level metadata for downstream review.

---

## 🌟 Overview

The **YOLO Video Inference & Object Tracking Processor** is a temporal computer vision component within the AgriDrone Vision Evaluation Pipeline.

Unlike static image inference, video processing introduces temporal state. The system does not only detect objects frame by frame; it can also track object identities across frames and compute unique object counts using YOLO tracking IDs.

Conceptual flow:

```text
input video
    │
    ▼
OpenCV frame decoding
    │
    ▼
YOLO model.track()
    │
    ▼
tracking ID extraction
    │
    ▼
ObjectCounter
    │
    ▼
manual OpenCV rendering
    │
    ├── annotated MP4
    ├── JSON counts
    └── optional SRT frame summaries
```

This component is part of the inference runtime layer, but it has different correctness criteria from image or SAHI inference.

---

## 🛠️ Component Classification

This component can be classified as:

- **Data processor**
- **Computer vision inference service**
- **Temporal video analytics component**
- **Synchronous frame-by-frame processor**
- **Advanced prototype / pre-production video processing module**

It is not a background worker, queue, or distributed video service. It currently operates as a synchronous local batch processor.

---

## 🎯 Main Responsibility

The processor is responsible for:

- opening input videos with OpenCV
- reading frames sequentially
- running YOLO tracking with persistent IDs
- extracting bounding boxes, confidence scores, class IDs, and tracking IDs
- counting unique tracked objects by class
- rendering custom bounding boxes and visual overlays
- preserving configured class colors
- avoiding color degradation from RGB/BGR conversion issues
- writing annotated video outputs
- exporting final count summaries as JSON
- optionally exporting `.srt` frame-level detection summaries
- logging progress and frame-level errors

---

## 📥 Inputs

Typical inputs:

```text
input_video_path
trained YOLO model best.pt
confidence_threshold
class dictionary
class color map
output directory
device GPU/CPU
```

Optional inputs:

```text
tracking configuration
SRT output flag
rendering style configuration
frame failure threshold
video codec / FPS output settings
```

---

## 📤 Outputs

Generated outputs may include:

```text
annotated video file
final unique-count JSON
frame-level metadata JSON
optional .srt subtitle file
processing logs
failed-frame summary
```

Example output layout:

```text
outputs/
├── videos/
│   ├── input_video_annotated.mp4
│   └── input_video_detections.srt
├── metadata/
│   └── input_video_tracking_summary.json
└── logs/
    └── input_video_processing_log.json
```

---

## 🔄 High-Level Workflow

```text
Load YOLO Model
        │
        ▼
Open VideoCapture
        │
        ▼
Read Frame
        │
        ▼
Run model.track()
        │
        ▼
Extract box, class, confidence, track_id
        │
        ▼
Update ObjectCounter
        │
        ▼
Render Bounding Boxes and Counters
        │
        ▼
Write Annotated Frame
        │
        ▼
Persist JSON / SRT Summary
```

---

## 🔍 Detailed Workflow

### Step 1: Load Model

The processor loads a trained YOLO checkpoint.

Typical artifact:

```text
best.pt
```

The model should match the configured class dictionary and class color mapping.

---

### Step 2: Open Video with OpenCV

The input video is opened using OpenCV.

Representative components:

```text
cv2.VideoCapture
cv2.VideoWriter
```

The processor should validate:

- input file exists
- video can be opened
- FPS is readable
- frame width and height are valid
- output codec is available
- output directory is writable

---

### Step 3: Frame-by-Frame Processing

The processor reads frames sequentially.

Current execution model:

```text
for each frame:
    read frame
    run tracking
    render annotations
    write output frame
```

This is synchronous processing. It is not a persistent asynchronous service.

---

### Step 4: YOLO Tracking

The processor uses Ultralytics tracking, typically through:

```text
model.track()
```

Required output contract:

```text
box.xyxy
box.conf
box.cls
box.id
```

The `box.id` field is essential for unique object counting.

If `box.id` is missing, the system should either:

- skip that detection from unique counting
- count it only as a frame-level detection
- record the missing ID in the output summary

---

### Step 5: ObjectCounter Logic

The ObjectCounter tracks unique object IDs across frames.

Conceptual rule:

```text
unique object count = number of unique tracking IDs observed
```

The processor may maintain structures such as:

```text
seen_track_ids_by_class
total_unique_ids
class_counts
```

Important distinction:

```text
frame-level detections != unique tracked objects
```

A video may contain thousands of frame-level detections but far fewer unique objects.

---

### Step 6: Custom OpenCV Rendering

The processor renders annotations manually with OpenCV rather than relying only on native YOLO plotting.

Reasons:

- preserve original video colors
- avoid RGB/BGR degradation
- use configured class colors
- render custom confidence labels
- render total and per-class counters
- apply semi-transparent overlays and readable text backgrounds

Typical overlays:

```text
bounding boxes
class labels
confidence labels
tracking IDs
total object count
per-class count summary
```

---

### Step 7: Color-Space Contract

The pipeline mixes OpenCV, PIL/Pillow, and Ultralytics utilities. Color-space handling must be explicit.

Recommended contract:

```text
OpenCV frame input     → BGR
PIL image operations   → RGB
VideoWriter output     → BGR
YOLO plotting outputs  → validate before writing
```

Risk:

```text
Incorrect RGB/BGR conversion can degrade colors or produce visually misleading outputs.
```

---

### Step 8: SRT Frame-Level Output

When enabled, the processor can generate `.srt` outputs containing frame-aligned detection summaries.

Recommended SRT contract:

```text
frame_index
start_time
end_time
total_detections
unique_tracked_objects
class_counts
```

SRT outputs should be synchronized with:

- video FPS
- frame index
- output video duration
- processing start and end timestamps

The `.srt` file is a temporal metadata artifact, not equivalent to static image JSON.

---

### Step 9: Final JSON Summary

A final JSON summary should include:

```json
{
  "video_name": "input_video.mp4",
  "model_path": "weights/best.pt",
  "frames_total": 3600,
  "frames_processed": 3592,
  "frames_failed": 8,
  "total_unique_objects": 125,
  "class_counts": {
    "weed": 82,
    "crop": 43
  },
  "frames_with_missing_track_ids": 17,
  "output_video": "input_video_annotated.mp4",
  "srt_output": "input_video_detections.srt"
}
```

---

## 🧠 Tracking Correctness

The most important correctness constraint is:

```text
Object counts are only as reliable as tracker ID persistence.
```

Potential issues:

- `box.id` missing for some frames
- ID switches across frames
- same object receives multiple IDs
- different objects share an ID
- temporary occlusion causes identity reset
- tracker behavior changes across Ultralytics versions

Recommended metrics:

```text
frames_with_missing_track_ids
unique_track_ids_total
track_id_switch_suspected_count
detections_without_id
average_track_duration
```

---

## 📊 Performance Measurement Strategy

Video throughput is affected by several stages, not only model inference.

Recommended timing fields:

```text
mean_decode_ms
mean_track_ms
mean_render_ms
mean_write_ms
mean_srt_write_ms
effective_processing_fps
input_video_fps
output_video_fps
frames_total
frames_processed
frames_failed
```

Why this matters:

- decoding may be CPU-bound
- tracking may be GPU-bound
- rendering may be CPU-bound
- encoding may be I/O/codec-bound
- SRT/JSON serialization may become non-trivial for long videos

---

## 🚨 Error Handling and Failure Policy

The processor should isolate errors at frame level where possible.

Recommended behavior:

```text
single frame failure
    → log error and continue

repeated frame failures above threshold
    → stop video processing and mark job as partial_failed
```

Recommended summary fields:

```text
frames_processed
frames_failed
failure_threshold
stop_reason
partial_output_status
```

This distinction is important because a few corrupted frames may be tolerable, while many failures may indicate a broken video or decoding problem.

---

## 🧾 Artifact Consistency Policy

Video processing can leave inconsistent outputs if the job fails mid-run.

Example inconsistent states:

```text
MP4 written but JSON incomplete
JSON written but MP4 corrupted
SRT missing while MP4 exists
```

Recommended mitigation:

- write outputs to temporary paths first
- promote artifacts only after successful completion
- write a job status field
- generate a final manifest

Recommended status values:

```text
processing
complete
partial_failed
failed
validated
```

---

## 🧰 Dependencies

Core dependencies:

- Python
- OpenCV
- Ultralytics YOLO
- PyTorch / CUDA
- NumPy
- JSON
- `collections.defaultdict`
- local filesystem

Optional / related dependencies:

- Pillow
- pillow-heif
- ExifTool
- QGIS consumer layer for geospatial outputs generated from image workflows

---

## ⚠️ Technical Risks

### Tracker ID Instability

Unique counts depend on stable `box.id` values.

Mitigation:

- record frames with missing IDs
- separate frame-level detections from unique-object counts
- validate counts visually on sampled clips

### Ultralytics Result Structure Coupling

The processor depends on `result.boxes` structure.

Mitigation:

- validate required fields at runtime
- pin Ultralytics versions
- add regression tests for result parsing

### Frame-by-Frame Bottleneck

Video processing is synchronous and may be slow for long videos.

Mitigation:

- track per-stage timing
- separate decode, inference, rendering, and encoding metrics
- consider worker-based processing for production

### Rendering Coupling

Tracking, rendering, counting, and output writing are tightly coupled.

Mitigation:

- separate tracker, counter, renderer, and serializer modules
- persist raw tracking metadata before rendering when possible

### Color-Space Bugs

Mixing OpenCV, PIL, and YOLO plotting can produce visual artifacts.

Mitigation:

- define a strict BGR/RGB contract
- add sample-frame visual tests
- avoid unnecessary conversions

### Partial Output Risk

Long-running video jobs can fail after generating incomplete artifacts.

Mitigation:

- use temporary outputs
- write manifests
- promote completed artifacts atomically

---

## 🧪 Recommended Validations

Before processing:

- model path exists
- video file exists
- OpenCV can open the video
- output codec is available
- class dictionary matches model classes
- colors exist for all classes
- output directory is writable

During processing:

- `box.id` exists when unique counting is enabled
- boxes are within frame bounds
- confidence values are numeric
- frame dimensions remain stable
- failure threshold is not exceeded

After processing:

- output video exists
- output video can be opened
- JSON summary is valid
- SRT file is synchronized when generated
- frame counts match expected processed frames
- unique counts are separated from frame-level detections

---

## 📌 Portfolio Summary

This component demonstrates temporal computer vision engineering beyond static image inference.

It uses YOLO tracking to process videos frame by frame, extract persistent object IDs, count unique objects by class, render custom OpenCV overlays, preserve visual fidelity, and export annotated videos, JSON summaries, and optional SRT frame-level artifacts.

The module highlights applied experience in:

- video inference
- object tracking
- OpenCV rendering
- temporal metadata generation
- unique object counting
- pipeline error handling
- batch video processing
- ML output serialization

---

## 🔒 Privacy and Confidentiality Notice

This documentation describes generalized video inference and tracking architecture. It does not expose private videos, client data, field locations, proprietary model weights, or sensitive operational details.
