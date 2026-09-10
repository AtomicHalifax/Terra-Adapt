````markdown
# TerraAdapt

**Domain Adaptation for Satellite Image Classification**

![TerraAdapt Banner](figures/terraadapt_banner.png)

TerraAdapt explores how a pretrained land-cover classification model performs when transferred from its original **EuroSAT** domain to a new **Sentinel-2 + Dynamic World** target domain.

The project starts with a pretrained **ResNet50** model and adapts it from 10 EuroSAT classes to four broader target classes: **Water, Trees, Crops, and Built**.

---

## Overview

A strong source-domain model does not necessarily generalize to a different geographic, temporal, and label distribution.

TerraAdapt investigates this domain shift using a practical transfer-learning pipeline:

```text
EuroSAT
   │
   ▼
Pretrained ResNet50
   │
   ▼
10-class → 4-class head
   │
   ▼
Target-domain adaptation
   │
   ▼
Sentinel-2 + Dynamic World
   │
   ▼
Water · Trees · Crops · Built
````

The target dataset contains **4,057 RGB patches** generated from Sentinel-2 imagery and labeled using Dynamic World pseudo-labels.

---

## Dataset

### Target Dataset

| Property   | Value                   |
| ---------- | ----------------------- |
| Images     | 4,057                   |
| Image Size | 224 × 224               |
| Input      | RGB                     |
| Imagery    | Sentinel-2              |
| Labels     | Dynamic World           |
| Classes    | 4                       |
| Date Range | 2024-01-01 → 2025-01-01 |

Sentinel-2 imagery was obtained from:

`COPERNICUS/S2_SR_HARMONIZED`

Dynamic World was used to construct target-domain pseudo-labels:

`GOOGLE/DYNAMICWORLD/V1`

Dynamic World labels are treated as **pseudo-labels rather than manually verified ground truth**.

### Target Classes

| Target Class | Source Categories                  |
| ------------ | ---------------------------------- |
| Water        | River + SeaLake                    |
| Trees        | Forest                             |
| Crops        | AnnualCrop + PermanentCrop         |
| Built        | Residential + Industrial + Highway |

`Pasture` and `HerbaceousVegetation` were excluded from the target mapping.

### Class Mapping

```python
class_to_idx = {
    "water": 0,
    "trees": 1,
    "crops": 2,
    "built": 3
}
```

---

## Spatial Split

To reduce geographic leakage, the target dataset was split spatially rather than using a purely random image-level split.

| Split      | Samples |
| ---------- | ------: |
| Train      |   2,839 |
| Validation |     609 |
| Test       |     609 |
| Total      |   4,057 |

The final test set remained untouched until final evaluation.

---

## Spectral Analysis

Three spectral indices were calculated during dataset analysis and quality control:

### NDVI

```text
NDVI = (B8 - B4) / (B8 + B4)
```

NDVI provides a vegetation-related signal.

### GNDVI

```text
GNDVI = (B8 - B3) / (B8 + B3)
```

GNDVI provides another vegetation-sensitive measurement using the green band.

### NGRDI

```text
NGRDI = (B3 - B4) / (B3 + B4)
```

NGRDI measures green-red spectral contrast.

These indices were used for **analysis and quality control only**.

They were **not used as model inputs**.

The model receives RGB imagery.

---

## Model

TerraAdapt uses **ResNet50** as the backbone.

The starting point is an existing ResNet50 model trained on the EuroSAT dataset.

### Source Model

```text
ResNet50
    │
    ▼
2048-dimensional feature representation
    │
    ▼
10-class classifier
```

Total parameters:

**23,516,228**

The source checkpoint contains a 10-class classifier:

```text
fc.weight → [10, 2048]
fc.bias   → [10]
```

### Source Performance

| Metric        |      Score |
| ------------- | ---------: |
| Test Accuracy | **96.56%** |
| F1 Score      | **0.9656** |
| Macro ROC-AUC | **0.9977** |

---

## Adaptation

The original 10-class classifier was replaced with:

```text
Linear(2048 → 4)
```

The four target classes are:

```text
Water
Trees
Crops
Built
```

### Experiment 1

The first adaptation configuration fine-tuned:

```text
ResNet50 layer4
+
Target classifier
```

| Parameter Group |      Count |
| --------------- | ---------: |
| Trainable       | 14,972,932 |
| Frozen          |  8,543,296 |
| Total           | 23,516,228 |

### Experiment 2

A deeper configuration additionally fine-tuned `layer3`:

```text
ResNet50 layer3
+
ResNet50 layer4
+
Target classifier
```

| Parameter Group |      Count |
| --------------- | ---------: |
| Trainable       | 22,071,300 |
| Frozen          |  1,444,928 |
| Total           | 23,516,228 |

The best validation checkpoint was selected for final testing.

---

## Results

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

The selected model was evaluated once on the untouched target-domain test set.

| Metric           |      Score |
| ---------------- | ---------: |
| Accuracy         | **54.02%** |
| Macro Precision  | **0.5402** |
| Macro Recall     | **0.5431** |
| Macro F1         | **0.5361** |
| Macro ROC-AUC    | **0.8259** |
| Weighted ROC-AUC | **0.8252** |

### Confusion Matrix

```text
                 Predicted

              Water Trees Crops Built

Actual Water    93    13    26    18
       Trees    34    68    34    14
       Crops     5    36    56    62
       Built     2    16    20   112
```

Built performed strongest on the final test set, while Crops showed substantial confusion, particularly with Built.

---

## Source vs Target

One of the main observations from TerraAdapt is the difference between source-domain and target-domain performance.

| Model                  |   Accuracy |
| ---------------------- | ---------: |
| EuroSAT source model   | **96.56%** |
| TerraAdapt target test | **54.02%** |

The result demonstrates the difficulty of transferring a model across different remote-sensing domains.

The target imagery differs from the original training distribution in geographic coverage, acquisition period, dataset construction, and label definitions.

---

## Limitations

The target labels come from **Dynamic World pseudo-labels**, so they should not be treated as equivalent to manually verified ground truth.

Other important limitations include:

* Geographic domain shift
* Temporal domain shift
* Differences between EuroSAT and target class definitions
* Pseudo-label noise
* Limited target-domain sample size
* Spatially separated evaluation
* Confusion between visually similar land-cover classes

Therefore, the final model should be viewed as an **experimental domain-adaptation model**, not a production-ready land-cover classifier.

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
    ├── terraadapt_banner.png
    ├── dataset_samples.png
    ├── spatial_split.png
    ├── validation_confusion_matrix.png
    ├── validation_metrics.png
    └── validation_roc_auc.png
```

---

## Documentation

Detailed project documentation is available in [`docs/`](docs/).

| Document                                            | Description                             |
| --------------------------------------------------- | --------------------------------------- |
| [`why.md`](docs/why.md)                             | Project motivation and research framing |
| [`Data_Collection.md`](docs/Data_Collection.md)     | Target dataset construction             |
| [`Feature_Catalog.md`](docs/Feature_Catalog.md)     | Input features and spectral indices     |
| [`Model_Development.md`](docs/Model_Development.md) | Architecture and adaptation             |
| [`Model_Evaluation.md`](docs/Model_Evaluation.md)   | Validation and test evaluation          |
| [`Workflow.md`](docs/Workflow.md)                   | End-to-end workflow                     |

---

## Reproducibility

The complete workflow is available in:

[`notebooks/TerraAdapt.ipynb`](notebooks/TerraAdapt.ipynb)

The notebook covers:

* Target dataset construction
* Candidate selection
* Dataset analysis
* Spatial splitting
* ResNet50 initialization
* Classifier replacement
* Adaptation experiments
* Validation
* Model selection
* Final test evaluation

Large datasets and model checkpoints are intentionally excluded from the repository.

---

## Requirements

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

Install dependencies with:

```bash
pip install -r requirements.txt
```

---

## License

This project is licensed under the MIT License.

````
e banner look like a proper full-width visual and have GitHub render it reliably. The EuroSAT repo itself uses this exact general pattern of putting a visual image directly under the project heading, which is much closer to what you're asking for.
