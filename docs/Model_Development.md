# Model Development

## Overview

TerraAdapt uses a pretrained **ResNet50 model originally trained on EuroSAT** as the starting point for target-domain adaptation.

Instead of training a new model from scratch, the experiment transfers the learned visual representation from the EuroSAT source domain to a modern Sentinel-2 target domain containing four broader land-cover classes.

The adaptation is performed using **PyTorch**.

## Source Model

The source model is a ResNet50 previously fine-tuned on the EuroSAT dataset.

### Source Checkpoint

```text
resnet50_finetuned_layer4_best.pth
```

The checkpoint contains a 10-class classification head:

```text
ResNet50
    ↓
2048-dimensional feature representation
    ↓
10-class classifier
```

The final classifier parameters have the following shapes:

```text
fc.weight → [10, 2048]
fc.bias   → [10]
```

This confirms that the source model was trained for the original 10-class EuroSAT classification task.

### Source Performance

| Metric        |  Score |
| ------------- | -----: |
| Test Accuracy | 96.56% |
| F1 Score      | 0.9656 |
| Macro ROC-AUC | 0.9977 |

The source model contains:

```text
23,516,228 total parameters
```

The pretrained weights provide the initialization for TerraAdapt rather than training the target-domain model from random initialization.

## Target Classification Head

The original 10-class EuroSAT classifier is not directly compatible with the four-class target problem.

Therefore, the original classification head is replaced with:

```python
Linear(2048, 4)
```

The target classes are:

| ID | Class |
| -: | ----- |
|  0 | Water |
|  1 | Trees |
|  2 | Crops |
|  3 | Built |

The resulting architecture is:

```text
Pretrained ResNet50
        ↓
2048-dimensional representation
        ↓
New 4-class classification head
        ↓
Water / Trees / Crops / Built
```

## Adaptation Strategy

The adaptation experiment progressively fine-tuned deeper portions of the pretrained ResNet50.

The initial configuration fine-tuned:

```text
layer4 + new classification head
```

Earlier ResNet50 layers remained frozen so that the model could retain general visual representations learned from the source domain while adapting higher-level features to the target domain.

### Layer 4 + FC

| Parameter Group |      Count |
| --------------- | ---------: |
| Trainable       | 14,972,932 |
| Frozen          |  8,543,296 |
| Total           | 23,516,228 |

This configuration allows substantial adaptation while preserving a large portion of the pretrained network.

## Deeper Adaptation Experiment

A second configuration additionally fine-tuned `layer3`:

```text
layer3 + layer4 + classification head
```

This increased the number of trainable parameters:

| Configuration          | Trainable Parameters | Frozen Parameters |
| ---------------------- | -------------------: | ----------------: |
| Layer 4 + FC           |           14,972,932 |         8,543,296 |
| Layer 3 + Layer 4 + FC |           22,071,300 |         1,444,928 |

The deeper configuration therefore allows the model to modify a much larger portion of the pretrained representation.

This experiment was useful for examining whether the target-domain shift required adaptation beyond the final ResNet block.

## Training Approach

The target-domain model was trained using the spatially separated training and validation sets.

The validation set was used during model development to monitor adaptation performance and select the best checkpoint.

The final test set was kept untouched until the end of the experiment.

The best TerraAdapt checkpoint was saved as:

```text
terraadapt_resnet50_best.pth
```

## Evaluation Strategy

Model development followed a strict separation between:

```text
Training Set
     ↓
Model Optimization

Validation Set
     ↓
Model Selection / Monitoring

Test Set
     ↓
Final Untouched Evaluation
```

The test set was not used for model selection or training decisions.

This is particularly important for TerraAdapt because the target dataset contains spatially related satellite imagery.

## Why Fine-Tune Instead of Train From Scratch?

Training from scratch would discard the visual representation already learned by the EuroSAT model.

Using the pretrained model provides:

* learned low- and mid-level visual features
* a strong initialization
* fewer parameters requiring adaptation
* a direct way to study transfer between satellite-image domains

The experiment therefore focuses on the question:

> **How well can a model trained on one satellite-imagery domain adapt to a newer target domain?**

## Model Input

The model receives RGB satellite patches:

```text
224 × 224 × 3
```

The RGB channels are constructed from Sentinel-2:

```text
B4 → Red
B3 → Green
B2 → Blue
```

The spectral indices NDVI, GNDVI, and NGRDI are **not model inputs**. They are used only for analysis and quality control.

## Checkpoint Policy

Large trained model files are intentionally not included in the GitHub repository.

The repository documents:

* source model architecture
* adaptation strategy
* parameter counts
* checkpoint names
* training/evaluation workflow

This keeps the repository lightweight while preserving the information required to understand and reproduce the methodology.

## Important Experimental Constraint

TerraAdapt is an adaptation experiment, not a claim that the target-domain model achieves production-level land-cover classification performance.

The final evaluation shows a substantial difference between validation and untouched test performance.

This gap is retained in the project because it provides an important observation about:

* domain shift
* spatial generalization
* pseudo-label noise
* target-domain difficulty

The limitations and final results are discussed in `Model_Evaluation.md`.
