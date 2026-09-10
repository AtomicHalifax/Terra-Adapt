# Model Evaluation

## Overview

TerraAdapt was evaluated using a held-out validation set during model development and a completely untouched spatial test set for the final evaluation.

The results show a meaningful gap between validation and final test performance. Rather than treating this as a failure to hide, TerraAdapt uses the gap to demonstrate the difficulty of transferring a satellite vision model across domains.

## Validation Results

The best validation performance achieved during adaptation was:

| Metric           |  Score |
| ---------------- | -----: |
| Accuracy         | 71.76% |
| Macro Precision  | 0.7444 |
| Macro Recall     | 0.7191 |
| Macro F1         | 0.7179 |
| Macro ROC-AUC    | 0.8682 |
| Weighted ROC-AUC | 0.8687 |

The validation set was used during model development and checkpoint selection.

## Final Test Results

The final test set was kept untouched until the end of the experiment.

The final evaluation produced:

| Metric           |  Score |
| ---------------- | -----: |
| Accuracy         | 54.02% |
| Macro Precision  | 0.5402 |
| Macro Recall     | 0.5431 |
| Macro F1         | 0.5361 |
| Macro ROC-AUC    | 0.8259 |
| Weighted ROC-AUC | 0.8252 |

The test results therefore show substantially weaker generalization than the validation results.

## Validation vs Test

| Metric           | Validation | Final Test |
| ---------------- | ---------: | ---------: |
| Accuracy         |     71.76% |     54.02% |
| Macro Precision  |     0.7444 |     0.5402 |
| Macro Recall     |     0.7191 |     0.5431 |
| Macro F1         |     0.7179 |     0.5361 |
| Macro ROC-AUC    |     0.8682 |     0.8259 |
| Weighted ROC-AUC |     0.8687 |     0.8252 |

The difference indicates that performance on the validation regions does not fully translate to the unseen test regions.

This is an important result for the project because the test set was spatially separated and was not used for model selection.

## Final Test Confusion Matrix

The final test confusion matrix is:

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

## Class-Level Interpretation

### Water

Water achieves relatively strong recognition compared with some of the other classes.

However, some water samples are confused with trees, crops, and built surfaces.

### Trees

Trees show moderate recognition but have noticeable confusion with water and crops.

This suggests that vegetation and surrounding land-cover characteristics can overlap visually in RGB satellite imagery.

### Crops

Crops are the most difficult class in the final test evaluation.

A major source of error is:

```text
Crops → Built
```

with 62 crop samples predicted as built.

There is also substantial confusion between crops and trees.

This may reflect the visual similarity of agricultural areas to other land-cover types, together with the difficulty of assigning broad labels from RGB imagery.

### Built

Built is the strongest-performing class in the final confusion matrix.

112 built samples are correctly classified, while relatively few are predicted as water.

This suggests that built environments contain visual characteristics that are more distinguishable from the other target classes.

## Understanding the Validation–Test Gap

The validation accuracy of 71.76% falls to 54.02% on the untouched test set.

Several factors may contribute to this difference.

### 1. Domain Shift

The original ResNet50 was trained on EuroSAT imagery, while TerraAdapt operates on a different target-domain dataset constructed from modern Sentinel-2 imagery.

Differences can occur in:

* geographic regions
* image acquisition conditions
* temporal distribution
* visual appearance
* land-cover composition
* labeling characteristics

The pretrained representation therefore does not necessarily transfer perfectly to the new domain.

### 2. Spatial Generalization

Satellite imagery contains strong spatial correlations.

Even when a spatial split is used, different geographic regions can contain substantially different visual patterns.

The final test set therefore provides a more challenging measure of geographic generalization than simply evaluating randomly sampled neighboring patches.

### 3. Dynamic World Pseudo-Labels

The target labels originate from Dynamic World predictions rather than manually verified ground truth.

Therefore, some target samples may contain label noise.

For example, a patch assigned to `crops` may contain mixed land cover or may not visually correspond cleanly to the simplified four-class mapping.

This can affect both training and evaluation.

### 4. Class Consolidation

The four target classes are broader than the original EuroSAT categories.

Several source categories are mapped into a single target category:

```text
Water  ← River + SeaLake
Trees  ← Forest
Crops  ← AnnualCrop + PermanentCrop
Built  ← Residential + Industrial + Highway
```

This simplifies the target problem but can also introduce heterogeneous visual patterns within a single class.

## ROC-AUC Interpretation

The final macro ROC-AUC is:

```text
0.8259
```

while the weighted ROC-AUC is:

```text
0.8252
```

ROC-AUC remains substantially above random discrimination despite the relatively low final classification accuracy.

This indicates that the model retains useful ranking/discriminative information even though the final class predictions are not sufficiently reliable for strong target-domain classification performance.

## What the Results Demonstrate

The central result of TerraAdapt is not simply a classification score.

The experiment demonstrates that:

```text
Strong source-domain performance
            ↓
      Pretrained model
            ↓
      Target adaptation
            ↓
      Spatially unseen data
            ↓
Performance can degrade substantially
```

The source EuroSAT model achieved 96.56% test accuracy, while the adapted model achieved 54.02% accuracy on the final target-domain test set.

This provides a concrete example of why high performance on a source dataset does not guarantee strong generalization to a different satellite-imagery domain.

## Limitations

The evaluation has several important limitations:

1. **Dynamic World labels are pseudo-labels**, not manually verified ground truth.
2. The target dataset contains only **4,057 patches**.
3. The target problem uses only RGB imagery as model input.
4. NDVI, GNDVI, and NGRDI were used for analysis and quality control but were not provided to the model.
5. The target classes are consolidated from broader land-cover categories.
6. Geographic and temporal differences between source and target domains introduce domain shift.
7. The validation-to-test gap indicates limited generalization to the held-out spatial regions.

## Conclusion

TerraAdapt should not be interpreted as a production-ready land-cover classifier.

Instead, it is a **domain adaptation experiment** showing how a strong pretrained satellite vision model can encounter substantial performance degradation when transferred to a new target domain.

The final test result provides evidence that successful source-domain learning does not automatically translate into robust target-domain generalization.

This motivates future experiments involving improved pseudo-label filtering, stronger domain adaptation methods, multispectral inputs, and independently verified target labels.
