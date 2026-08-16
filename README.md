# Predicting Electoral Abstention in Spanish Municipalities

End-to-end data mining project that predicts **voter abstention** across the **8,117 municipalities of Spain**, using socio-demographic and economic census data (36 variables). Two prediction targets are studied:

- **`AbstentionPtge`** (continuous) — abstention rate per municipality, for linear regression.
- **`AbstencionAlta`** (binary) — whether a municipality has _high_ abstention, for logistic regression.

The full analysis is documented in three Jupyter notebooks (narrative in Spanish) plus a final written report in PDF.

## Results at a glance

The winning logistic regression model — chosen by the **principle of parsimony** among four candidates with statistically indistinguishable performance — achieves:

| Metric    | Train | Test      |
| --------- | ----- | --------- |
| ROC AUC   | 0.841 | **0.839** |
| Pseudo R² | 0.294 | 0.284     |

| Cut-off                 | Accuracy | Sensitivity | Specificity |
| ----------------------- | -------- | ----------- | ----------- |
| 0.50 (accuracy-optimal) | 0.799    | 0.555       | 0.913       |
| 0.27 (Youden-optimal)   | 0.764    | 0.747       | 0.771       |

Candidate models were compared with **20× repeated cross-validation** (mean AUC 0.833 ± 0.011 across all four), so the winner is not a lucky split. Train/test metrics are nearly identical — no meaningful overfitting.

A key substantive finding: **where you vote matters most**. Geographic variables (autonomous community, province) show the strongest association with abstention (Cramér's V), ahead of any single demographic or economic factor.

## Project pipeline

The notebooks are meant to be read (and run) in order — each stage feeds the next through a serialized pickle:

```
data/DatosEleccionesEspaña.xlsx
        │
        ▼
1. Depuración_Adolfo_Viguera.ipynb        ← data cleaning & preparation
        │   • type auditing (numeric vars that are really categorical)
        │   • error correction: impossible values (negative percentages,
        │     percentages > 100, sentinel 99999 codes), '?' categories,
        │     duplicated municipality names disambiguated into a unique ID
        │   • outlier analysis & treatment, missing-value imputation
        ▼
data_clean/datosEleccionesDep.pickle
        │
        ├─▶ 2. Relaciones_Adolfo_Viguera.ipynb   ← relationship analysis
        │       • correlation structure between inputs (collinearity:
        │         Population / TotalCensus / firm counts move together)
        │       • Cramér's V of every predictor vs. both targets
        │       • automatic variable transformations (Transf_Auto)
        │
        └─▶ 3. Regresión_logística_Adolfo_Viguera.ipynb   ← modelling
                • train/test split
                • classical selection: forward / backward / stepwise (AIC & BIC)
                • random-subset ("aleatoria") variable selection
                • 20× repeated CV comparison of the finalists
                • optimal cut-off search (Youden index vs. accuracy)
                • final evaluation: ROC curves, confusion-matrix metrics, pseudo R²
```

**`Tarea_minería_Adolfo_Viguera_Varea.pdf`** contains the complete written report with all the reasoning behind every decision.

## Repository structure

```
├── data/
│   └── DatosEleccionesEspaña.xlsx        # raw dataset (8,117 municipalities × 36 vars)
├── data_clean/
│   ├── datosEleccionesDep.pickle         # cleaned dataset (output of notebook 1)
│   └── datosEleccionesDep2.pickle        # alternative cleaned version
├── functions/
│   ├── FuncionesMineria.py               # data mining toolkit used by the notebooks
│   │                                     #   (Cramér's V, outlier/missing handling,
│   │                                     #    stepwise GLM selection, CV, ROC utils…)
│   └── FuncionesMineria2.py              # PCA plotting helpers (not used by the notebooks)
├── Depuración_Adolfo_Viguera.ipynb       # 1 — cleaning
├── Relaciones_Adolfo_Viguera.ipynb       # 2 — relationships
├── Regresión_logística_Adolfo_Viguera.ipynb  # 3 — modelling
├── Tarea_minería_Adolfo_Viguera_Varea.pdf    # final report
└── requirements.txt
```

## Getting started

Requires **Python 3.10+**.

```bash
git clone https://github.com/adolfoviguera/predicting_spanish_elections.git
cd predicting_spanish_elections

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter notebook
```

Run the notebooks **from the repository root** (paths are relative to it), in numbered order.

**Shortcut:** the cleaned pickle is already committed in `data_clean/`, so you can jump straight into notebook 2 or 3 without re-running the cleaning stage.

> ⚠️ Notebook 3 performs exhaustive stepwise and random-subset variable selection over ~70+ parameters — it can take a long time to run end to end. All cell outputs are saved in the notebooks, so you can read every result without executing anything.

## Notes

- Notebook narrative and code comments are written in **Spanish** (academic coursework for a data mining module).
- All modelling metrics quoted above come from the stored notebook outputs — nothing is cherry-picked from intermediate runs.

## License

[MIT](LICENSE)
