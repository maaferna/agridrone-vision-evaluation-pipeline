# Precision Agriculture Object Detection Pipeline

> **Research-grade computer vision pipeline for object detection, geospatial processing, and COCO-based model evaluation on high-resolution agricultural drone imagery.**

---

## 🌟 Overview

**AgriDrone Vision Evaluation Pipeline** is a computer vision and machine learning evaluation system designed to process high-resolution drone imagery in agricultural environments. The project integrates YOLO-based object detection, SAHI slicing inference, geospatial metadata extraction, standardized COCO evaluation metrics, and automated reporting.

The pipeline was designed to support reproducible experimentation with object detection models applied to aerial agricultural imagery, where objects of interest may be small, partially occluded, visually ambiguous, or distributed across large 4K images.

The system enables direct comparison between standard YOLO inference and SAHI-based sliced inference, helping evaluate how different inference strategies affect detection quality, recall, precision, and model robustness in real-world drone image conditions.

---


## 🛠️ Technology Stack

The project leverages the following technologies and tools:

| **Category**            | **Technologies**                                                                 |
|-------------------------|---------------------------------------------------------------------------------|
| **Programming Language**| ![Python](https://img.shields.io/badge/Python-3.9%2B-blue)                     |
| **Machine Learning**    | ![PyTorch](https://img.shields.io/badge/PyTorch-Deep%20Learning-red)           |
| **Computer Vision**     | ![YOLO](https://img.shields.io/badge/YOLO-Object%20Detection-yellow)           |
|                         | ![SAHI](https://img.shields.io/badge/SAHI-Slicing%20Inference-green)           |
|                         | ![OpenCV](https://img.shields.io/badge/OpenCV-Image%20Processing-blueviolet)   |
| **Evaluation**          | ![pycocotools](https://img.shields.io/badge/pycocotools-COCO%20Metrics-orange) |
| **Data Processing**     | ![NumPy](https://img.shields.io/badge/NumPy-Array%20Computations-lightgrey)    |
|                         | ![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-yellowgreen)     |
| **Visualization**       | ![Matplotlib](https://img.shields.io/badge/Matplotlib-Data%20Visualization-blue)|
| **Geospatial Processing**| ![Rasterio](https://img.shields.io/badge/Rasterio-Geospatial%20Data-brightgreen)|
|                         | ![GeoJSON](https://img.shields.io/badge/GeoJSON-Geospatial%20Format-lightblue) |
|                         | ![Shapefile](https://img.shields.io/badge/Shapefile-GIS%20Data-red)            |
| **Infrastructure**      | ![CUDA](https://img.shields.io/badge/CUDA-GPU%20Acceleration-green)            |

---



## 🚀 Problem Statement

Agricultural image analysis using drones introduces several technical challenges:

- **High-resolution images** are computationally expensive to process.
- **Small objects** are difficult to detect when images are resized before inference.
- **Object density, occlusion, lighting variation, and image noise** can degrade model performance.
- **Standard evaluation workflows** are often inconsistent or difficult to reproduce.
- **Geospatial context** is frequently separated from computer vision results.
- **Manual inspection** does not scale for large datasets.

This project addresses these challenges by building a reproducible evaluation pipeline that connects object detection, geospatial processing, structured outputs, and scientific metrics into a single workflow.

---

## 🎯 Main Objectives

- Execute YOLO inference on high-resolution drone images.
- Support SAHI slicing inference for improved detection of small objects.
- Export normalized YOLO predictions.
- Convert YOLO ground truth and predictions into COCO format.
- Evaluate model performance using `pycocotools`.
- Generate global and per-class metrics.
- Produce CSV, JSON, and visualization outputs for reporting.
- Extract GPS/EXIF metadata from drone images.
- Generate geospatial outputs such as GeoJSON, CSV, and shapefiles.
- Provide a reproducible workflow for model benchmarking and applied research.

---

## 🏗️ System Architecture

### Diagram 1: High-Level Architecture

![High-Level Architecture](assets/images/agridrone-vision-evaluation-pipeline-1.png)

### Diagram 2: Detailed Architecture

![Detailed Architecture](assets/images/agridrone-vision-evaluation-pipeline-2-arquitecture.png)

---
## 📈 System Flow

### Step 1: Execution Trigger

The user runs the main script and selects the desired workflow, such as inference, evaluation, or comparative analysis.

### Step 2: Data Loading

The system loads:

- Drone images
- YOLO model weights
- Ground truth annotations
- Class dictionary
- Inference parameters
- Input and output directories

### Step 3: Inference

The system executes one of two inference strategies:

1. Direct YOLO inference
2. SAHI sliced inference

For SAHI inference, image slices are processed independently and then reconstructed into full-image coordinates.

### Step 4: Prediction Export

Detections are normalized and exported as YOLO-format `.txt` files.

### Step 5: Geospatial Metadata Extraction

The system extracts GPS and EXIF metadata from drone images, converts coordinates, and generates geospatial output files.

### Step 6: COCO Conversion

Ground truth annotations and model predictions are converted into COCO JSON format.

### Step 7: Evaluation

The system evaluates predictions using COCO metrics with `pycocotools`.

### Step 8: Reporting

The system generates JSON reports, CSV files, and plots for global and per-class model performance.

### Step 9: Final Outputs

The final result is a reproducible evaluation package containing predictions, geospatial files, COCO artifacts, metrics, and visual reports.

---

## 📚 Privacy & Confidentiality Notice

This repository is intended to document the architecture, methodology, and technical approach of a computer vision system for agricultural drone imagery.

It does not include:

- Confidential client data
- Proprietary datasets
- Private business information
- Production credentials
- Internal endpoints
- Sensitive geospatial locations
- Private model weights, unless explicitly authorized

Any sample images, annotations, or outputs included in this repository should be anonymized, synthetic, or publicly shareable.
