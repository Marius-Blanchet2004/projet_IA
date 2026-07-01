# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Freelance mission for **Fiabila** (nail-polish manufacturer). SVM-based quality classification for the **B430 grinder** (broyeur). Predicts whether a manufacturing batch is "BON" (good) or "MAUVAIS" (bad) from three process parameters: `Durée (min)`, `Temp. (°C)`, `Pression (bar)`.

The deliverable chain ties together everything in the repo:

```
bilan SVM.xlsx  ──►  analyse_SVM.ipynb  ──►  *.png figures  ──►  presentation_B430.html
   (raw data)        analyse_kfold.ipynb      + data3d.js          (client-facing slide deck)
                     (modeling & viz)         (Plotly export)      + PDFs + preparation_negociation.md
```

- **Notebooks** train/evaluate the models and export the PNG figures and the Plotly data.
- **`presentation_B430.html`** is the client-facing deck; it embeds those figures and an interactive 3D scatter.
- **PDFs** (`Methodes_classification_B430.pdf`, `validation_modeles_B430.pdf`) are the written reports.
- **`preparation_negociation.md`** is Marius's private commercial prep for the Fiabila mission (pricing, leverage, technical caveats) — not part of the technical pipeline.

## Repository layout

| File | Role |
|---|---|
| `analyse_SVM.ipynb` | Main notebook — data load, 3D viz, 8 models, LOOCV, optimization, decision zones |
| `analyse_kfold.ipynb` | Companion — RepeatedStratifiedKFold vs LOOCV comparison |
| `bilan SVM.xlsx` | Raw data (dual-column layout, see below) |
| `optimal_params.json` | Persisted best hyperparameters from GridSearchCV (NOT gitignored — tracked/committed) |
| `presentation_B430.html` | Client slide deck; embeds figures + interactive Plotly 3D |
| `plotly.min.js` | Local Plotly runtime for the HTML deck |
| `data3d.js` | 3D scatter data exported for the HTML deck (loaded by `presentation_B430.html`) |
| `data_proba.js` | Probability-grid export — **currently orphaned**, not referenced by the HTML |
| `Methodes_classification_B430.pdf` | Methodology report |
| `validation_modeles_B430.pdf` | Validation report |
| `preparation_negociation.md` | Private negotiation prep for the Fiabila mission |
| `fiabila-logo-crop.png`, `fiabila-logo2.png`, `fiabila-banner.png`, `fiabila-mains-creme.png` | Fiabila branding used in the HTML deck |
| `*.png` (figures) | Exported by the notebooks — see figure list below |
| `README.md` | Public project summary — **stale** (still cites 28 lots / old accuracies); update or ignore |

## Running the notebooks

```bash
# Launch JupyterLab (browser opens automatically)
py -m jupyterlab

# Execute all cells headlessly and save outputs in-place
py -m jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=180 analyse_SVM.ipynb
```

Python is invoked with `py` (not `python` or `python3`) on this machine. The virtualenv interpreter is `.venv\Scripts\python.exe`.

## RUN_OPTIMIZATION flag

Cell `4b1ffbb1` defines `RUN_OPTIMIZATION = False` — the three long optimization cells (sections 8, 8b, 10) are skipped silently so **Run All** completes in seconds. Set to `True` only to re-optimize; it then rewrites `optimal_params.json`. After a `True` run, set it back to `False`.

When `RUN_OPTIMIZATION = True`, sections 8b and 10 automatically write their results to `optimal_params.json` and rebuild `models` / `rf_final` in memory. Cell 5 reads that file on the next run, so optimized parameters persist without re-running the optimization.

## optimal_params.json — parameter persistence

`optimal_params.json` stores the latest optimal hyperparameters found by GridSearchCV. It is committed (not gitignored). Current contents:

```json
{
  "SVM Linéaire":          {"C": 0.001},
  "SVM Linéaire (bal.)":   {"C": 10},
  "SVM RBF":               {"C": 1, "gamma": "scale"},
  "SVM RBF (bal.)":        {"C": 10, "gamma": 0.1},
  "SVM Poly deg=2 (bal.)": {"C": 100},
  "SVM Poly deg=3 (bal.)": {"C": 100},
  "NuSVC RBF (bal.)":      {"nu": 0.3},
  "Random Forest":         {"n_estimators": 50}
}
```

These values were regenerated on the clean 33-lot dataset (after the loader fix below). `SVM Linéaire` (non-balanced) legitimately stays at C=0.001 ≈ majority baseline — a linear kernel simply can't beat baseline on this data.

Cell 5 (`72f549c1`) loads this file if present, falls back to hardcoded `_DEFAULT` if absent. Sections 8b and 10 update and overwrite it after each optimization run.

## Data structure — bilan SVM.xlsx

The Excel file has a non-standard layout: **BON PRODUIT** batches occupy columns A–E and **MAUVAIS PRODUIT** batches occupy columns G–J (column F is an empty separator), side by side in the same rows. There is no single tidy header row. The notebook reads it with `header=None` and reconstructs a flat DataFrame via `extract_group()`.

- Row 0 = title, rows 2–3 = headers, **rows 4–23 = batch data**, row 24 = averages (`MOYENNE`), row 25 = footnote.
- **Current dataset: 33 batches — 20 BON, 13 MAUVAIS** (per the Excel footnote: "20 lots bon produit … et 13 lots mauvais produit"). The 6 yellow-highlighted BON are "produits excellents conservés".
- Some entries have `N/D` (no temperature reading) — imputed with group mean via `groupby().transform()`, not dropped.
- Columns used: `Durée (min)`, `Temp. (°C)`, `Pression (bar)`, `label` (1=BON, 0=MAUVAIS).

> **Loader fix (important).** `extract_group()` iterates `range(DATA_START, DATA_END)` with `DATA_END = 26`. When the Excel was shortened, the `MOYENNE` row moved up from row 26 to row 24, so it fell *inside* the read range and was ingested as a fake extra batch in **both** classes (giving a bogus 35 rows = 21/14 with two "perfectly average" points near each class centroid — this distorted the SVM boundaries and produced the earlier degenerate C=0.001 / fake 100% results). The loader now filters that row by its label (`str(lot).lower().startswith('moy')`), which is robust to future row shifts. If you re-shorten the sheet, this guard keeps the count correct — verify the printed "Bon / Mauvais" counts after any data edit.
>
> The README still cites an even older 28-lot version — treat `README.md` numbers as stale.

## Notebook architecture (analyse_SVM.ipynb)

| Cell ID | Section | Role |
|---|---|---|
| `eca35d93` | — | Title markdown |
| `9e50ab8a` | — | Bootstrap `openpyxl` install |
| `4b1ffbb1` | — | `RUN_OPTIMIZATION = False` flag |
| `2c06037c` | — | Imports (`SVC`, `NuSVC`, `GridSearchCV`, `RandomForestClassifier`, `plotly`) |
| `4cf957ad` | 1 | Load Excel, parse dual-column layout, **filter the MOYENNE row** (see loader fix), impute N/D, build flat DataFrame (prints counts — must read 20 BON / 13 MAUVAIS) |
| `e8466e98` | 2 | Main figure: 3D scatter + 3 orthographic 2D projections → `visualisation_3D_projections.png` |
| `12a3b22c` | 3 | 4-angle 3D views → `vues_multiples_3D.png` |
| `615bbcbe` | 3b | Interactive 3D scatter (Plotly) |
| `e1a1220c` | 4 | `StandardScaler` on `X_raw` → `X` |
| `72f549c1` | 5 | Load `OPT` from `optimal_params.json` (fallback to `_DEFAULT`), build 8 models |
| `8ca47255` | 6 | LOOCV via `cross_val_score` + `LeaveOneOut`, `n_jobs=-1` |
| `d37a372c` | 7 | Styled comparison table (best model highlighted green) |
| `2cc5e200` | 8 | GridSearchCV on SVM RBF (bal.) over C × gamma — skipped if `RUN_OPTIMIZATION=False` |
| `86107272` | 8b | Full SVM optimization (all C, nu values) — saves to `optimal_params.json`, rebuilds `models` — skipped if `RUN_OPTIMIZATION=False` |
| `caa41654` | 8c | 4 interactive Plotly figures (SVM RBF orange MAUVAIS bubble, SVM Linéaire boundary, SVM Poly deg=3 boundary, Random Forest 3 nested green surfaces) |
| `18b5770c` | 9 | Markdown header — **mislabeled "## 8. Importance des variables — Random Forest"** (should read 9) |
| `e56314ef` | 9 | Random Forest feature importances → `feature_importance_RF.png`. Note: `rf_final` hardcodes `n_estimators=500` here (not read from `optimal_params.json`) |
| `0aca1151` | 9b | Plot first 3 RF trees at max_depth=3 (trained on `X_raw` for readable thresholds in min/°C/bar) |
| `206045c0` | 10 | Markdown header — n_estimators optimization |
| `50bc1c04` | 10 | n_estimators optimization for RF — **picks the smallest n on the plateau** (within a 1-sample tolerance, not the noisy argmax), saves it to `optimal_params.json`, rebuilds `rf_final` — skipped if `RUN_OPTIMIZATION=False` |
| `70927b44` | 11 | Markdown conclusions |
| `ebf6d1d4` | 12 | Markdown: section 12 description (3 threshold lines: black=50%, blue=70%, gold=90%) |
| `02f19a4d` | 12 | 2D heatmaps of P(BON) for each pair of variables × 2 models, 3 contour lines (50/70/90%) → `zones_validite.png` |
| `5300382f` | 12b | Markdown: section 12b description |
| `91f6e1cc` | 12b | RF-only 3D Plotly isosurface at P(BON) = 98% |
| `c11bc43e` | 12b | Connected components at 98%: top-4 volumes with centroid + extent (`scipy.ndimage.label`) |
| `6c3e4b44` | — | **Currently empty** trailing cell (previously held the ≥90%-confidence parameter-range table) |

`StandardScaler` is fit on the full dataset (no separate test set) because LOOCV is the sole evaluation strategy. This is intentional for small industrial datasets.

## Models tested

`class_weight='balanced'` is applied to all SVM variants to compensate for the 20/13 class imbalance (majority baseline = 20/33 ≈ 60.6%). Parameters and accuracies below are from the last GridSearchCV run on the **clean 33-lot dataset** (section 8b + section 10), matching `optimal_params.json`.

| Modèle | Paramètres optimisés | LOOCV |
|---|---|---|
| **Random Forest** | **n_estimators=50** | **72.7%** (plateau 50→200; n=500 spikes to 75.8% = LOOCV noise) |
| SVM RBF (bal.) | C=10, gamma=0.1 | 69.7% |
| SVM RBF | C=1, gamma='scale' | 69.7% |
| NuSVC RBF (bal.) | nu=0.3 | 63.6% |
| SVM Linéaire / Poly deg=3 (bal.) | C=10 / C=100 | ≈ 60.6% (near baseline) |
| SVM Poly deg=2 (bal.) | C=100 | 51.5% |
| SVM Linéaire (non-bal.) | C=0.001 | 60.6% (degenerate — predicts all BON) |

**Only the non-balanced linear SVM is now degenerate** (C=0.001 = majority baseline; a linear kernel can't beat baseline here). The earlier "SVM RBF (bal.) = 100%" and the C=0.001 selections for the *balanced* models were **artifacts of the ingested MOYENNE row** — they disappeared once the loader was fixed. For the 3D decision-boundary visualization (section 8c), C=100 is used so a visible boundary always exists.

**Numbers are still fragile:** with 33 points, one flipped sample = ±3%, so the exact "best" hyperparameter is largely LOOCV noise (e.g. the lone n=500 → 75.8% spike). The RF selector deliberately takes the **smallest n on the plateau** (within a 1-sample tolerance) rather than the noisy argmax. The K-Fold companion (`analyse_kfold.ipynb`) gives a more honest generalization estimate with real error bars.

Most discriminating variable: **Temp. (°C)** (Gini importance ~51%), then Pression (~27%), Durée (~22%).

## Section 8c — Decision zones (4 figures)

1. **SVM RBF (bal.)**: orange isosurface (caps=False) = MAUVAIS zone to avoid. BON = outside the bubble.
2. **SVM Linéaire (bal.)**: blue boundary surface (C=100 to avoid the degenerate model).
3. **SVM Poly deg=3 (bal.)**: blue boundary surface (C=100).
4. **Random Forest**: 3 nested green isosurfaces at P(BON) = 50%, 70%, 90% via `predict_proba`.

`viz_models['SVM RBF (bal.)']` here uses `decision_function` directly (no probability).

## Section 12 — Validity zones

- **Cell `02f19a4d`**: 2×3 heatmap grid (2 models × 3 variable pairs). Each heatmap averages P(BON) over the 3rd variable. Three contour lines: black=50%, blue=70%, gold=90% → `zones_validite.png`.
- **Cell `91f6e1cc`**: RF-only 3D Plotly isosurface at exactly P(BON) = 98%.
- **Cell `c11bc43e`**: Connected-component analysis (`scipy.ndimage.label`, 26-connectivity) of the 98% mask on an 80³ grid. Prints top-4 volumes with centroid + extent.

`svm_zone` is a separate SVC instance trained with `probability=True` (Platt scaling), used only for section 12 heatmaps.

## Tree visualization (section 9b)

`rf_viz` is trained on `X_raw` (unscaled) purely for visualization so thresholds display in real units (min, °C, bar). `rf_final` trained on `X` is used for LOOCV and feature importances. Random Forest does not require scaling, so both give equivalent results.

## Second notebook — analyse_kfold.ipynb

Companion notebook testing `RepeatedStratifiedKFold` (K=4, 10 repeats = 40 scores per model) as an alternative to LOOCV. Uses `Pipeline([('scaler', StandardScaler()), ('clf', clf)])` to avoid data leakage. It reads the same `bilan SVM.xlsx` with its **own copy** of `extract_group()` — the MOYENNE-row filter must stay in sync with the main notebook's loader (both were fixed together). Produces:

- K-Fold accuracy table with meaningful std (not just √(p×(1-p)))
- Side-by-side comparison table LOOCV vs K-Fold with Δ column
- Bar chart with error bars → `comparaison_kfold_loocv.png`

K=4 chosen because 13 MAUVAIS / 4 folds ≈ 3 MAUVAIS per test fold (minimum for stability).

## Figures exported by the notebooks

`visualisation_3D_projections.png`, `vues_multiples_3D.png`, `feature_importance_RF.png`, `optimisation_n_arbres.png`, `zones_validite.png`, `comparaison_kfold_loocv.png`, `arbres_foret.png`. (`output.png` and the `Capture d'écran …png` files are ad-hoc screenshots, not pipeline outputs.)

## Client deck — presentation_B430.html

Standalone HTML slide deck for Fiabila. Loads `plotly.min.js` + `data3d.js` locally and calls `Plotly.newPlot('plot-scatter', …)` for an interactive 3D scatter; embeds `zones_validite.png` and the Fiabila branding PNGs. Open directly in a browser (no server needed). If the notebook regenerates `data3d.js`, the deck's interactive plot updates on reload. `data_proba.js` exists but is not currently wired into the HTML.

## Dependencies

Already installed in `.venv`: `numpy`, `pandas`, `scipy`, `matplotlib`, `scikit-learn`, `openpyxl`, `jupyterlab`, `plotly`.

Install if missing:
```bash
& ".venv\Scripts\python.exe" -m pip install numpy pandas scipy matplotlib scikit-learn openpyxl jupyterlab plotly
```
