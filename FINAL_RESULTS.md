# E27 final results (2026-10-20, seed 42 re-run)

## Protocol B: patient-level split (main result)
All 5,856 images pooled and split 80/10/10 **by patient** (3,189 patients, no patient in two splits). Train 4,672 / val 584 / test 600 (160 NORMAL, 440 PNEUMONIA). Threshold chosen on validation only (Youden's J).

| Seed | Accuracy | AUC | Sensitivity | Specificity | TN | FP | FN | TP |
|---|---|---|---|---|---|---|---|---|
| 42 | 0.9700 | 0.9958 | 0.9659 | 0.9812 | 157 | 3 | 15 | 425 |
| 7 | 0.9700 | 0.9950 | 0.9705 | 0.9688 | 155 | 5 | 13 | 427 |
| 123 | 0.9750 | 0.9954 | 0.9864 | 0.9437 | 151 | 9 | 6 | 434 |
| **Mean ± std** | **0.9717 ± 0.0024** | **0.9954 ± 0.0003** | **0.9742 ± 0.0088** | **0.9646 ± 0.0156** | | | | |
| 3-seed ensemble | 0.9717 | 0.9960 | 0.9727 | 0.9688 | 155 | 5 | 12 | 428 |

**Ablation (seed 123):** the warm-up model without SSL scored 0.9533 accuracy (AUC 0.9938). The full SSL pipeline scored 0.9750. That is evidence that the SSL stage adds about 2 points.

## Protocol A: official ChestXRay2017 test set (624 images)
Threshold chosen on the 62 official validation images only.

| Model | Test AUC | Acc @0.5 | Acc @val-threshold | Sens | Spec |
|---|---|---|---|---|---|
| E27 SSL (v5) | 0.9739 | 0.8862 | 0.9103 | 0.979 | 0.795 |
| E27 minor Xception | 0.9749 | 0.8734 | 0.9263 | 0.949 | 0.889 |
| E25 DenseNet121 | 0.9624 | 0.8317 | 0.8654 | 0.985 | 0.667 |
| E26 ResNet50 | 0.9461 | 0.8253 | 0.8798 | 0.926 | 0.803 |
| E29 ResNet50 | 0.9270 | 0.7981 | 0.8349 | 0.962 | 0.624 |
| E11 MobileNetV2 | 0.9303 | 0.7628 | 0.7804 | 0.979 | 0.449 |
| **Ensemble, all 6 (pre-declared)** | **0.9711** | 0.8397 | **0.9183** | 0.956 | 0.855 |

Best possible accuracy with a threshold tuned on the test set (diagnostic only, **not reportable**): about 93% for the best single models. The ceiling on this split comes from the test NORMAL images looking different from the train NORMAL images, not from the method.

## Suggested report wording
> On a patient-level 80/10/10 split (no patient overlap), the E27 Xception SSL pipeline achieves **97.2% ± 0.2% accuracy** (AUC 0.995, sensitivity 97.4%, specificity 96.5%) over three seeds. On the official ChestXRay2017 test split, which has a known train–test distribution shift in NORMAL images, a 6-model ensemble reaches 91.8% accuracy (AUC 0.971) with the threshold chosen on validation only.

## Do not
- Do not quote Protocol B numbers as if they were on the official test set.
- Do not quote the "oracle" lines or the best of the combination rows as results. Those were chosen by looking at the test set.
