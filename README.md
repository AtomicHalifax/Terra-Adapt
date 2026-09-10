<div align="center">

# TERRA ADAPT

### DOMAIN ADAPTATION FOR SATELLITE IMAGE CLASSIFICATION

**EuroSAT ResNet50 → Target-Domain Adaptation → Sentinel-2 + Dynamic World**

<br>

`EUROSAT`  →  `RESNET50`  →  `ADAPT`  →  `SENTINEL-2`

<br>

**WATER**  ·  **TREES**  ·  **CROPS**  ·  **BUILT**

<br>

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square\&logo=python\&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-ResNet50-EE4C2C?style=flat-square\&logo=pytorch\&logoColor=white)
![Sentinel-2](https://img.shields.io/badge/Sentinel--2-Earth%20Observation-2E7D32?style=flat-square)
![Dynamic World](https://img.shields.io/badge/Dynamic%20World-Pseudo--Labels-607D8B?style=flat-square)

</div>

---

## Overview

**TerraAdapt** is a domain-adaptation experiment for satellite image classification.

The project starts from an existing **ResNet50 model trained on EuroSAT** and adapts it to a new target domain constructed from **Sentinel-2 imagery** with **Dynamic World labels**.

The goal is not simply to train another classifier, but to study how a model trained on one remote-sensing dataset behaves when transferred to a different geographic, temporal, and data-generation domain.

The target domain contains four broad land-cover classes:

* Water
* Trees
* Crops
* Built

Dynamic World labels are treated as **pseudo-labels rather than ground truth**, which is an important limitation when interpreting the final results.

---

## Dataset

The target dataset was constructed using:

| Component         | Source                        |
| ----------------- | ----------------------------- |
| Satellite imagery | Sentinel-2                    |
| Image collection  | `COPERNICUS/S2_SR_HARMONIZED` |
| Target labels     | Dynamic World                 |
| Label collection  | `GOOGLE/DYNAMICWORLD/V1`      |
| Time range        | 2024-01-01 → 2025-01-01       |
| Patch size        | 224 × 224                     |
| Image format      | RGB                           |
| Number of patches | 4,057                         |
| Number of classes | 4                             |

### Target Classes

| ID | Class |
| -: | ----- |
|  0 | Water |
|  1 | Trees |
|  2 | Crops |
|  3 | Built |

The target classes were constructed by grouping Dynamic World / EuroSAT-compatible land-cover categories:

```text
Water
├── River
└── SeaLake

Trees
└── Forest

Crops
├── AnnualCrop
└── PermanentCrop

Built
├── Residential
├── Industrial
└── Highway
```

`Pasture` and `HerbaceousVegetation` were excluded from the four-class target setup.

---

## Spatial Split

A spatial split was used to reduce geographic leakage between training and evaluation data.

| Split      |   Samples |
| ---------- | --------: |
| Train      |     2,839 |
| Validation |       609 |
| Test       |       609 |
| **Total**  | **4,057** |

Spatial groups were kept together during the split rather than randomly distributing individual patches.

The final test set remained untouched until the final evaluation.

---

## Spectral Analysis

Three spectral indices were calculated for analysis and quality checking:

### NDVI

$$
NDVI = \frac{B8-B4}{B8+B4}
$$

NDVI provides a measure related to vegetation presence and vigor.

### GNDVI

$$
GNDVI = \frac{B8-B3}{B8+B3}
$$

GNDVI uses the green band instead of red and provides another vegetation-sensitive measurement.

### NGRDI

$$
NGRDI = \frac{B3-B4}{B3+B4}
$$

NGRDI captures differences between green and red reflectance and can provide additional information about vegetation and surface characteristics.

These indices were used for **analysis and quality control only**.

They were **not provided as inputs to the ResNet50 model**.

---

## Source Model

TerraAdapt starts from an existing **EuroSAT ResNet50** model.

The source model was fine-tuned on the original EuroSAT task before being used as the pretrained starting point for target-domain adaptation.

### Source Performance

| Metric        |      Score |
| ------------- | ---------: |
| Test Accuracy | **96.56%** |
| F1 Score      | **0.9656** |
| Macro ROC-AUC | **0.9977** |

### Architecture

```text
ResNet50
   ↓
2048-dimensional feature representation
   ↓
10-class classifier
```

Total parameters:

```text
23,516,228
```

The source checkpoint contains a 10-class classification head:

```text
fc.weight → [10, 2048]
fc.bias   → [10]
```

---

## Adaptation

The original 10-class classifier was replaced with a four-class target-domain classifier:

```text
Linear(2048 → 4)
```

The target architecture therefore becomes:

```text
                 Pretrained EuroSAT
                       ResNet50
                          │
                          ▼
                 Feature Representation
                       2048-d
                          │
                 Replace Source Head
                          │
                          ▼
                    Linear(2048, 4)
                          │
                          ▼
              Water / Trees / Crops / Built
```

### Initial Adaptation

The initial adaptation fine-tuned:

```text
layer4 + classification head
```

Parameter distribution:

| Parameter Group | Parameters |
| --------------- | ---------: |
| Trainable       | 14,972,932 |
| Frozen          |  8,543,296 |
| Total           | 23,516,228 |

### Deeper Adaptation Experiment

A deeper configuration additionally fine-tuned `layer3`:

```text
layer3 + layer4 + classification head
```

| Configuration          | Trainable Parameters |
| ---------------------- | -------------------: |
| Layer 4 + FC           |           14,972,932 |
| Layer 3 + Layer 4 + FC |           22,071,300 |

The deeper configuration left:

```text
1,444,928
```

parameters frozen.

---

## Results

The model performed substantially better on the validation split than on the final untouched test set.

### Validation

| Metric           |      Score |
| ---------------- | ---------: |
| Accuracy         | **71.76%** |
| Macro Precision  | **0.7444** |
| Macro Recall     | **0.7191** |
| Macro F1         | **0.7179** |
| Macro ROC-AUC    | **0.8682** |
| Weighted ROC-AUC | **0.8687** |

### Final Test

| Metric           |      Score |
| ---------------- | ---------: |
| Accuracy         | **54.02%** |
| Macro Precision  | **0.5402** |
| Macro Recall     | **0.5431** |
| Macro F1         | **0.5361** |
| Macro ROC-AUC    | **0.8259** |
| Weighted ROC-AUC | **0.8252** |

### Final Test Confusion Matrix

```text
              Predicted
             W    T    C    B
Actual W    93   13   26   18
       T    34   68   34   14
       C     5   36   56   62
       B     2   16   20  112
```

The final test results show that **Built** was the strongest class, while **Crops** was considerably more difficult.

A notable source of confusion was:

```text
Crops → Built
```

---

## Source vs Target

The difference between the original EuroSAT performance and TerraAdapt's final target-domain performance highlights the difficulty of transferring a model between remote-sensing domains.

|               | EuroSAT Source | TerraAdapt Target |
| ------------- | -------------: | ----------------: |
| Classes       |             10 |                 4 |
| Model         |       ResNet50 |          ResNet50 |
| Accuracy      |     **96.56%** |        **54.02%** |
| Macro F1      |     **0.9656** |        **0.5361** |
| Macro ROC-AUC |     **0.9977** |        **0.8259** |

The results should not be interpreted as a failure of ResNet50 itself.

Instead, they demonstrate that strong performance on a source dataset does not necessarily transfer directly to a new geographic, temporal, labeling, and imaging domain.

---

## Limitations

Several limitations are important when interpreting TerraAdapt.

### Dynamic World pseudo-labels

Dynamic World labels are automatically generated predictions rather than manually verified ground truth.

Therefore, target-domain evaluation contains potential label noise.

### Domain shift

The source and target datasets differ in:

* Geographic distribution
* Acquisition period
* Dataset construction
* Label-generation process
* Image characteristics

This creates a genuine domain-shift problem.

### Target classes

The four target classes are broad groupings constructed from multiple land-cover categories.

This simplifies the original classification problem but can also introduce semantic overlap.

### Final test performance

The final untouched test accuracy of **54.02%** is relatively weak.

The result is reported directly rather than selecting or tuning against the test set.

---

## Reproducibility

The complete experimental workflow is documented in the notebook:

```text
notebooks/TerraAdapt.ipynb
```

The workflow covers:

```text
Sentinel-2 + Dynamic World
          ↓
Target Dataset Construction
          ↓
Candidate Selection
          ↓
Quality / Spectral Analysis
          ↓
Spatial Train / Validation / Test Split
          ↓
Load EuroSAT ResNet50
          ↓
Replace 10-Class Head
          ↓
Target-Domain Adaptation
          ↓
Validation
          ↓
Model Selection
          ↓
Untouched Test Evaluation
```

The dataset itself is **not included in this repository**.

Large model checkpoints are also intentionally excluded from Git tracking.

---

## Project Structure

```text
TerraAdapt/
│
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
│
├── docs/
│   ├── why.md
│   ├── Data_Collection.md
│   ├── Feature_Catalog.md
│   ├── Model_Development.md
│   ├── Model_Evaluation.md
│   └── Workflow.md
│
├── notebooks/
│   └── TerraAdapt.ipynb
│
├── models/
│   └── README.md
│
└── figures/
    ├── dataset_samples.png
    ├── spatial_split.png
    ├── validation_confusion_matrix.png
    ├── validation_metrics.png
    └── validation_roc_auc.png
```

---

## Documentation

Detailed project documentation:

* [`why.md`](docs/why.md) — motivation and project reasoning
* [`Data_Collection.md`](docs/Data_Collection.md) — target dataset construction
* [`Feature_Catalog.md`](docs/Feature_Catalog.md) — model inputs and spectral analysis
* [`Model_Development.md`](docs/Model_Development.md) — architecture and adaptation strategy
* [`Model_Evaluation.md`](docs/Model_Evaluation.md) — validation and final test evaluation
* [`Workflow.md`](docs/Workflow.md) — complete experimental workflow

---

## Requirements

Main dependencies:

```text
torch
torchvision
numpy
pandas
scikit-learn
matplotlib
Pillow
jupyter
```

Install with:

```bash
pip install -r requirements.txt
```

---

## Model Checkpoint

The best TerraAdapt model was saved as:

```text
terraadapt_resnet50_best.pth
```

Large checkpoint files are intentionally not committed to the repository.

The notebook documents the model-loading and adaptation procedure.

---

## Key Takeaway

TerraAdapt explores a simple but important question:

> **How well does a strong satellite image classifier transfer to a different remote-sensing domain?**

The experiment shows that high source-domain performance does not guarantee high target-domain performance.

The gap between the **96.56% EuroSAT source accuracy** and the **54.02% final target-domain test accuracy** illustrates the practical difficulty of domain adaptation under geographic, temporal, and pseudo-label distribution shifts.

The project therefore focuses not only on the final score, but on understanding **what changes when a model leaves the domain it was originally trained on**.

---

<div align="center">

### TerraAdapt

**Domain Adaptation for Satellite Image Classification**

`EuroSAT` · `ResNet50` · `Sentinel-2` · `Dynamic World`

</div>
