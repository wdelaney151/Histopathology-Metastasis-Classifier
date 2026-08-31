# Histopathology Metastasis Classification with Transfer Learning

Binary tumor-vs-normal classification on the [PatchCamelyon (PCam)](https://github.com/basveeling/pcam) benchmark using transfer learning and Grad-CAM.

## Problem

Detecting metastatic tissue in lymph node biopsies is an important step in cancer staging. This project builds a patch-level classifier for identifying tumor tissue in 96×96 H&E-stained lymph node images.

Rather than optimizing accuracy alone, the project focuses on data integrity, clinically relevant metrics, and understanding model errors.

## Approach

- **Dataset:** PatchCamelyon (PCam), 327,680 labeled histopathology patches
- **Baseline:** 2-layer CNN trained from scratch
- **Transfer learning:** ImageNet-pretrained ResNet18
- **Fine-tuning:** Frozen early layers; fine-tuned `layer4` and classification head
- **Optimization:** Tested discriminative learning rates and weight decay
- **Augmentation:** Horizontal/vertical flips, rotation, and mild color jitter
- **Interpretability:** Grad-CAM for investigating false negatives

## Data integrity

I verified that the train, validation, and test splits contain no overlapping whole-slide images, reducing the risk of slide-level data leakage.

I also examined the class distribution by slide and confirmed the role of hard-negative mining in creating the approximately balanced dataset.

## Results

Both models were trained using the same augmented dataset and evaluated on the held-out test set.

| Model | Best Val. Accuracy | Test Accuracy | Balanced Accuracy | Macro F1 | Tumor Recall | Tumor Precision |
|---|---|---|---|---|---|---|
| Baseline CNN | 82.75% | 81.2% | 81.2% | 81.1% | 74.0% | 87.0% |
| ResNet18 (Transfer Learning) | 86.09% | 83.3% | 83.3% | 83.2% | 76.0% | 89.0% |

## Final Model

The ResNet18 transfer-learning model outperformed the baseline CNN across the main evaluation metrics:

- 83.3% test accuracy
- 83.3% balanced accuracy
- 83.2% macro F1
- 76.0% tumor recall
- 89.0% tumor precision

The ResNet18 model also achieved a best validation accuracy of 86.09%, compared with 82.75% for the baseline CNN.

Overall, transfer learning provided a consistent improvement over the CNN trained from scratch while both models used the same augmented training pipeline.

## Error Analysis

The results show a consistent precision/recall asymmetry for tumor classification.

The baseline CNN achieved 87% tumor precision but only 74% tumor recall, while the ResNet18 model improved both metrics to 89% precision and 76% recall. In other words, both models were better at identifying normal tissue than detecting every tumor patch.

This pattern is important because accuracy alone does not fully describe the model's behavior. The final ResNet18 model achieved 83.3% accuracy, but still missed approximately one in four tumor patches in the held-out test set.

Grad-CAM was used to investigate where the ResNet18 model focused when making predictions. In false-negative examples, attention often concentrated on visually prominent tissue outside the labeled center region. This is consistent with a limitation of PCam's patch-level labeling rule, where the label depends on whether tumor tissue is present in the central 32×32 region.

The baseline CNN showed a similar precision/recall asymmetry, suggesting that this behavior may be related to the dataset or labeling setup rather than being specific to the ResNet18 architecture.

## Limitations & Next Steps

- Patch-level classification does not replace whole-slide analysis.
- Residual false negatives suggest potential benefits from segmentation or multi-scale approaches.
- Further regularization and fine-tuning strategies could be explored.
- A clinical system would require aggregation across many patches and validation on independent datasets.

## Key Takeaway

The main finding of this project is that transfer learning improved performance over a CNN trained from scratch, but tumor recall remains a challenge.

Using the same augmented data pipeline, ResNet18 improved test accuracy from 81.2% to 83.3%, balanced accuracy from 81.2% to 83.3%, and tumor recall from 74.0% to 76.0%.

More importantly, evaluating class-specific metrics and using Grad-CAM revealed that the remaining errors are not captured by accuracy alone and may be related to the spatial labeling characteristics of the PCam dataset.

The project therefore demonstrates the value of combining transfer learning, targeted augmentation, clinically relevant evaluation metrics, and model interpretability when developing models for digital pathology.
