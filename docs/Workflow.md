# Workflow

TerraAdapt follows a simple end-to-end domain adaptation pipeline.

## 🛰️ 1. Build the Target Dataset

Modern Sentinel-2 imagery is combined with Dynamic World predictions to construct the target-domain dataset.

```text
Sentinel-2
    +
Dynamic World
    ↓
Candidate Selection
    ↓
4,057 RGB Patches
```

Dynamic World predictions are treated as **pseudo-labels**, not verified ground truth.

## 🗺️ 2. Define Target Classes

The target labels are consolidated into four broader classes:

```text
Water
Trees
Crops
Built
```

This creates a four-class target-domain problem compatible with the adaptation experiment.

## 🔬 3. Quality Analysis

The target imagery is inspected using:

* NDVI
* GNDVI
* NGRDI

These indices are used for **analysis and quality control only**.

They are not model inputs.

## 📍 4. Spatial Split

To reduce spatial leakage, samples are grouped spatially before splitting.

```text
4,057 patches
      ↓
┌────────────┬────────────┬────────────┐
│   Train    │ Validation │    Test    │
│   2,839    │    609     │    609     │
│    70%     │    15%     │    15%     │
└────────────┴────────────┴────────────┘
```

The test set remains untouched during model development.

## 🧠 5. Start From the Source Model

TerraAdapt begins with the pretrained **EuroSAT ResNet50** model.

```text
EuroSAT ResNet50
      ↓
Pretrained visual representation
      ↓
Replace 10-class head
      ↓
New 4-class head
```

The model is adapted rather than trained from scratch.

## 🔧 6. Fine-Tune

The adaptation progressively modifies deeper layers of the network.

### Initial configuration

```text
Layer 4 + Classification Head
```

### Deeper experiment

```text
Layer 3 + Layer 4 + Classification Head
```

This tests whether adapting more of the pretrained representation improves transfer to the target domain.

## 📊 7. Validate

The validation set is used to monitor model performance and select the best checkpoint.

Key validation metrics include:

* Accuracy
* Macro Precision
* Macro Recall
* Macro F1
* Macro ROC-AUC

## 🔒 8. Final Test

After model development is complete, the untouched spatial test set is evaluated once.

```text
Best Validation Checkpoint
          ↓
   Untouched Test Set
          ↓
   Final Evaluation
```

The final test results are reported without hiding the observed generalization gap.

## 🔎 The Complete Pipeline

```text
🛰️ Sentinel-2 + Dynamic World
              ↓
       Candidate Selection
              ↓
        4,057 RGB Patches
              ↓
      Quality Analysis
     NDVI / GNDVI / NGRDI
              ↓
        Spatial Split
              ↓
     Train / Validation / Test
              ↓
     Pretrained EuroSAT
          ResNet50
              ↓
      4-Class Adaptation
              ↓
       Validation & Selection
              ↓
      🔒 Untouched Test
              ↓
       Final Evaluation
```

## Reproducibility

The complete workflow is documented in the accompanying notebook:

```text
notebooks/TerraAdapt.ipynb
```

The notebook contains the data construction, spatial splitting, model adaptation, validation analysis, and final test evaluation used in this project.
