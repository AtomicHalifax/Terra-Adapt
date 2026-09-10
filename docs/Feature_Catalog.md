# Feature Catalog

## Overview

TerraAdapt uses RGB satellite imagery as the model input.

The project also computes several spectral indices during target-data analysis and quality control. These indices help characterize the imagery but are **not used as additional model inputs**.

## Model Input

| Feature    | Source     | Description         | Model Input |
| ---------- | ---------- | ------------------- | ----------- |
| Red (B4)   | Sentinel-2 | Red spectral band   | Yes         |
| Green (B3) | Sentinel-2 | Green spectral band | Yes         |
| Blue (B2)  | Sentinel-2 | Blue spectral band  | Yes         |

The three bands form the RGB input:

```text
B4 (Red) + B3 (Green) + B2 (Blue)
                ↓
          224 × 224 RGB
                ↓
            ResNet50
```

## Spectral Analysis Features

### NDVI

```text
NDVI = (B8 - B4) / (B8 + B4)
```

NDVI uses the near-infrared and red bands to characterize vegetation strength.

**Role in TerraAdapt:** analysis and quality control only.

### GNDVI

```text
GNDVI = (B8 - B3) / (B8 + B3)
```

GNDVI uses the near-infrared and green bands to provide additional information about vegetation characteristics.

**Role in TerraAdapt:** analysis and quality control only.

### NGRDI

```text
NGRDI = (B3 - B4) / (B3 + B4)
```

NGRDI measures the normalized difference between green and red reflectance.

**Role in TerraAdapt:** analysis and quality control only.

## Why the Indices Are Not Model Inputs

The adaptation experiment was designed to evaluate how a pretrained RGB ResNet50 model transfers to a modern target domain.

Therefore, NDVI, GNDVI, and NGRDI were intentionally kept outside the model input pipeline.

This provides a cleaner experiment:

```text
Sentinel-2 RGB
      ↓
  ResNet50
      ↓
Target-domain prediction
```

rather than introducing additional engineered spectral features into the neural network.

## Target Classes

The target classifier predicts four consolidated land-cover classes:

| ID | Class |
| -: | ----- |
|  0 | Water |
|  1 | Trees |
|  2 | Crops |
|  3 | Built |

These classes are derived from the target-domain labeling process described in `Data_Collection.md`.

## Feature Summary

| Category         | Features                      | Purpose                            |
| ---------------- | ----------------------------- | ---------------------------------- |
| RGB              | B4, B3, B2                    | ResNet50 model input               |
| Vegetation index | NDVI                          | Analysis / QC                      |
| Vegetation index | GNDVI                         | Analysis / QC                      |
| Surface index    | NGRDI                         | Analysis / QC                      |
| Location         | Latitude / Longitude          | Spatial organization and splitting |
| Target label     | Water / Trees / Crops / Built | Supervised adaptation target       |

## Important Distinction

The latitude, longitude, and spectral indices are **not neural-network input features**.

They are used for dataset construction, spatial organization, analysis, or quality control.

The model itself receives RGB image patches and predicts one of the four target classes.
