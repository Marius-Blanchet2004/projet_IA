# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

SVM-based quality classification for the B430 grinder. Predicts whether a manufacturing batch is "BON" (good) or "MAUVAIS" (bad) from three process parameters: Durée (min), Temp. (°C), Pression (bar).

## Running the notebook

```bash
# Launch JupyterLab (browser opens automatically)
py -m jupyterlab

# Execute all cells headlessly and save outputs in-place
py -m jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=180 analyse_SVM.ipynb
```

Python is invoked with `py` (not `python` or `python3`) on this machine.

## RUN_OPTIMIZATION flag

Cell `4b1ffbb1` defines `RUN_OPTIMIZATION = False`. Set to `True` to re-run the three long optimization cells (sections 8, 8b, 10). When `False`, those cells are skipped silently so **Run All** completes in seconds.

## Data structure — bilan SVM.xlsx

The Excel file has a non-standard layout: **BON PRODUIT** batches occupy columns A–E and **MAUVAIS PRODUIT** batches occupy columns G–J, side by side in the same rows (row index 4 to 23, row 24 = averages). There is no single tidy header row. The notebook reads it with `header=None` and reconstructs a flat DataFrame via `extract_group()`.

- 28 total batches (20 BON, 8 MAUVAIS)
- 2 entries have `N/D` (no temperature reading) — imputed with group mean via `groupby().transform()`, not dropped
- Columns used: `Durée (min)`, `Temp. (°C)`, `Pression (bar)`, `label` (1=BON, 0=MAUVAIS)

## Notebook architecture (analyse_SVM.ipynb)

| Cell ID | Section | Role |
|---|---|---|
| `9e50ab8a` | — | Bootstrap `openpyxl` install |
| `4b1ffbb1` | — | `RUN_OPTIMIZATION = False` flag |
| `2c06037c` | — | Imports (`SVC`, `NuSVC`, `GridSearchCV`, `RandomForestClassifier`, `plotly`) |
| `4cf957ad` | 1 | Load Excel, parse dual-column layout, impute N/D, build flat DataFrame |
| `e8466e98` | 2 | Main figure: 3D scatter + 3 orthographic 2D projections → `visualisation_3D_projections.png` |
| `12a3b22c` | 3 | 4-angle 3D views → `vues_multiples_3D.png` |
| `615bbcbe` | 3b | Interactive 3D scatter (Plotly) |
| `e1a1220c` | 4 | `StandardScaler` on `X_raw` → `X` |
| `72f549c1` | 5 | Define 8 models with optimized parameters |
| `8ca47255` | 6 | LOOCV via `cross_val_score` + `LeaveOneOut`, `n_jobs=-1` |
| `d37a372c` | 7 | Styled comparison table (best model highlighted green) |
| `2cc5e200` | 8 | GridSearchCV on SVM RBF (bal.) over C × gamma — skipped if `RUN_OPTIMIZATION=False` |
| `86107272` | 8b | Full SVM optimization (all C, nu values) — skipped if `RUN_OPTIMIZATION=False` |
| `caa41654` | 8c | 4 interactive Plotly figures: SVM RBF (orange MAUVAIS bubble), SVM Linéaire (blue boundary), SVM Poly deg=3 (blue boundary), Random Forest (3 nested green surfaces at 50/70/90%) |
| `e56314ef` | 9 | Random Forest feature importances → `feature_importance_RF.png` |
| `0aca1151` | 9b | Plot first 3 RF trees at max_depth=3 (trained on `X_raw` for readable thresholds in min/°C/bar) |
| `206045c0` | 10 | n_estimators optimization for RF — skipped if `RUN_OPTIMIZATION=False` |
| `70927b44` | 11 | Markdown conclusions |
| `ebf6d1d4` | 12 | Markdown: section 12 description (3 threshold lines: black=50%, blue=70%, gold=90%) |
| `02f19a4d` | 12 | 2D heatmaps of P(BON) for each pair of variables × 2 models, with 3 contour lines (50/70/90%) → `zones_validite.png` |
| `5300382f` | 12b | Markdown: section 12b description |
| `91f6e1cc` | 12b | 3D interactive validity zones (Plotly Isosurface) at 50/70/90% for SVM RBF and Random Forest |
| `6c3e4b44` | — | Parameter ranges at ≥90% confidence for both Random Forest and SVM RBF (printed table) |

`StandardScaler` is fit on the full dataset (no separate test set) because LOOCV is the sole evaluation strategy. This is intentional for small industrial datasets.

## Models tested

`class_weight='balanced'` is applied to all SVM variants to compensate for the 20/8 class imbalance.
Parameters below are the optimized values found by GridSearchCV (section 8b).

| Modèle | Accuracy LOOCV | Paramètres optimisés |
|---|---|---|
| Random Forest | **85.7%** | n_estimators=100 |
| SVM RBF (bal.) | 82.1% | C=1, gamma=1.0 |
| SVM RBF | 82.1% | C=10, gamma=0.1 |
| NuSVC RBF (bal.) | 78.6% | nu=0.2 |
| SVM Linéaire | 71.4% | C=0.001 (dégénéré — prédit tout BON) |
| SVM Linéaire (bal.) | 71.4% | C=0.001 (dégénéré — prédit tout BON) |
| SVM Poly deg=3 (bal.) | 71.4% | C=0.001 (dégénéré — prédit tout BON) |
| SVM Poly deg=2 (bal.) | 71.4% | C=0.001 (dégénéré — prédit tout BON) |

**Note on degenerate models:** C=0.001 (chosen by GridSearchCV for linear/poly) produces a model that predicts all BON, achieving 71.4% accuracy by majority class baseline — not real learning. For the 3D decision boundary visualization (section 8c), C=100 is used instead so a visible boundary exists.

Most discriminating variable: **Temp. (°C)** (Gini importance ~51%), then Pression (~27%), Durée (~22%).

## Section 8c — Decision zones (4 figures)

1. **SVM RBF (bal.)**: orange isosurface (caps=False) = MAUVAIS zone to avoid. BON = outside the bubble.
2. **SVM Linéaire (bal.)**: blue boundary surface (C=100 to avoid degenerate model)
3. **SVM Poly deg=3 (bal.)**: blue boundary surface (C=100)
4. **Random Forest**: 3 nested green isosurfaces at P(BON) = 50%, 70%, 90% via `predict_proba`

## Section 12 — Validity zones

- **Cell `02f19a4d`**: 2×3 heatmap grid (2 models × 3 variable pairs). Each heatmap averages P(BON) over the 3rd variable (12 samples). Three contour lines: black=50%, blue=70%, gold=90%.
- **Cell `91f6e1cc`**: 3D Plotly versions of the same for SVM RBF and RF, using `svm_zone` (`probability=True`) and `rf_final`.
- **Cell `6c3e4b44`**: Printed table of parameter ranges where P(BON) ≥ 90%, for both models.

`svm_zone` is a separate SVC instance trained with `probability=True` (Platt scaling) used only for sections 12/12b. `viz_models['SVM RBF (bal.)']` in section 8c uses `decision_function` directly (no probability).

## Tree visualization (section 9b)

`rf_viz` is trained on `X_raw` (unscaled) purely for visualization so thresholds display in real units (min, °C, bar). `rf_final` trained on `X` is used for LOOCV and feature importances. Random Forest does not require scaling so both give equivalent results.

## Second notebook — analyse_kfold.ipynb

Companion notebook testing RepeatedStratifiedKFold (K=4, 10 repeats = 40 scores per model) as an alternative to LOOCV. Produces:
- K-Fold accuracy table with meaningful std (not just √(p×(1-p)))
- Side-by-side comparison table LOOCV vs K-Fold with Δ column
- Bar chart with error bars → `comparaison_kfold_loocv.png`

K=4 chosen because 8 MAUVAIS / 4 folds = exactly 2 MAUVAIS per test fold (minimum for stability).

## Dependencies

Already installed in `.venv`: `numpy`, `pandas`, `scipy`, `matplotlib`, `scikit-learn`, `openpyxl`, `jupyterlab`, `plotly`.

Install if missing:
```bash
& ".venv\Scripts\python.exe" -m pip install numpy pandas scipy matplotlib scikit-learn openpyxl jupyterlab plotly
```
