<div align="center">

# TerraAdapt

### Adapting satellite vision models to a changing Earth.

**A domain adaptation experiment using a pretrained EuroSAT ResNet50 model on modern Sentinel-2 imagery and Dynamic World pseudo-labels.**

<br>

![TerraAdapt Banner](figures/terraadapt_banner.png)

<br>

[![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-Deep%20Learning-ee4c2c?logo=pytorch)](https://pytorch.org/)
[![Computer Vision](https://img.shields.io/badge/Computer%20Vision-ResNet50-orange)]()
[![Remote Sensing](https://img.shields.io/badge/Domain-Remote%20Sensing-green)]()
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

</div>

---

## Overview

Satellite imagery changes over time.

A model trained on one satellite-image distribution may perform very differently when applied to imagery from a newer period, different sensor processing pipeline, or different geographic distribution.

**TerraAdapt** explores this problem through a practical domain-adaptation experiment.

The project starts with an existing **10-class EuroSAT ResNet50 model** and adapts its learned representation to a new target domain built from:

* **Sentinel-2 Surface Reflectance Harmonized imagery**
* **Google Dynamic World land-cover predictions**
* RGB satellite patches
* A spatially separated train/validation/test split

Instead of training a new model from scratch, TerraAdapt investigates whether a strong model learned from EuroSAT can be adapted to a modern target domain with a different class structure.

> **This is an experiment in domain shift—not a claim that the adapted model achieves state-of-the-art performance.**

---

## Why TerraAdapt?

A model can perform extremely well on its original benchmark and still struggle when the data distribution changes.

The source model used in this project achieved:

| Metric        | EuroSAT Source Model |
| ------------- | -------------------: |
| Test Accuracy |           **96.56%** |
| F1 Score      |           **0.9656** |
| Macro ROC-AUC |           **0.9977** |

TerraAdapt asks a different question:

> **What happens when this pretrained satellite vision model is moved to a new target domain?**

The target domain differs from EuroSAT in several important ways:

* New imagery period
* Modern Sentinel-2 data
* Different geographic distribution
* Different label-generation process
* Four broader land-cover classes instead of ten EuroSAT classes

This makes the project useful as a study of **transfer learning, domain shift, pseudo-label noise, and remote-sensing model generalization**.

---

# Dataset

## Target Domain

The target dataset contains **4,057 RGB satellite patches**.

| Property     | Value                   |
| ------------ | ----------------------- |
| Images       | **4,057**               |
| Image size   | **224 × 224**           |
| Channels     | **RGB**                 |
| Imagery      | Sentinel-2              |
| Label source | Dynamic World           |
| Time range   | 2024-01-01 → 2025-01-01 |
| Classes      | 4                       |

The target dataset was constructed from Sentinel-2 imagery using Dynamic World land-cover predictions to assign target-domain labels.

### Important

Dynamic World labels are **pseudo-labels**, not manually verified ground truth.

Therefore, the reported target-domain metrics should be interpreted with this limitation in mind.

---

# Target Classes

TerraAdapt reduces the original EuroSAT-style land-cover categories into four broader target classes:

| Target Class | ID |
| ------------ | -: |
| Water        |  0 |
| Trees        |  1 |
| Crops        |  2 |
| Built        |  3 |

The conceptual mapping from EuroSAT is:

```text
River + SeaLake
        ↓
      Water

Forest
   ↓
 Trees

AnnualCrop + PermanentCrop
          ↓
        Crops

Residential + Industrial + Highway
              ↓
            Built
```

`Pasture` and `HerbaceousVegetation` were excluded from the target mapping.

---

# Spatial Data Split

A spatially aware split was used to reduce the possibility of nearby patches appearing across different subsets.

| Split      |   Samples |
| ---------- | --------: |
| Train      | **2,839** |
| Validation |   **609** |
| Test       |   **609** |
| Total      | **4,057** |

Approximately:

```text
70% Train
15% Validation
15% Test
```

Spatial groups were kept together during the split.

The final test set was kept untouched until the final evaluation.

---

# Model

## Source Model

TerraAdapt starts from an existing **ResNet50** model trained on the EuroSAT dataset.

The source model was trained as a **10-class satellite image classifier**.

```text
EuroSAT
   │
   ▼
ResNet50
   │
   ▼
10-class classifier
```

Source checkpoint:

```text
resnet50_finetuned_layer4_best.pth
```

The checkpoint contains the learned ResNet50 weights before adapting the classifier to the four target classes.

### Source Model Parameters

| Parameter                           |          Count |
| ----------------------------------- | -------------: |
| Total parameters                    | **23,516,228** |
| Trainable during initial adaptation | **14,972,932** |
| Frozen                              |  **8,543,296** |

---

# TerraAdapt Adaptation

The source classifier is replaced with a new four-class classification head:

```text
ResNet50
   │
   ├── Layer 1
   ├── Layer 2
   ├── Layer 3
   ├── Layer 4
   │
   ▼
2048-dimensional representation
   │
   ▼
Linear(2048 → 4)
   │
   ▼
Water / Trees / Crops / Built
```

The initial adaptation experiment fine-tuned:

```text
Layer 4 + Classification Head
```

while earlier layers remained frozen.

A deeper adaptation experiment was also performed:

```text
Layer 3 + Layer 4 + Classification Head
```

with:

| Configuration          | Trainable Parameters |
| ---------------------- | -------------------: |
| Layer 4 + FC           |       **14,972,932** |
| Layer 3 + Layer 4 + FC |       **22,071,300** |

The deeper configuration leaves:

**1,444,928 frozen parameters.**

---

# Spectral Indices

Three vegetation/land-cover related indices were calculated during target-domain analysis:

### NDVI

Normalized Difference Vegetation Index:

```text
NDVI = (B8 - B4) / (B8 + B4)
```

Used to characterize vegetation strength.

### GNDVI

Green Normalized Difference Vegetation Index:

```text
GNDVI = (B8 - B3) / (B8 + B3)
```

Used as an additional vegetation-related indicator.

### NGRDI

Normalized Green-Red Difference Index:

```text
NGRDI = (B3 - B4) / (B3 + B4)
```

Used to capture differences between green and red reflectance.

**These indices were used for analysis and quality control, not as model input features.**

The model itself operates on RGB image patches.

---

# Results

## Validation

The best validation performance was:

| Metric           |      Score |
| ---------------- | ---------: |
| Accuracy         | **71.76%** |
| Macro Precision  | **74.44%** |
| Macro Recall     | **71.91%** |
| Macro F1         | **71.79%** |
| Macro ROC-AUC    | **0.8682** |
| Weighted ROC-AUC | **0.8687** |

These results were used during model selection.

---

## Final Untouched Test

After model selection, the final model was evaluated on the untouched test set.

| Metric           |      Score |
| ---------------- | ---------: |
| Accuracy         | **54.02%** |
| Macro Precision  | **54.02%** |
| Macro Recall     | **54.31%** |
| Macro F1         | **53.61%** |
| Macro ROC-AUC    | **0.8259** |
| Weighted ROC-AUC | **0.8252** |

The gap between validation and final test performance is an important result of the experiment.

It demonstrates that good validation performance did **not** translate directly into strong generalization on the final target-domain test set.

---

# Confusion Matrix

Final test confusion matrix:

```text
              Predicted
             W    T    C    B
Actual W    93   13   26   18
       T    34   68   34   14
       C     5   36   56   62
       B     2   16   20  112
```

Where:

```text
W = Water
T = Trees
C = Crops
B = Built
```

### Observations

**Built** was the strongest class on the final test set.

**Crops** was the most difficult class, with substantial confusion between crops and built areas.

This is particularly important because agricultural and developed areas can share similar visual characteristics in RGB satellite imagery.

---

# What Did TerraAdapt Actually Show?

The most important result is not simply the final accuracy.

TerraAdapt demonstrates that:

> **A model that performs very strongly on its source dataset can experience substantial degradation after being transferred to a new target domain.**

The experiment highlights several challenges:

* Domain shift
* Geographic variation
* Temporal variation
* Different class definitions
* Pseudo-label noise
* Limited target-domain data
* Visual similarity between land-cover classes

The final test performance shows that adaptation is **not automatically sufficient for robust target-domain generalization**.

---

# Project Workflow

```text
                 SOURCE DOMAIN
                      │
                      ▼
              EuroSAT ResNet50
                      │
                      │ pretrained weights
                      ▼
              Target Data Setup
                      │
                      ▼
        Sentinel-2 + Dynamic World
                      │
                      ▼
              RGB Patch Extraction
                      │
                      ▼
             Target Class Mapping
                      │
                      ▼
              Spatial Data Split
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
           Train     Val      Test
             │        │        │
             ▼        ▼        │
       ResNet50 Adaptation      │
             │                  │
             ▼                  │
       Model Selection          │
             │                  │
             └──────────┐       │
                        ▼       ▼
                  Final Evaluation
                        │
                        ▼
                  Error Analysis
```

---

# Repository Structure

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
│   ├── Workflow.md
│   └── Future_Work.md
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

# Reproducibility

The main experiment is provided in:

```text
notebooks/TerraAdapt.ipynb
```

The notebook contains the workflow for:

1. Loading target-domain metadata
2. Preparing the target dataset
3. Working with Sentinel-2 imagery
4. Processing Dynamic World labels
5. Constructing RGB patches
6. Creating the spatial split
7. Loading the pretrained EuroSAT ResNet50
8. Replacing the classification head
9. Fine-tuning the target-domain model
10. Selecting the best validation checkpoint
11. Evaluating the untouched test set
12. Generating evaluation metrics and analysis

Large datasets and generated training artifacts are **not included in the repository**.

---

# Data Policy

The repository intentionally does **not** contain the complete target imagery dataset or private/generated test files.

This keeps the repository lightweight and avoids unnecessarily duplicating a large remote-sensing dataset.

The notebook documents the data-generation workflow required to reconstruct the target-domain dataset.

---

# Limitations

TerraAdapt has several important limitations.

### 1. Dynamic World pseudo-labels

The target labels originate from Dynamic World predictions rather than manually verified ground truth.

Therefore, label noise may directly affect adaptation and evaluation.

### 2. Domain mismatch

The source and target domains differ in geography, time period, imagery characteristics, and class definitions.

This makes direct transfer difficult.

### 3. Limited target dataset

The target dataset contains 4,057 patches, which is relatively small for adapting a deep convolutional network.

### 4. Validation-to-test gap

Validation performance was substantially stronger than final test performance.

This indicates that the selected model did not generalize strongly to the untouched test distribution.

### 5. Four broad classes

The target task combines several original land-cover categories into broader groups.

This simplifies the problem but can also introduce visual ambiguity within classes.

### 6. RGB-only model input

Although several spectral indices were analyzed, the model itself uses RGB imagery.

Additional spectral information could potentially improve land-cover discrimination.

---

# Future Work

Potential directions include:

* Incorporating multispectral Sentinel-2 bands
* Using NDVI/GNDVI/NGRDI as model features
* Improving pseudo-label quality
* Confidence-based pseudo-label filtering
* Self-training with target-domain predictions
* Domain-adversarial training
* Feature alignment between source and target domains
* Contrastive representation learning
* Larger geographic target datasets
* More rigorous geographic cross-validation
* Comparing different adaptation depths
* Testing additional pretrained remote-sensing models

The goal would be to determine whether stronger target-domain representation learning can close the gap between validation performance and real-world target-domain generalization.

---

# Key Takeaways

### Source model

**96.56% EuroSAT test accuracy**

↓

### Target-domain adaptation

**Sentinel-2 + Dynamic World**

↓

### Validation

**71.76% accuracy**

↓

### Untouched final test

**54.02% accuracy**

The experiment shows why **domain adaptation matters in remote sensing**.

A model can learn an excellent representation for its original dataset while still struggling when the data distribution changes.

TerraAdapt is therefore best viewed as a **practical investigation of domain shift and adaptation**, rather than simply another satellite-image classification benchmark.

---

# Tech Stack

* **Python**
* **PyTorch**
* **ResNet50**
* **Torchvision**
* **NumPy**
* **Pandas**
* **Scikit-learn**
* **Matplotlib**
* **Google Colab**
* **CUDA**
* **Sentinel-2**
* **Dynamic World**

---

# Citation & References

This project builds upon publicly available remote-sensing datasets and models.

### EuroSAT

Helber, P., Bischke, B., Dengel, A., & Borth, D.
*EuroSAT: A Novel Dataset and Deep Learning Benchmark for Land Use and Land Cover Classification.*

### Sentinel-2

Copernicus Sentinel-2 mission and Sentinel-2 Surface Reflectance imagery.

### Dynamic World

Brown, C. F. et al.
*Dynamic World, Near real-time global 10 m land use land cover mapping.*

---

# License

This project is released under the **MIT License**.

See [LICENSE](LICENSE) for details.

---

<div align="center">

### Built to understand what happens when a strong vision model meets a new Earth.

**TerraAdapt**

</div>

