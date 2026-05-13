# 🔐 Public Release Sanitization Audit

## Purpose

This document records the publication-safety assumptions applied to the public Markdown version of the AgriDrone Vision Evaluation Pipeline documentation.

## Scope

The public release includes Markdown documentation only.

It does not include:

- source code
- private datasets
- original drone imagery
- styled detection images from private data
- trained model weights
- ClearML exports
- CVAT or Roboflow private project exports
- real GPS / UTM coordinates
- GeoJSON / Shapefile / JGW / GeoTIFF outputs from real field data
- production credentials
- internal reports
- unpublished experimental metrics

## Sanitization Actions Applied

The public documentation was sanitized by:

- replacing coordinate examples with anonymized placeholders
- replacing metric values in example JSON with illustrative placeholders
- adding public-safe confidentiality notices to major documents
- adding geospatial confidentiality warnings
- avoiding institution, client, field, farm, station, and researcher names
- avoiding absolute local filesystem paths
- framing the documents as generalized engineering documentation rather than literal institutional project documentation

## Remaining Publication Guidance

Before publishing, manually verify that no files include:

```text
INSTITUTION_NAME
CLIENT_NAME
FIELD_NAME
FARM_NAME
REAL_DATASET_NAME
REAL_COORDINATES
REAL_METRICS
CLEARML_WORKSPACE
CVAT_PROJECT
ROBOFLOW_WORKSPACE
/home/
/media/
```

## Recommended Repository Policy

The public repository should be positioned as:

```text
A generalized and anonymized technical portfolio version of an agricultural computer vision, geospatial processing, and video tracking pipeline.
```

It should not be positioned as:

```text
A literal release of a client, institutional, or production project.
```
