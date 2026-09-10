# Why TerraAdapt?

## Motivation

Satellite imagery is not static.

Landscapes change over time, sensors and processing pipelines evolve, and models trained on one dataset may not generalize well to imagery collected under different conditions.

TerraAdapt explores this problem through a practical **domain adaptation** experiment.

The starting point is an existing ResNet50 model trained on the EuroSAT dataset. That source model achieved **96.56% test accuracy**, with an F1 score of **0.9656** and a macro ROC-AUC of **0.9977**.

Instead of training a completely new model from scratch, TerraAdapt asks a different question:

> **Can an existing satellite vision model be adapted to a newer target domain?**

The target domain consists of modern Sentinel-2 imagery paired with Dynamic World labels. This creates a realistic shift from the original EuroSAT data distribution.

---

## Why Domain Adaptation?

A model can perform very well on the dataset it was trained and evaluated on while still struggling when the input distribution changes.

For remote sensing, domain differences can arise from:

* Different acquisition dates
* Geographic differences
* Changes in land cover
* Differences in image processing
* Different label definitions
* Changes in the visual characteristics of satellite imagery

Training from scratch on every new domain is expensive and may discard useful representations learned from previous data.

TerraAdapt therefore uses **transfer learning and fine-tuning** to reuse the representation learned by the EuroSAT ResNet50 model and adapt it to a new target-domain classification problem.

The goal is not to claim state-of-the-art performance.

The goal is to study the **adaptation process, domain shift, and its limitations**.

---

## Why Start From the EuroSAT Model?

The source EuroSAT model already contains useful visual representations learned from satellite imagery.

The pretrained model uses a ResNet50 architecture with:

```text
23,516,228 total parameters
```

Its original classification head predicts 10 EuroSAT classes.

For TerraAdapt, the original classifier is replaced with a new four-class target-domain classifier:

```text
Linear(2048 → 4)
```

This allows the convolutional representation learned from the source domain to be reused while adapting the final decision boundary to the target domain.

This approach also makes the experiment more meaningful than simply training an independent model on the target dataset because the adaptation begins from an already trained remote-sensing representation.

---

## Why Four Target Classes?

The target dataset uses four broader land-cover categories:

| ID | Target Class |
| -: | ------------ |
|  0 | Water        |
|  1 | Trees        |
|  2 | Crops        |
|  3 | Built        |

These categories provide a simplified target-domain classification problem while allowing related EuroSAT classes to be mapped into broader semantic groups.

The conceptual mapping is:

| Target Class | Source EuroSAT Classes           |
| ------------ | -------------------------------- |
| Water        | River, SeaLake                   |
| Trees        | Forest                           |
| Crops        | AnnualCrop, PermanentCrop        |
| Built        | Residential, Industrial, Highway |

`Pasture` and `HerbaceousVegetation` are excluded from the target mapping.

This creates a target problem with fewer and broader classes than the original EuroSAT task.

---

## Why Use Modern Sentinel-2 Imagery?

The target domain uses Sentinel-2 surface-reflectance imagery from the:

```text
COPERNICUS/S2_SR_HARMONIZED
```

collection.

The imagery covers:

```text
2024-01-01 → 2025-01-01
```

The target dataset contains:

```text
4,057 RGB patches
224 × 224 pixels
```

The purpose is to move beyond the original source-domain evaluation and investigate how a pretrained satellite model behaves when exposed to a newer target-domain dataset.

---

## Why Dynamic World?

The target imagery is paired with:

```text
GOOGLE/DYNAMICWORLD/V1
```

Dynamic World provides land-cover predictions that can be used to construct target-domain labels at scale.

This makes it practical to create a target dataset without manually annotating thousands of satellite patches.

However, these labels are **pseudo-labels, not manually verified ground truth**.

That distinction is important when interpreting the results.

A model may be penalized for disagreeing with a pseudo-label even when its prediction is semantically reasonable. Conversely, agreement with a pseudo-label does not necessarily mean that the prediction is correct according to human-verified land-cover annotations.

Therefore, TerraAdapt treats Dynamic World labels as a practical target-domain supervision source rather than as perfect ground truth.

---

## Why a Spatial Split?

Satellite images located close to one another can be highly similar.

If neighboring patches are randomly distributed between training, validation, and test sets, the model may effectively see very similar geographic regions during training and evaluation.

That can produce an overly optimistic estimate of generalization.

TerraAdapt therefore uses a **spatial split**, keeping spatial groups together when constructing the:

* Training set
* Validation set
* Test set

The final split contains:

| Split      | Samples |
| ---------- | ------: |
| Train      |   2,839 |
| Validation |     609 |
| Test       |     609 |
| Total      |   4,057 |

This is approximately a:

```text
70% / 15% / 15%
```

split.

The spatial separation makes the final evaluation more representative of performance on geographically distinct target regions.

---

## Why Analyze Spectral Indices?

TerraAdapt also analyzes several vegetation and color-related spectral indices:

### NDVI

Normalized Difference Vegetation Index:

```text
NDVI = (B8 - B4) / (B8 + B4)
```

NDVI is commonly used to characterize vegetation using the contrast between near-infrared and red reflectance.

In TerraAdapt, NDVI provides an additional way to inspect the vegetation characteristics of the target patches.

### GNDVI

Green Normalized Difference Vegetation Index:

```text
GNDVI = (B8 - B3) / (B8 + B3)
```

GNDVI uses the green band together with near-infrared reflectance.

It provides another vegetation-sensitive measurement that can help characterize differences between target land-cover categories.

### NGRDI

Normalized Green-Red Difference Index:

```text
NGRDI = (B3 - B4) / (B3 + B4)
```

NGRDI measures the relative difference between green and red reflectance.

It can provide additional information about visible-spectrum characteristics of the target imagery.

---

## Why Are the Indices Not Model Inputs?

NDVI, GNDVI, and NGRDI are used for **analysis and quality-control purposes**, not as inputs to the TerraAdapt ResNet50 model.

The model operates on the RGB target patches.

This separation is intentional.

It allows the experiment to study the target-domain imagery while keeping the adaptation model itself based on the existing RGB ResNet50 pipeline.

Future experiments could investigate whether explicitly providing multispectral bands or derived indices improves adaptation performance.

---

## What TerraAdapt Is Actually Testing

The experiment can be summarized as:

```text
EuroSAT ResNet50
        ↓
Pretrained source representation
        ↓
Replace 10-class classifier
        ↓
Four-class target classifier
        ↓
Fine-tune on target-domain data
        ↓
Evaluate on spatially held-out target regions
```

The adaptation experiments include different depths of fine-tuning.

The initial configuration fine-tunes:

```text
ResNet50 layer4 + classification head
```

with:

```text
Trainable parameters: 14,972,932
Frozen parameters:     8,543,296
```

A deeper configuration additionally fine-tunes `layer3`:

```text
layer3 + layer4 + classification head
```

with:

```text
Trainable parameters: 22,071,300
Frozen parameters:     1,444,928
```

This makes it possible to investigate how much of the pretrained representation needs to adapt to the target domain.

---

## The Important Result

The selected TerraAdapt model performed substantially better on the validation set than on the final untouched test set.

### Validation

| Metric           |  Score |
| ---------------- | -----: |
| Accuracy         | 71.76% |
| Macro Precision  | 74.44% |
| Macro Recall     | 71.91% |
| Macro F1         | 71.79% |
| Macro ROC-AUC    | 0.8682 |
| Weighted ROC-AUC | 0.8687 |

### Final Untouched Test

| Metric           |  Score |
| ---------------- | -----: |
| Accuracy         | 54.02% |
| Macro Precision  | 54.02% |
| Macro Recall     | 54.31% |
| Macro F1         | 53.61% |
| Macro ROC-AUC    | 0.8259 |
| Weighted ROC-AUC | 0.8252 |

The final test performance is therefore **not strong enough to present TerraAdapt as a high-performing land-cover classifier**.

That is an important part of the experiment.

---

## What the Results Tell Us

The difference between validation and final test performance highlights the difficulty of target-domain generalization.

The final test confusion matrix shows that:

* `Built` is the strongest-performing class.
* `Crops` is particularly difficult.
* A substantial number of crop samples are predicted as built.
* Water and trees also show meaningful confusion.

One possible contributor is the combination of **domain shift and pseudo-label noise** in the target dataset.

The results should therefore be interpreted as evidence of a challenging adaptation problem rather than as evidence that the adaptation pipeline has solved the problem.

---

## What TerraAdapt Demonstrates

TerraAdapt demonstrates a complete experimental pipeline for:

1. Starting from a pretrained remote-sensing model.
2. Constructing a new target-domain dataset.
3. Mapping source classes into broader target classes.
4. Using spatially separated train, validation, and test sets.
5. Fine-tuning selected portions of a pretrained ResNet50.
6. Evaluating adaptation using multiple classification metrics.
7. Investigating target-domain characteristics using spectral indices.
8. Analyzing failure cases rather than reporting only the best metric.

The most valuable outcome is not a single accuracy number.

It is the demonstration that **strong source-domain performance does not guarantee strong target-domain generalization**.

---

## Limitations

Several limitations should be considered when interpreting TerraAdapt.

### 1. Dynamic World pseudo-labels

The target labels are derived from Dynamic World and should not be treated as manually verified ground truth.

Label noise can directly affect both training and evaluation.

### 2. Limited target dataset

The target dataset contains 4,057 patches. A larger and more geographically diverse dataset could provide a stronger estimate of target-domain generalization.

### 3. RGB-only model input

The ResNet50 model uses RGB imagery rather than the full Sentinel-2 multispectral information.

Important spectral information contained in additional Sentinel-2 bands is therefore not directly available to the model.

### 4. Domain shift

The source and target datasets differ in their geographic, temporal, and data-generation characteristics.

This makes adaptation inherently difficult.

### 5. No claim of state-of-the-art performance

TerraAdapt is an experimental study of domain adaptation and generalization.

The final test performance should be reported as-is rather than selectively presenting validation performance as evidence of strong generalization.

---

## Research Question

The central question behind TerraAdapt is:

> **How well can a pretrained satellite-image classifier adapt to a new target domain, and what happens when the target domain differs from the original training distribution?**

The answer from this experiment is deliberately nuanced:

**Pretrained representations can provide a useful starting point, but adaptation does not eliminate domain shift.**

The gap between validation and untouched test performance shows why geographic separation, careful evaluation, and awareness of pseudo-label quality are essential in remote-sensing machine learning.

---

## Future Direction

The next generation of TerraAdapt could investigate:

* Full Sentinel-2 multispectral inputs
* Explicit NDVI, GNDVI, and NGRDI model features
* Better pseudo-label filtering and refinement
* Self-training and pseudo-label refinement
* Domain-adversarial training
* Contrastive domain adaptation
* Larger geographic target datasets
* More rigorous geographic cross-validation
* Additional pretrained remote-sensing models
* Comparison between shallow and deep adaptation strategies

These extensions could help determine whether the limitations observed in the current experiment arise primarily from the model, the target-domain shift, the pseudo-labels, or the limited target dataset.

---

## Bottom Line

TerraAdapt is not designed to claim that one fine-tuning strategy solves satellite-domain adaptation.

It is designed to show the **real difficulty of adapting an existing vision model to a new geographic and temporal domain**.

Starting from a strong EuroSAT model and evaluating it on a spatially held-out modern target dataset exposes a key machine-learning lesson:

> **A model that performs extremely well on its source domain can still struggle when the world it sees changes.**
