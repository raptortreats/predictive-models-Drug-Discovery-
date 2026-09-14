# Solubility prediction for drug discovery

XGBoost models and exploratory visualizations for **aqueous solubility**, a core ADMET property in small-molecule drug discovery.

This is a **2022 portfolio project** (October 2022). It is not a production QSAR pipeline and was not re-trained in this hygiene pass. Notebooks are kept as the original analysis, with paths cleaned up so the repo is easy to clone and run.

## What is in this repo

| Notebook | What it does |
| --- | --- |
| [`Solubility Prediction/xgboost_classification.ipynb`](Solubility%20Prediction/xgboost_classification.ipynb) | Binary solubility classification (low vs. not-low) from Morgan fingerprints + XGBoost |
| [`Solubility Prediction/xgboost_regression.ipynb`](Solubility%20Prediction/xgboost_regression.ipynb) | Continuous log-solubility regression from RDKit descriptors + XGBoost |
| [`Visualization/molecule_visualization.ipynb`](Visualization/molecule_visualization.ipynb) | Descriptor vs. solubility scatter plots, ring-count distributions, and solubility histogram |

**Stack:** Python, pandas, NumPy, scikit-learn, XGBoost, RDKit, matplotlib, seaborn, Jupyter, openpyxl.

## Methods (original notebooks)

Both modeling notebooks start from SMILES in the curated solubility table, compute extra RDKit descriptors, and filter extreme molecules (different cutoffs in each notebook).

**Classification.** Solubility is binned (low / medium / high) using µg/mL-style thresholds, then collapsed to a binary label (low vs. other). Molecules are encoded with Morgan fingerprints (`SimilarityMaps.GetMorganFingerprint` → 2048-d vectors). An `XGBClassifier` is trained with an 80/20 train/test split. An unlabeled external SMILES set is scored after dropping one invalid SMILES.

**Regression.** After dropping very large molecules (`MolWt > 700`), an `XGBRegressor` (`n_estimators=10`) is trained on nine descriptors (MolWt, MolLogP, HeavyAtomCount, NumHAcceptors, NumRotatableBonds, NumAromaticRings, NumAliphaticCarbocycles, NumAromaticCarbocycles, FractionCSP3) with a 70/30 split. The notebook also includes t-SNE, Tanimoto distance histograms, and a descriptor covariance heatmap.

**Visualization.** Pairwise descriptor plots (MolWt vs TPSA / LogP, hue = solubility), ring-count bar/pie charts, and the solubility distribution.

## Results recorded in the original 2022 run

These numbers are **copied from the executed notebook outputs before they were cleared**, not from a new training run. Re-running may differ (no fixed `train_test_split` random state on some splits; package versions differ).

**Classification** (5,559 molecules after filters; 1,112 test rows):

- Train accuracy: **0.971**
- Test accuracy: **0.893**
- Test report: low-solubility class F1 **0.64**; not-low F1 **0.94**; overall accuracy **0.89**
- Cohen’s kappa: **0.58** (moderate agreement)

**Regression** (9,717 molecules after `MolWt` filter):

- Train R²: **0.805**
- Test R²: **0.710**
- Test RMSE: **1.26** (MSE **1.59**) on the solubility target as stored in the CSV

Treat these as a student-project baseline, not a literature claim.

## Datasets

All tables live under [`data/`](data/).

| File | Role |
| --- | --- |
| [`data/curated-solubility-dataset.csv`](data/curated-solubility-dataset.csv) | **9,982** compounds: identifiers, SMILES, aqueous solubility, and precomputed RDKit descriptors (`MolWt`, `MolLogP`, `TPSA`, …). Column names match the public **AqSolDB**-style curated set (including the original `Ocurrences` spelling). |
| [`data/External_set.xlsx`](data/External_set.xlsx) | **188** unlabeled compounds (`cpd_ID`, `SMILES`) used only in the classification notebook for out-of-sample predictions. One row (index 54, `SP-93`) has invalid SMILES (`[MeO]`) and is dropped in the notebook. |

A previous `Visulaization/sol.csv` was a **byte-identical duplicate** of the curated CSV and was removed.

## How to run

Python **3.10–3.12**. `requirements.txt` pins a 2024-era stack that still installs; the original notebooks ran on Anaconda/Windows with **xgboost 1.6.2** and **numpy 1.21.5**. RDKit is simplest from conda-forge if the pip wheel fails.

```bash
git clone https://github.com/raptortreats/predictive-models-Drug-Discovery-.git
cd predictive-models-Drug-Discovery-

python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter notebook
```

Then open:

1. `Visualization/molecule_visualization.ipynb`
2. `Solubility Prediction/xgboost_classification.ipynb`
3. `Solubility Prediction/xgboost_regression.ipynb`

Run **all cells** from the top. Paths are relative to each notebook directory (`../data/...`).

Notebook **outputs were cleared** so GitHub diffs stay small. Plots and scores reappear after you execute the notebooks. The classification notebook still contains a leftover `pip install xgboost` cell; skip it if you already installed `requirements.txt`.

## Repository layout

```text
data/
  curated-solubility-dataset.csv
  External_set.xlsx
Solubility Prediction/
  xgboost_classification.ipynb
  xgboost_regression.ipynb
Visualization/
  molecule_visualization.ipynb
requirements.txt
```

## Limitations

- Default XGBoost hyperparameters; no nested CV or prospective temporal split.
- Classification labels are imbalanced (few “low” examples), which shows up in the minority-class F1.
- Fingerprint / descriptor code is 2022-era RDKit (`GetMorganFingerprint` APIs have since been renamed in newer docs).
- Pickle / SHAP cells in the modeling notebooks are leftover `raw` cells and were not part of the saved run.

## License / reuse

Code and notebooks are provided as a public portfolio snapshot. Check AqSolDB (and any external SMILES source) for dataset terms before redistributing the CSV commercially.
