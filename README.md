# Predicting E. coli Contamination in River Water

My capstone project is using water-quality monitoring data from the UN GEMStat database. The goal is to build a baseline classification model that predicts whether a river water sample is contaminated with E. coli.

* Author: Rufaro Chirairo
* Notebook: https://github.com/RufaroChirairo/Capstone_Module20/blob/main/Capstone_Rough.ipynb

---

## Problem Statement

E. coli testing is slow, expensive, and requires lab work. This project asks: can routine physical and chemical water-quality measurements identify samples likely to be contaminated with E. coli?

Samples are labeled as contaminated (`1`) when E. coli is ≥ 100 per 100 mL and not contaminated (`0`) when it is below 100, based on the threshold described in the notebook as a high-risk level per the World Health Organization.

## Data

- Source: [GEMStat](https://gemstat.org/) (Global Water Quality Database), downloaded in five batches due to GEMStat download size limits.
- Raw size: 1,222,396 rows across five CSV files (semicolon-delimited, long format: one row per measurement).
- After cleaning: 1,135,919 rows, reshaped to 46,467 sampling events, each with E. coli measured.
- Features (20): Depth, BOD, COD, EC, FECALCOLI, NH3N, NO2N, NO3N, O2-Dis, TDS, TEMP, TKN, TN, TOC, TON, TP, TRP, TSS, TURB, pH.
- Target: ECOLI, converted to a binary label (see above). Class balance: 61.7% contaminated, 38.3% not contaminated.


## Methodology

1. Combine the five data files into one dataframe.
2. Filter out rows with `Poor` or `Suspect` data quality ratings, and drop the 137 turbidity measurements reported in JCU rather than NTU (0.27% of turbidity rows).
3. Reshape (pivot) so each row is one sampling event (station, date, time, depth) with one column per parameter. Repeat measurements within an event are averaged.
4. Keep only events with an E. coli measurement**, then drop identifier columns (index, station number, date, time).
5. Explore: mean vs. median comparisons, histograms, the E. coli / fecal coliform relationship, and the largest E. coli values (kept, since extreme values are not necessarily errors).
6. The model used a 75/25 train/test split. Preprocessing happens inside a Pipeline (median missing value imputation, then `RobustScaler`) so nothing from the test set leaks into training.

### Models compared

| Model | Description |
|---|---|
| Dummy baseline | Always predicts the majority class (contaminated) |
| Logistic regression with FECALCOLI | All 20 features |
| Logistic regression without FECALCOLI | 19 features. Fecal coliforms are a close proxy for E. coli, so this tests whether other parameters carry signal on their own |

## Results

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Dummy (majority class) | 0.6171 | 0.6171 | 1.0000 | 0.7632 | 0.5000 |
| LogReg + FECALCOLI | 0.7795 | 0.8325 | 0.8047 | 0.8184 | 0.8612 |
| LogReg − FECALCOLI | 0.7078 | 0.6989 | 0.9248 | 0.7962 | 0.7527 |

### Key takeaways

- Both logistic regression models beat the majority-class baseline.
- FECALCOLI matters a lot. Removing it lowers accuracy by about 7 percentage points and ROC-AUC from 0.861 to 0.753.
- Without FECALCOLI, the model still has real value (ROC-AUC 0.75), suggesting routine water chemistry helps identify contamination.
- In the no-FECALCOLI model, O2-Dis and pH have negative coefficients (higher values associated with lower predicted odds of contamination), while TP, TDS, and TN have positive coefficients.

## Limitations

- FECALCOLI is a near-proxy for the target (E. coli is a subset of fecal coliforms), so the with-FECALCOLI model answers a different question than the without-FECALCOLI model.
- Coefficients are not directly comparable across features because of differing scales. Depth is nearly constant (almost all samples at 0.3 m), so its large coefficient reflects scaling, not importance.
- This is a baseline only, with no hyperparameter tuning or more complex models.

## Future Work

- Perform Gradient Descent 3 fold cross-validation
- Nonlinear models (e.g., random forest, gradient boosting) and tuning.
