# Model Files

This directory is reserved for trained model checkpoints used by TerraAdapt.

## Source Model

TerraAdapt starts from a pretrained **ResNet50** model originally trained on EuroSAT.

Source checkpoint:

```text
resnet50_finetuned_layer4_best.pth
```

The source model is a 10-class classifier.

### Source Performance

| Metric        |  Score |
| ------------- | -----: |
| Test Accuracy | 96.56% |
| F1 Score      | 0.9656 |
| Macro ROC-AUC | 0.9977 |

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

## TerraAdapt Model

The source classifier is replaced with a four-class target-domain classifier:

```text
Linear(2048 → 4)
```

Target classes:

```text
0 — Water
1 — Trees
2 — Crops
3 — Built
```

The initial adaptation fine-tuned the final ResNet block (`layer4`) together with the new classification head.

| Parameter Group |      Count |
| --------------- | ---------: |
| Trainable       | 14,972,932 |
| Frozen          |  8,543,296 |
| Total           | 23,516,228 |

A deeper adaptation experiment additionally fine-tuned `layer3`.

| Configuration          | Trainable Parameters |
| ---------------------- | -------------------: |
| Layer 4 + FC           |           14,972,932 |
| Layer 3 + Layer 4 + FC |           22,071,300 |

The deeper configuration leaves **1,444,928 parameters frozen**.

## Checkpoints

Large `.pth` checkpoint files are intentionally **not committed to this repository**.

This keeps the repository lightweight and avoids storing large binary artifacts in Git history.

The notebook documents the model-loading and adaptation procedure required to reproduce the experiment.

## Final TerraAdapt Evaluation

The selected model achieved the following performance on the untouched target-domain test set:

| Metric           |  Score |
| ---------------- | -----: |
| Accuracy         | 54.02% |
| Macro Precision  | 54.02% |
| Macro Recall     | 54.31% |
| Macro F1         | 53.61% |
| Macro ROC-AUC    | 0.8259 |
| Weighted ROC-AUC | 0.8252 |

These results should be interpreted in the context of the target dataset's **Dynamic World pseudo-labels**, domain shift, and geographic/temporal differences from the source domain.
