# Data Collection

## Overview

TerraAdapt uses a modern target-domain dataset constructed from Sentinel-2 satellite imagery and Dynamic World land-cover predictions.

The goal was to create a target dataset that represents contemporary satellite imagery while keeping the experiment focused on domain adaptation from the existing EuroSAT ResNet50 model.

## Data Sources

### Sentinel-2

The target imagery comes from the **Sentinel-2 Surface Reflectance Harmonized** collection:

```text
COPERNICUS/S2_SR_HARMONIZED
```

The imagery covers the period:

```text
2024-01-01 → 2025-01-01
```

RGB imagery was extracted and converted into:

```text
224 × 224 × 3
```

RGB patches suitable for the ResNet50 input pipeline.

## Dynamic World

Land-cover information was obtained from Google's Dynamic World dataset:

```text
GOOGLE/DYNAMICWORLD/V1
```

Dynamic World provides near-real-time land-cover predictions associated with Sentinel-2 imagery.

For TerraAdapt, these predictions were used to construct **target-domain pseudo-labels**.

> **Important:** Dynamic World labels are treated as pseudo-labels, not manually verified ground truth.

This distinction is important when interpreting the final evaluation results.

## Target Classes

The original Dynamic World land-cover categories were consolidated into four broader classes:

| Target ID | TerraAdapt Class | Conceptual Mapping                 |
| --------: | ---------------- | ---------------------------------- |
|         0 | Water            | River + SeaLake                    |
|         1 | Trees            | Forest                             |
|         2 | Crops            | AnnualCrop + PermanentCrop         |
|         3 | Built            | Residential + Industrial + Highway |

The following EuroSAT categories were excluded from the target mapping:

```text
Pasture
HerbaceousVegetation
```

This produces a four-class target problem that is appropriate for the adaptation experiment.

## Candidate Selection

Candidate locations were identified using the Sentinel-2 and Dynamic World data.

The selection process considered the available land-cover predictions and confidence information before constructing the final RGB patch dataset.

The resulting candidate metadata was stored during the experiment in:

```text
terraadapt_candidates_4057.csv
```

The final target dataset contains:

```text
4,057 RGB patches
```

Each patch has a spatial location associated with its source imagery.

## Spatial Structure

Because satellite imagery is spatially correlated, randomly splitting individual image patches could allow visually similar neighboring regions to appear in both training and evaluation sets.

TerraAdapt therefore uses a **spatial split**, keeping spatial groups together when assigning samples to the train, validation, and test sets.

Final split:

| Split      |   Samples | Approx. Proportion |
| ---------- | --------: | -----------------: |
| Train      |     2,839 |                70% |
| Validation |       609 |                15% |
| Test       |       609 |                15% |
| **Total**  | **4,057** |           **100%** |

The test set remains untouched during model development and is used only for the final evaluation.

## Dataset Structure

The final RGB dataset is organized into four target classes:

```text
TerraAdapt_RGB_Final/
├── water/
├── trees/
├── crops/
└── built/
```

Each class contains 224×224 RGB satellite patches.

## Spectral Quality Analysis

Three spectral indices were calculated during dataset analysis and quality-control work:

### NDVI

Normalized Difference Vegetation Index:

```text
NDVI = (B8 - B4) / (B8 + B4)
```

NDVI provides an indication of vegetation strength and helps distinguish vegetation-rich areas from non-vegetated surfaces.

### GNDVI

Green Normalized Difference Vegetation Index:

```text
GNDVI = (B8 - B3) / (B8 + B3)
```

GNDVI uses the green band instead of the red band and provides additional information about vegetation characteristics.

### NGRDI

Normalized Green-Red Difference Index:

```text
NGRDI = (B3 - B4) / (B3 + B4)
```

NGRDI captures differences between green and red reflectance and can provide additional information about vegetation and surface characteristics.

### Role of the Indices

These indices were used for **analysis and quality control only**.

They were **not provided as additional inputs to the ResNet50 model**.

This keeps the adaptation experiment focused on RGB imagery while still allowing spectral information to be used to inspect the target dataset.

## Data Policy

The complete target dataset is **not included in this repository**.

The repository contains the code, documentation, notebook, and selected figures required to understand the experiment without distributing the full satellite dataset.

Similarly, the final test images are not uploaded to GitHub.

## Reproducibility

The notebook documents the target-data construction and preprocessing pipeline used to create the dataset.

The intended workflow is:

```text
Sentinel-2 imagery
        ↓
Dynamic World predictions
        ↓
Candidate selection
        ↓
Target class mapping
        ↓
RGB patch extraction
        ↓
Quality analysis
        ↓
Spatial grouping
        ↓
Train / Validation / Test
```

The resulting target dataset is then used for the ResNet50 adaptation experiment described in the model-development documentation.
