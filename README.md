# Projet IA — Classification qualité SVM pour le broyeur B430

Modèle de classification supervisée (SVM + Random Forest) pour prédire si un lot de production est **BON** ou **MAUVAIS** à partir de trois paramètres de procédé du broyeur B430.

## Paramètres d'entrée

| Paramètre | Unité |
|---|---|
| Durée | min |
| Température | °C |
| Pression | bar |

## Résultats — Précision LOOCV

| Modèle | Précision |
|---|---|
| **Random Forest** | **85.7%** |
| SVM RBF (équilibré) | 82.1% |
| SVM RBF | 82.1% |
| NuSVC RBF (équilibré) | 78.6% |
| SVM Linéaire / Poly | 71.4% |

Évaluation par **Leave-One-Out Cross-Validation** (LOOCV), adaptée aux petits datasets industriels (28 lots : 20 BON, 8 MAUVAIS).

La variable la plus discriminante est la **Température (°C)** (~51% d'importance Gini), suivie de la Pression (~27%) et de la Durée (~22%).

## Contenu du dépôt

```
├── analyse_SVM.ipynb          # Notebook principal (SVM + RF, visualisations 3D)
├── analyse_kfold.ipynb        # Notebook comparatif K-Fold vs LOOCV
├── bilan SVM.xlsx             # Données brutes (28 lots)
├── presentation_B430.html     # Présentation interactive (Plotly embarqué)
├── Methodes_classification_B430.pdf   # Rapport méthodologique
├── validation_modeles_B430.pdf        # Rapport de validation
└── *.png                      # Figures exportées
```

## Lancer les notebooks

```bash
# Installer les dépendances dans un environnement virtuel
python -m venv .venv
.venv\Scripts\activate
pip install numpy pandas scipy matplotlib scikit-learn openpyxl jupyterlab plotly

# Lancer JupyterLab
py -m jupyterlab
```

> Le flag `RUN_OPTIMIZATION = False` (cellule `4b1ffbb1`) permet de sauter les longues cellules GridSearchCV pour un **Run All** en quelques secondes. Passer à `True` pour relancer l'optimisation complète.

## Structure des données

Le fichier `bilan SVM.xlsx` a une disposition non standard : les lots **BON PRODUIT** (colonnes A–E) et **MAUVAIS PRODUIT** (colonnes G–J) sont côte à côte dans les mêmes lignes. Le notebook reconstruit un DataFrame plat via `extract_group()`. Deux entrées avec température `N/D` sont imputées par la moyenne de groupe.

## Visualisations produites

- **Scatter 3D + projections orthogonales** (`visualisation_3D_projections.png`)
- **4 vues 3D** (`vues_multiples_3D.png`)
- **Zones de décision interactives** (Plotly) : bulle MAUVAIS SVM RBF, frontières SVM Linéaire et Poly, isosurfaces RF à 50/70/90%
- **Heatmaps de validité 2D** par paire de variables (`zones_validite.png`)
- **Importances des variables RF** (`feature_importance_RF.png`)
- **Comparaison K-Fold vs LOOCV** (`comparaison_kfold_loocv.png`)

## Dépendances

`numpy` · `pandas` · `scipy` · `matplotlib` · `scikit-learn` · `openpyxl` · `jupyterlab` · `plotly`
