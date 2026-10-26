# Predicting Organic Solar Material Properties with Machine Learning

Predicting the **stability** and **HOMO/LUMO energy levels** of organic photovoltaic (OPV) molecules from chemical structure alone,
so researchers can shortlist promising candidates before expensive synthesis and lab testing.

Built for the *Block 1B Machine Learning* course (Applied Data Science & AI, Breda University of Applied Sciences, 2025-26)
as part of a major-wide Kaggle competition.

## Results

| Task | Model | Metric | Competition rank |
|------|-------|--------|------------------|
| Stability classification (high / medium / low) | L1 logistic regression | F1 0.85 (Kaggle public leaderboard) | **11th of 76** |
| HOMO / LUMO regression | SVR ensembles + cluster-based sample weighting | MAE 0.08 eV (Kaggle) | **10th of 76** |

- Classification: 5-fold cross-validated F1 of 0.883 ± 0.019, with SMOTE applied inside each fold to avoid leakage.
- Regression: after the deadline, a rebuilt pipeline without percentile clipping scored MAE 0.072, which would have ranked 6th.
- Post-competition fix: in notebook 02, the distance-to-cluster-center part of the sample weights originally used the last model from the
  elbow loop (k=19) instead of the final k=3 model. The notebook now uses the correct model, and the re-run scores MAE 0.0800 on Kaggle.

Full write-up: [`reports/report.pdf`](reports/report.pdf)

## The problem

Organic solar cells are flexible, light and cheap to produce, but the space of possible molecules is huge and every candidate
has to be synthesized and tested. A model that predicts key properties from a SMILES string can cut that list down.

- **Stability** decides whether a device keeps working under light and heat over time.
- **HOMO** (electron donating level) influences the open-circuit voltage.
- **LUMO** (electron accepting level) affects charge separation.

## Approach

**Data**
- 350 molecules from the Harvard OPV dataset (stability labels + a ~375,000-line raw file parsed with a custom parser)
  plus 232 molecules from a course SQL database, merged in DBeaver.
- Only partial orbital coverage: 466 molecules with HOMO, 454 with LUMO.

**Features**
- 217 RDKit molecular descriptors + bond and atom counts.
- About 40 chemistry-informed features (conjugation, heteroatom density, electron-withdrawing/donating groups, charge distribution).
- Cleaning: NaN/inf set to 0, an extreme RDKit descriptor (`Ipc`, values above 10^25) clipped, percentile clipping and constant-feature removal for classification.

**Classification** (`notebooks/01`, `03`)
- Compared logistic regression, random forest, gradient boosting, voting and stacking ensembles.
- Final model: logistic regression with L1 penalty, tuned with Optuna (100 trials, 5-fold CV), SMOTE on training data only.
- The chemistry-informed features gave the biggest gain (F1 from about 0.80 to 0.85); voting and stacking added nothing because
  logistic regression and random forest agreed on 98% of predictions.

**Regression** (`notebooks/02`, `04`)
- Separate models for HOMO and LUMO, since they respond to different chemistry.
- Support Vector Regression on standardized features: HOMO = RBF 60% + poly-2 20% + poly-3 20%, LUMO = RBF 70% + poly-2 30%. Hyperparameters tuned with Optuna.
- **Domain adaptation with clustering:** K-means on training + Kaggle molecules in scaled feature space; training molecules in clusters
  that contain many test molecules, and close to their cluster center, get higher sample weights.

## Repository structure

```
notebooks/
  01_classification_stability_training.ipynb     feature engineering, SMOTE, Optuna, model comparison
  02_regression_homo_lumo_training.ipynb         clustering weights, SVR tuning, ensembles
  03_classification_submission_pipeline.ipynb    inference on the Kaggle test set
  04_regression_submission_pipeline.ipynb        inference on the Kaggle test set
reports/
  report.pdf                                     course report (business understanding to conclusion)
data/       see data/README.md (data not included)
models/     trained artifacts are written here (git-ignored)
requirements.txt
```

## Running it

```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Add the data as described in `data/README.md`, then run the notebooks in order (01, 02, then 03, 04).
Notebooks 03 and 04 load the artifacts that 01 and 02 save (`algor_used\`, `tools_used\`); adjust those folder names if you move things.

## Limitations

- Very small dataset (roughly 300-470 molecules) compared with 240-275 features, so overfitting remains a risk.
- SMOTE creates synthetic molecules that may not be chemically valid.
- Cluster-based weighting tunes the regression to the competition's test distribution and may generalize worse to unrelated molecules.
- Predictions are for pre-screening, not a replacement for experimental measurement.

## Next steps

More data, graph neural networks on molecular graphs, and an active-learning loop that feeds the model's most uncertain
molecules back for measurement.

## Tech

Python, pandas, NumPy, scikit-learn, imbalanced-learn, RDKit, Optuna, SQL (DBeaver), Jupyter.
