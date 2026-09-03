# Alzheimer's Disease Stage Classification

Binary classification of Alzheimer's disease stage from regional amyloid burden,
using SUVR measurements across 148 brain regions.

Independent research project completed in 2023 under the mentorship of a research
scientist.

> **This notebook is not runnable as published.** The dataset is a restricted
> clinical cohort governed by a data use agreement and cannot be redistributed.
> All outputs shown are preserved from the original 2023 run.

---

## The problem

The source data labels each scan with one of five clinical stages along the
Alzheimer's continuum. Modelling five ordered stages directly on a cohort of this
size gives very thin support for the middle classes, so the problem was reframed as
a binary one: cognitively normal (CN) against Alzheimer's disease (AD), grouping
clinically similar stages together.

The classification pipeline was written to accept variable class definitions, so the
same code can be re-run for different groupings rather than being rewritten for each.

## The data

| | |
|---|---|
| Records | 2,753 scans |
| Features | 148 regional amyloid SUVR values (`Node 1` to `Node 148`) |
| Target | `DX`, clinical diagnosis stage |
| Subject key | `PTID` |

Demographic and genetic columns (age, sex, education, ethnicity, race, marital
status, APOE4 status) were dropped, restricting the model to imaging-derived
features alone. 11 records (0.4%) had a missing diagnosis label.

Records are **scans, not patients**. The same subject appears multiple times across
different scan dates, which drives the most important design decision below.

## Method

**Leakage prevention.** Because subjects are scanned repeatedly, a naive random split
would place scans from the same person on both sides of the train/test boundary,
letting the model recognise the individual rather than the pathology. All records
sharing a `PTID` were therefore kept within a single split.

**Class imbalance.** Handled with oversampling (SMOTE), applied inside the training
split only.

**Subject-level aggregation.** Predictions were made per scan and then aggregated to
a single prediction per subject by majority vote, which suppresses individual noisy
scans and better matches how a clinical judgement would actually be formed.

**Models.** Five configurations were compared with systematic hyperparameter tuning.

## Results

CN vs AD, held-out test set:

| Model | Train accuracy | Test accuracy |
|---|---|---|
| Logistic Regression (L1) | 0.930 | 0.830 |
| Logistic Regression (L2) | 0.850 | 0.840 |
| Logistic Regression (cross-validated) | 0.850 | 0.770 |
| Neural network | see note | 0.825 |
| **Random Forest (GridSearchCV)** | 0.957 | **0.871** |

> **Note on the neural network row:** the training accuracy recorded in the original
> run is `0.000`, which is a logging error rather than a result. The test accuracy of
> 0.825 is valid. The dataset is no longer available to re-run, so the erroneous value
> has been left in the notebook rather than edited, and is flagged here instead.

## What I would do differently

Three things stand out looking back at this work:

**Accuracy was the wrong headline metric.** On an imbalanced clinical problem where
the cost of a false negative is not the cost of a false positive, AUC or F1 would be
far more informative, and reporting accuracy after oversampling flatters the result.

**The random forest is overfitting.** It won on test accuracy, but at a 0.957/0.871
train/test gap. L2 logistic regression sat at 0.850/0.840, a roughly one-point gap,
and on a cohort this size that stability is arguably worth more than the 3-point
accuracy advantage. A larger held-out set or nested cross-validation would settle it.

**Dropping the demographic columns was a choice worth testing, not assuming.** Age
and APOE4 status carry real predictive signal for Alzheimer's. Excluding them isolates
the imaging contribution, which was the intent, but the trade-off was never quantified
against a model that includes them.

## Files

- `Alzheimer_Classification.ipynb` — full analysis, outputs preserved from the original run.
