# ADRD Dementia Detection in the Emergency Department — EDDA Replication

Replicating the Emergency Department Dementia Algorithm (EDDA) from Cohen et al. (2025, *Alzheimer's & Dementia*) on a simulated dataset.

---

## Background

Older adults with dementia are twice as likely to visit the emergency department (ED) and 1.5 times more likely to have avoidable visits. Despite this, dementia is frequently missed in the ED — clinicians are busy, documentation is incomplete, and formal cognitive screening rarely happens under time pressure.

The EDDA study (Cohen et al. 2025) showed that structured EHR data already collected at the time of an ED visit — medications, prior visits, diagnosis history, lab values — can identify patients at risk of dementia without any additional clinical workload. The original model was trained on 759,665 ED visits across nine Yale New Haven Health hospitals and achieved an AUROC of 0.85 on an external clinician-adjudicated test set.

This project replicates that pipeline on a simulated dataset to demonstrate the analytical approach and test how well the method generalizes to a new health system context.

---

## What this project does

- Builds an XGBoost classifier using the same feature domains as EDDA: medications, prior ED utilization, hospitalization history, diagnosis codes, lab values, and comorbidity scores
- Follows the original paper's Table 1 reporting format (cohort characteristics by outcome, and by train/test split)
- Uses patient-level train/test splitting — all visits from a given patient stay on one side of the split, preventing data leakage
- Handles class imbalance (~22% dementia prevalence) using `scale_pos_weight`
- Produces a threshold analysis table mirroring EDDA Table 3: sensitivity, specificity, PPV, NPV, and number-needed-to-treat across probability cutoffs 0.1–0.9
- Runs SHAP analysis to show which features drive risk predictions
- Checks model performance separately by sex, race/ethnicity, and insurance status

---

## Features

The EDDA paper identified 20 top predictors from 1,641 candidate features. This project uses the same feature domains:

| Feature | Type |
|---|---|
| Geriatric outpatient specialist visits | Count |
| Dementia drug prescriptions (any) | Yes/No |
| Psychotherapeutic drug prescriptions | Yes/No |
| Autonomic drug prescriptions | Yes/No |
| Prior ED visits in the past 2 years | Count |
| History of falls | Yes/No |
| History of Parkinson's disease | Yes/No |
| History of schizophrenia/psychotic disorder | Yes/No |
| Elixhauser comorbidity score | Score |
| Charlson comorbidity score | Score |
| Age | Continuous |
| Prior hospitalizations (24 months) | Count |
| Neurology visits | Count |
| Mean platelet volume (lab) | Continuous |
| Mean corpuscular hemoglobin concentration (lab) | Continuous |
| Head CT ordered in ED (past year) | Yes/No |
| Memantine prescription | Yes/No |
| Donepezil prescription | Yes/No |
| Other procedures in ED (past year) | Count |
| No previous discharge disposition on record | Yes/No |

Missing lab values (up to ~30% in some fields) are handled with median imputation. Categorical variables use mode imputation and one-hot encoding.

---

## Data

This project uses a **simulated EHR dataset** that mirrors the structure of real ED visit data, including lab results, outpatient medication records, prior utilization, and structured diagnosis history. Simulation allows the full pipeline to be shared openly without patient privacy constraints.

The original EDDA study used de-identified EHR data from nine hospitals in the Yale New Haven Health system (January 2014 – March 2022), covering 213,281 older adults aged 65+. Dementia prevalence in that cohort was 12.5% using ICD-based definition.

---

## Methods

**Outcome:** Binary dementia diagnosis (`dementia_outcome`). Defined as presence of two or more historical ICD-9/ICD-10 dementia codes prior to the index ED visit, following the Bynum definition standard used in the original paper.

**Split:** 80% of patients in training, 20% in test. Split is on patient ID, not on individual visits, to prevent the same patient from appearing in both sets.

**Model:** XGBoost with `scale_pos_weight` to account for class imbalance. Preprocessing pipeline includes median imputation for numeric features and one-hot encoding for categorical ones.

**Evaluation:**
- AUC-ROC and AUC-PR (precision-recall AUC) as primary metrics
- Calibration curve and Brier score
- Threshold table (sensitivity / specificity / PPV / NPV / NNT) to support clinical operating point selection
- Subgroup AUC by sex, race/ethnicity, and insurance status

---

## Key results

| Metric | Value |
|---|---|
| AUC-ROC | *0.696* |
| AUC-PR | *0.403* |
| Brier score | *0.218* |

Top predictors by SHAP importance: *dementia drug Rx, prior ED visits, Elixhauser score, fall history, age*


---

## Figures

| File | What it shows |
|---|---|
| `roc_pr_curves.png` | ROC and precision-recall curves |
| `feature_importance.png` | Top 15 features by XGBoost gain |
| `shap_beeswarm.png` | SHAP values — which features push risk up or down |
| `calibration_curve.png` | Predicted vs. observed dementia probability |
| `fairness_analysis.png` | AUC-ROC by sex, race/ethnicity, and insurance |

---

## Clinical context

The original EDDA paper found that 17% of patients flagged by the model had probable undiagnosed dementia — cases that would have been missed by ICD codes alone. The algorithm needs as few as 20 structured EHR features and is designed to run at the time of an ED visit without adding to clinician workload.

At threshold 0.1, EDDA in the original study achieved sensitivity of 0.57 — comparable to commonly used primary care cognitive screening tools — while also surfacing undiagnosed cases. The threshold analysis in this project mirrors that table (EDDA Table 3) to allow direct comparison.

Potential uses in practice include: flagging patients for a brief cognitive assessment before discharge, triggering geriatric consultation, or routing patients into memory clinic referral pathways.

---

## Limitations

**Simulated data.** Results are illustrative of the pipeline, not estimates of real-world performance. Prospective validation on actual EHR data from a health system is required before any clinical use.

**ICD-based outcome.** The dementia label relies on documented diagnosis codes, which undercount true dementia prevalence — especially in Black and Hispanic patients, where diagnosis disparities are well documented. The original EDDA study addressed this with a clinician-adjudicated gold standard test set; this replication does not have that resource.

**Single-site generalizability.** The original EDDA model was trained at Yale New Haven Health. Performance may differ across health systems with different patient populations, coding practices, and EHR structures.

---

## Reference

Cohen I, Taylor RA, Xue H, et al. Detection of emergency department patients at risk of dementia through artificial intelligence. *Alzheimer's & Dementia*. 2025;21:e70334. https://doi.org/10.1002/alz.70334

---

## Repository structure

```
adrd-prediction-xgboost/
├── adrd_prediction_xgboost_yw_v2.ipynb
├── figures/
│   ├── roc_pr_curves.png
│   ├── feature_importance.png
│   ├── shap_beeswarm.png
│   ├── calibration_curve.png
│   └── fairness_analysis.png
└── README.md
```

---

## About

[Yihan Wang](https://honoluluhan717.github.io/yihanwang-site/) is a Research Scientist II at the Icahn School of Medicine at Mount Sinai, Brookdale Department of Geriatrics and Palliative Medicine. Her work focuses on Medicare claims analytics, causal inference, and longitudinal modeling in aging and ADRD populations.

Contact: [Honoluluhan717@gmail.com](mailto:Honoluluhan717@gmail.com)
