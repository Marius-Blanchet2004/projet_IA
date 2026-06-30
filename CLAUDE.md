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

When `RUN_OPTIMIZATION = True`, sections 8b and 10 automatically write their results to `optimal_params.json` and rebuild `models` in memory. Cell 5 reads this file on the next run, so optimized parameters persist without re-running the optimization.

## optimal_params.json — paramètre persistence

`optimal_params.json` (gitignored) stores the latest optimal hyperparameters found by GridSearchCV:

```json
{
  "SVM Linéaire":          {"C": 0.001},
  "SVM Linéaire (bal.)":   {"C": 0.001},
  "SVM RBF":               {"C": 1, "gamma": "scale"},
  "SVM RBF (bal.)":        {"C": 0.001, "gamma": 0.001},
  "SVM Poly deg=2 (bal.)": {"C": 0.001},
  "SVM Poly deg=3 (bal.)": {"C": 0.001},
  "NuSVC RBF (bal.)":      {"nu": 0.4},
  "Random Forest":         {"n_estimators": 500}
}
```

Cell 5 (`72f549c1`) loads this file if present, falls back to hardcoded `_DEFAULT` if absent. Sections 8b and 10 update and overwrite it after each optimization run.

## Data structure — bilan SVM.xlsx

The Excel file has a non-standard layout: **BON PRODUIT** batches occupy columns A–E and **MAUVAIS PRODUIT** batches occupy columns G–J, side by side in the same rows (row index 4 to 25, row 26 = averages). There is no single tidy header row. The notebook reads it with `header=None` and reconstructs a flat DataFrame via `extract_group()`.

- 34 total batches (22 BON, 12 MAUVAIS)
- Some entries have `N/D` (no temperature reading) — imputed with group mean via `groupby().transform()`, not dropped
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
| `72f549c1` | 5 | Load `OPT` from `optimal_params.json` (fallback to `_DEFAULT`), build 8 models |
| `8ca47255` | 6 | LOOCV via `cross_val_score` + `LeaveOneOut`, `n_jobs=-1` |
| `d37a372c` | 7 | Styled comparison table (best model highlighted green) |
| `2cc5e200` | 8 | GridSearchCV on SVM RBF (bal.) over C × gamma — skipped if `RUN_OPTIMIZATION=False` |
| `86107272` | 8b | Full SVM optimization (all C, nu values) — saves to `optimal_params.json`, rebuilds `models` — skipped if `RUN_OPTIMIZATION=False` |
| `caa41654` | 8c | 4 interactive Plotly figures: SVM RBF (orange MAUVAIS bubble), SVM Linéaire (blue boundary), SVM Poly deg=3 (blue boundary), Random Forest (3 nested green surfaces at 50/70/90%) |
| `e56314ef` | 9 | Random Forest feature importances → `feature_importance_RF.png` |
| `0aca1151` | 9b | Plot first 3 RF trees at max_depth=3 (trained on `X_raw` for readable thresholds in min/°C/bar) |
| `50bc1c04` | 10 | n_estimators optimization for RF — saves best n to `optimal_params.json`, rebuilds `rf_final` — skipped if `RUN_OPTIMIZATION=False` |
| `70927b44` | 11 | Markdown conclusions |
| `ebf6d1d4` | 12 | Markdown: section 12 description (3 threshold lines: black=50%, blue=70%, gold=90%) |
| `02f19a4d` | 12 | 2D heatmaps of P(BON) for each pair of variables × 2 models, with 3 contour lines (50/70/90%) → `zones_validite.png` |
| `5300382f` | 12b | Markdown: section 12b description |
| `91f6e1cc` | 12b | RF-only 3D Plotly isosurface at P(BON) = 98% |
| `c11bc43e` | 12b | Connected components at 98%: top-4 volumes with centroid + extent (scipy.ndimage.label) |
| `6c3e4b44` | — | Parameter ranges at ≥90% confidence for both Random Forest and SVM RBF (printed table) |

`StandardScaler` is fit on the full dataset (no separate test set) because LOOCV is the sole evaluation strategy. This is intentional for small industrial datasets.

## Models tested

`class_weight='balanced'` is applied to all SVM variants to compensate for the 22/12 class imbalance.
Parameters below are the optimized values found by GridSearchCV (section 8b) on the 34-point dataset.

| Modèle | Accuracy LOOCV | Paramètres optimisés |
|---|---|---|
| SVM Linéaire (bal.) | **97.1%** | C=0.001 |
| SVM RBF (bal.) | **97.1%** | C=0.001, gamma=0.001 |
| SVM Poly deg=2 (bal.) | 100.0%* | C=0.001 |
| SVM Poly deg=3 (bal.) | 100.0%* | C=0.001 |
| Random Forest | 82.4% | n_estimators=500 |
| SVM RBF | 73.5% | C=1, gamma='scale' |
| NuSVC RBF (bal.) | 73.5% | nu=0.4 |
| SVM Linéaire | 64.7% | C=0.001 (dégénéré — prédit tout BON) |

*SVM Poly 100% LOOCV is an artifact: polynomial features expand 3D space to ~20D, making 34 points linearly separable by chance. K-Fold (section `analyse_kfold.ipynb`) reveals the true accuracy ~52% (below majority baseline).

**Note on degenerate models:** C=0.001 (chosen by GridSearchCV for linear/poly) produces a model that predicts all BON, achieving majority-class baseline accuracy — not real learning. For the 3D decision boundary visualization (section 8c), C=100 is used instead so a visible boundary exists.

Most discriminating variable: **Temp. (°C)** (Gini importance ~51%), then Pression (~27%), Durée (~22%).

## Section 8c — Decision zones (4 figures)

1. **SVM RBF (bal.)**: orange isosurface (caps=False) = MAUVAIS zone to avoid. BON = outside the bubble.
2. **SVM Linéaire (bal.)**: blue boundary surface (C=100 to avoid degenerate model)
3. **SVM Poly deg=3 (bal.)**: blue boundary surface (C=100)
4. **Random Forest**: 3 nested green isosurfaces at P(BON) = 50%, 70%, 90% via `predict_proba`

## Section 12 — Validity zones

- **Cell `02f19a4d`**: 2×3 heatmap grid (2 models × 3 variable pairs). Each heatmap averages P(BON) over the 3rd variable (12 samples). Three contour lines: black=50%, blue=70%, gold=90%.
- **Cell `91f6e1cc`**: RF-only 3D Plotly isosurface at exactly P(BON) = 98%.
- **Cell `c11bc43e`**: Connected-component analysis (scipy.ndimage.label, 26-connectivity) of the 98% mask on an 80³ grid. Prints top-4 volumes with centroid + extent.
- **Cell `6c3e4b44`**: Printed table of parameter ranges where P(BON) ≥ 90%, for both RF and SVM RBF.

`svm_zone` is a separate SVC instance trained with `probability=True` (Platt scaling) used only for section 12 heatmaps. `viz_models['SVM RBF (bal.)']` in section 8c uses `decision_function` directly (no probability).

## Tree visualization (section 9b)

`rf_viz` is trained on `X_raw` (unscaled) purely for visualization so thresholds display in real units (min, °C, bar). `rf_final` trained on `X` is used for LOOCV and feature importances. Random Forest does not require scaling so both give equivalent results.

## Second notebook — analyse_kfold.ipynb

Companion notebook testing RepeatedStratifiedKFold (K=4, 10 repeats = 40 scores per model) as an alternative to LOOCV. Uses `Pipeline([('scaler', StandardScaler()), ('clf', clf)])` to avoid data leakage. Produces:
- K-Fold accuracy table with meaningful std (not just √(p×(1-p)))
- Side-by-side comparison table LOOCV vs K-Fold with Δ column
- Bar chart with error bars → `comparaison_kfold_loocv.png`

K=4 chosen because 12 MAUVAIS / 4 folds = exactly 3 MAUVAIS per test fold (minimum for stability).

## Dependencies

Already installed in `.venv`: `numpy`, `pandas`, `scipy`, `matplotlib`, `scikit-learn`, `openpyxl`, `jupyterlab`, `plotly`.

Install if missing:
```bash
& ".venv\Scripts\python.exe" -m pip install numpy pandas scipy matplotlib scikit-learn openpyxl jupyterlab plotly
```
