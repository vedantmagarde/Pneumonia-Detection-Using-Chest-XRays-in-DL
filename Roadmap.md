# Project Roadmap: Pneumonia Detection from Chest X-Rays

This repository tracks the research and development of a pneumonia detection system using chest X-rays. After the exploratory **Minor Project** (30 experiments), the **Major Project** focuses on a single champion pipeline, rigorous evaluation and deployment.

This `Roadmap.md` is a living document. It records what the code actually does, the results it produced, and the next milestones.

---

## Headline Results (October 2026, final)

| Evaluation protocol | Model | Accuracy | AUC | Sensitivity | Specificity |
|---|---|---|---|---|---|
| **Protocol B:** patient-level 80/10/10 split (main result) | E27 Xception SSL, mean of 3 seeds | **97.17% ± 0.24%** | **0.9954 ± 0.0003** | 97.42% ± 0.88% | 96.46% ± 1.56% |
| Protocol B | E27 Xception SSL, 3-seed ensemble | 97.17% | 0.9960 | 97.27% | 96.88% |
| **Protocol A:** official ChestXRay2017 test set (624 images) | 6-model ensemble (pre-declared) | 91.83% | 0.9711 | 95.6% | 85.5% |
| Protocol A | Best single model (E27 minor Xception) | 92.63% | 0.9749 | 94.9% | 88.9% |

- In every row, the decision threshold is chosen on **validation data only** (Youden's J). No number in this table was tuned on test labels.
- **Protocol B** pools all 5,856 images (3,189 patients). It splits them 80/10/10 so that **no patient appears in two splits** (train 4,672 / val 584 / test 600). Train and test therefore come from the same distribution.
- **Protocol A** uses the official `test/` folder. Its NORMAL images differ systematically from the training NORMAL images (a known property of ChestXRay2017). That shift caps every model we trained at about 92–93%, even with a threshold tuned on test. Protocol A and Protocol B numbers are **not interchangeable**, and Protocol B must always be labelled as such.
- **SSL ablation (Protocol B, seed 123):** the warm-up model alone gets 95.33% accuracy and the full SSL pipeline gets 97.50%. The semi-supervised stage adds about 2 points.

Full tables: `FINAL_RESULTS.md`.

---

## Repository Architecture (Final Standard)

```
Minor_Major_Pneumonia/
│
├── Major/                                         # ACTIVE RESEARCH & DEPLOYMENT PHASE
│   ├── custom_E27_CLAHE_Contrast_Xception_P4I3A3M7T4_v7.ipynb          # Champion pipeline (PROTOCOL switch)
│   ├── custom_E27_CLAHE_Contrast_Xception_P4I3A3M7T4_v7_seed42.ipynb   # Protocol B run, seed 42 (with outputs)
│   ├── custom_E27_CLAHE_Contrast_Xception_P4I3A3M7T4_v7_seed7.ipynb    # Protocol B run, seed 7 (with outputs)
│   ├── custom_E27_CLAHE_Contrast_Xception_P4I3A3M7T4_v7_seed123.ipynb  # Protocol B run, seed 123 (with outputs)
│   ├── E27_ensemble_eval_v2.ipynb                 # Protocol A multi-model ensemble (inference only)
│   ├── seed_results.ipynb                         # Protocol B per-seed metrics, mean ± std, seed ensemble
│   ├── ChestXRay2017/chest_xray/
│   │   ├── train/ (1,318 NORMAL, 3,852 PNEUMONIA)
│   │   ├── val/   (31 NORMAL, 31 PNEUMONIA)
│   │   └── test/  (234 NORMAL, 390 PNEUMONIA)
│   └── H5/
│       ├── custom_E27_..._L1PNEU.keras (+ _labels.json)              # Protocol A model (v5/v6 run)
│       ├── custom_E27_..._L1PNEU_RESPLIT_seed{42,7,123}.keras        # Protocol B models, one per seed
│       ├── custom_E27_..._L1PNEU_RESPLIT_seed42_labels.json          # Protocol B label sidecar (seed 42)
│       ├── custom_E27_..._L1PNEU_RESPLIT.keras (+ _labels.json)      # copy of the last run (seed 42)
│       ├── preds_patient_resplit_seed{42,7,123}.npz                  # Protocol B val/test predictions (evidence)
│       ├── preds_seed42.npz                                          # Protocol A v5 predictions
│       └── ensemble_preds/                                           # Protocol A per-model predictions + detected settings
│
├── Minor/                                         # FROZEN ARCHIVE (E1–E30)
│   ├── Notebooks/   (E1 … E30_Ensemble_Final_M8.ipynb)
│   ├── H5/          (E1 … E29 checkpoints; E11, E25, E26, E27, E29 used in the ensemble)
│   └── ChestXRay2017/
│
├── Roadmap.md · FINAL_RESULTS.md · Variations.txt · requirements.txt · open_minor_project_gpu.txt
```

**Paths:** root `C:\CODES__SSD512\Minor_Major_Pneumonia`. Data in `Major/ChestXRay2017/chest_xray/{train,val,test}`. Major weights in `Major/H5/`, minor weights in `Minor/H5/`.

---

## Minor Project (Completed): 30 Experiments, E1–E30

1. **Baseline stabilization (E1–E8):** custom CNNs; 150 vs. 224 px; normalization vs. CLAHE vs. CLAHE + contrast; augmentation levels; class weights.
2. **Transfer learning at 224 px (E9–E23):** MobileNetV2, EfficientNetB0, DenseNet121, ResNet50, Xception; head-only vs. partial unfreezing.
3. **Regularization and imbalance (E24–E29):** medium augmentation and class-balanced focal loss.
4. **Ensembling (E30):** soft-voting ensemble of the top models.

**Outcome:** E27 Xception (CLAHE + contrast) chosen as champion.

---

## Major Project

### Phase 1: Champion Selection (Completed)
- [x] Consolidated metrics from all 30 runs and selected **E27 Xception (CLAHE + contrast)**.

### Phase 2: Training Pipeline (Completed)
- [x] **Label convention `1 = PNEUMONIA`, `0 = NORMAL`:**
  - `LABEL_MAP = {'NORMAL': 0, 'PNEUMONIA': 1}` and `CLASS_NAMES = ['NORMAL', 'PNEUMONIA']`.
  - `FOLDER_ORDER = ['PNEUMONIA', 'NORMAL']`, kept identical to the original notebook so data splits stay reproducible. The iteration order does not affect the labels.
  - The sigmoid output is P(PNEUMONIA). A sanity-check cell asserts the class counts on every split.
- [x] **Reproducibility:** fixed seeds (`GLOBAL_SEED` for Python, NumPy and TensorFlow). `SPLIT_SEED = 42` fixes the data split, so different seeds share the same train/val/test.
- [x] **Evaluation protocols:** a `PROTOCOL` switch selects `"official"` (Protocol A) or `"patient_resplit"` (Protocol B). Protocol B groups images by patient ID parsed from file names (`personXXX_…`, `IM-XXXX-…`) using `StratifiedGroupKFold`, and asserts there is no patient overlap.
- [x] **Validation set:**
  - Protocol A: the 62 official val images plus a stratified 10% hold-out from train (579 images).
  - Protocol B: a patient-level 10% split (584 images).
- [x] **Input pipeline:**
  - CLAHE (clip 2.0, 8×8) plus contrast ×1.2 at 224×224, stored as `uint8` in RAM (~0.8 GB instead of ~3.1 GB as float32).
  - Images are scaled per batch to Xception's ImageNet range [-1, 1].
- [x] **Augmentation:**
  - Geometric: rotation 15°, shift 10%, zoom 10%, horizontal flip.
  - Intensity (training only): gamma 0.8–1.25, contrast ±15%, brightness ±12.
- [x] **Model:** Xception (ImageNet), top 30 layers trainable, BatchNorm frozen, then GAP → Dense(128, ReLU) → Dropout(0.3) → Dense(1, sigmoid).
- [x] **Custom optimizer, AdaMed:** Adam-style moments with a gradient–momentum cosine-alignment term (ρ_t) and alignment-adaptive weight decay.
- [x] **Class-balanced focal loss:** γ = 2. The runs reported above used `ALPHA_MODE = "previous"`, where α = the PNEUMONIA share of the training data (≈0.73 under Protocol B). A `"balanced"` option (α = 0.5) is available.
- [x] **Semi-supervised learning:**
  - Warm-up (Phase A, 21 epochs) on a 30% labelled subset D_L.
  - Then 6 pseudo-labelling cycles × 14 epochs on the remaining 70% (D_U). Thresholds: PNEUMONIA if P ≥ 0.92, NORMAL if P ≤ 0.15. Pseudo-label weight λ ramps up exponentially across cycles.
  - The best validation epoch of each cycle is restored and used as the teacher for the next cycle.
  - A pseudo-label error count is printed as a diagnostic. It uses the hidden D_U labels, which are never used for training.
- [x] **MixUp** (Beta(0.2, 0.2)) on labelled samples only.
- [x] **Progressive unfreezing:** the top 30 Xception layers are unfrozen from SSL cycle 2, with the learning rate lowered to 1e-5.

### Phase 3: Evaluation & Explainability (Completed)
- [x] **Decision threshold:** Youden's J on validation TTA predictions only. Test results are also printed at the default 0.5 for reference.
- [x] **Test-time augmentation:** 5 views (1 clean + 4 light augmentations: rotation 5°, shift/zoom 5%, flip), **averaged with equal weights**.
- [x] **Clinical metrics:** accuracy, AUC, sensitivity, specificity, PPV, NPV, F1, confusion-matrix heatmap and ROC curve.
- [x] **Grad-CAM** on `block14_sepconv2_act`, using gradients of P(PNEUMONIA).
- [x] **Model versioning:** `_L1PNEU` (Protocol A) and `_L1PNEU_RESPLIT` (Protocol B). Each has a JSON sidecar recording the label map, the validation threshold, the input scaling and the protocol.
- [x] **Multi-seed evaluation (Protocol B):** seeds 42, 7 and 123. `seed_results.ipynb` reports per-seed metrics, mean ± std, and the seed ensemble.
- [x] **Multi-model ensemble (Protocol A):**
  - `E27_ensemble_eval_v2.ipynb` combines E27 v5 with the minor models E27 Xception, E25 DenseNet121, E26/E29 ResNet50 and E11 MobileNetV2.
  - Each minor model's label direction and input scaling are detected automatically from the 62 official val images, and its threshold is set on those same images.
  - Averaging uses mean or rank-mean.
- [x] **Ablation:** warm-up-only vs. full SSL is reported in every run.

### Phase 4: Next Steps
- [ ] **Error analysis:** Grad-CAM on the Protocol B false positives and false negatives, and on the Protocol A false positives (NORMAL images called PNEUMONIA).
- [ ] **Lung segmentation or cropping:** remove borders, text markers and devices. This is the most promising way to narrow the Protocol A gap.
- [ ] **Cross-dataset check** (e.g. a sample of NIH ChestX-ray14) to measure generalization.
- [ ] **Inference API (`inference.py`):** a single X-ray in, a JSON out with P(PNEUMONIA), the label from the sidecar threshold, and a Grad-CAM overlay.
- [ ] **Clinical web app:** Streamlit or FastAPI front-end on top of `inference.py`.

---

## Reporting Rules
1. Always state the protocol next to a number: "97.1% (patient-level split)" or "91.8% (official test set)".
2. Thresholds and checkpoints come from validation only. Never report a threshold tuned on test or "oracle" accuracy.
3. Report mean ± std over seeds plus AUC, sensitivity and specificity, not accuracy alone.
4. Keep the `preds_*.npz` files. They are the evidence behind every reported number.

---
*Keep this file updated as Major Project tasks are completed. Use it as the central reference in meetings with the project guide.*
