# Forecasting European Housing Price Growth
### Machine Learning and Deep Learning — CBS Spring 2026

---

## Project Overview

This project forecasts **next-quarter residential house price growth** across EU-27 countries using a country-quarter panel dataset spanning 2005–2025. It combines macroeconomic, supply-side, financing, demographic, and institutional variables from ten data sources (Eurostat, ECB, MARPOR) into a unified modelling framework.

The analysis proceeds in two parallel tracks:

- **Pooled models** — trained on all 26 countries simultaneously
- **Cluster-stratified models** — trained separately within housing market regimes, identified through both data-driven and theory-based clustering

The central question is whether understanding *which type* of housing market a country has improves our ability to forecast its price dynamics.

---

## Research Questions

**RQ1 — Forecasting performance**
Can machine learning models outperform naive and linear benchmark approaches when forecasting next-quarter residential house price growth across EU-27 countries?

**RQ2 — Cluster stratification**
Does stratifying predictive models by housing market regime — identified through unsupervised learning or established in the comparative housing literature — improve forecast accuracy over a single pooled model?

**RQ3 — Feature importance**
Which macroeconomic, supply-side, and institutional factors most strongly drive short-term housing price dynamics, and do these vary across market regimes?

---

## Repository Structure

```
ml-final-project/
│
├── data/                          # Raw source files (not committed)
│
├── data_exploring.ipynb           # Notebook 1 — Data pipeline
├── eda.ipynb                      # Notebook 2 — Exploratory data analysis
├── clustering.ipynb               # Notebook 3 — Clustering analysis
├── modelling.ipynb                # Notebook 4 — Model training
├── metrics.ipynb                  # Notebook 5 — Evaluation and comparison
│
├── model_data.xlsx                # Processed panel dataset (output of Notebook 1)
├── model_data_clustered.xlsx      # Panel with cluster labels (output of Notebook 3)
├── predictions.csv                # All model predictions on test set (output of Notebook 4)
├── report_metrics_table.csv       # Final metrics table for the report (output of Notebook 5)
│
└── README.md
```

**Run order:** 1 → 2 → 3 → 4 → 5. Each notebook depends on the output file of the previous one.

---

## Notebooks

### 1. `data_exploring.ipynb` — Data Pipeline

Constructs the final modelling panel from ten raw data sources. Handles frequency alignment (monthly → quarterly, annual → quarterly), country standardisation across Eurostat, ECB, and MARPOR naming conventions, feature engineering (quarter-on-quarter growth rates, two lags for all key predictors, eurozone flags), and forward-filling of slow-moving structural variables.

**Input:** Raw CSV files in `./data/`
**Output:** `model_data.xlsx` — 1,704 rows × 39 columns, 26 EU countries, 2005-Q3 to 2025-Q2

Key variables: HPI growth (target), GDP, HICP inflation, unemployment, building permits, long-term interest rates, ECB mortgage rates, old-age dependency ratio, population density, parliament ideology score (MARPOR rile).

---

### 2. `eda.ipynb` — Exploratory Data Analysis

Systematic exploration of the panel dataset across nine sections: dataset overview, target variable deep-dive, feature distributions, temporal trends, correlation analysis, cross-country heterogeneity, autocorrelation structure, train/test split analysis, and a pre-modelling checklist.

**Input:** `model_data.xlsx`
**Key findings:**
- Target (next-quarter HPI growth) is near-normally distributed (skewness −0.24, kurtosis 1.94)
- Strong autocorrelation at lag-1 (0.632) and lag-4 (0.548), suggesting a quarterly cycle
- `lt_interest_rate` is the strongest single predictor (r = 0.32 with target)
- Eastern European markets (HU, EE, LT) average over twice the growth of Southern markets (IT, FI, ES)
- Train/test split confirmed at 2022: train = 1,340 rows, test = 364 rows (23%)

---

### 3. `clustering.ipynb` — Clustering Analysis

Identifies housing market regimes using two approaches. Includes a full bibliography section with key references from the comparative housing literature.

**Input:** `model_data.xlsx`
**Output:** `model_data_clustered.xlsx` — adds `cluster_data`, `cluster_dbscan`, `cluster_literature`

**Data-driven clustering (PCA + K-Means + DBSCAN):**
- PCA retains 4 components explaining 82% of variance; PC1 captures a growth/dynamism vs ageing/stagnation axis; PC2 captures political orientation and interest rate environment
- K-Means with K=4 selected based on Calinski-Harabasz local maximum, consistent with the four-group literature taxonomy
- DBSCAN (tested at ε = 1.5, 1.75, 2.0) consistently finds only 1 main cluster with 5 persistent noise points (ES, HU, IE, LU, MT), confirming that EU housing markets form a continuum rather than tight density-based clusters
- ANOVA confirms K-Means cluster means differ significantly (F = 13.42, p < 0.001)

**Literature-based clustering (Andrews et al. 2011; Egert & Mihaljek 2007):**

| Group | Countries | Rationale |
|---|---|---|
| West | AT, BE, DE, FR, LU, NL | Coordinated market economies; stable, mature markets |
| North | DK, FI, SE | Nordic welfare states; variable-rate mortgage tradition |
| South | CY, ES, IT, MT, PT | Mediterranean; high homeownership; boom-bust history |
| East | BG, CZ, EE, HR, HU, IE, LT, LV, PL, RO, SI, SK | Post-communist transition; income convergence dynamics |

Ireland is classified East despite its geography because its boom-bust volatility profile (std = 2.94, 5th most volatile) matches Eastern convergence economies rather than the stable Western core.
ANOVA confirms literature cluster means differ significantly (F = 7.56, p < 0.001).

---

### 4. `modelling.ipynb` — Model Training

Trains all models and generates predictions on the held-out test set (2022-Q1 to 2025-Q2). Uses walk-forward expanding-window cross-validation (5 folds, `TimeSeriesSplit`) within the training set for hyperparameter selection. The test set is never used during training.

**Input:** `model_data_clustered.xlsx`
**Output:** `predictions.csv` — 364 rows × 46 columns (7 model metadata columns + 39 prediction columns)

**Phase 1 — Benchmarks (pooled):**
- Naive persistence (predict current quarter's growth)
- Rolling mean (4-quarter window)
- OLS linear regression

**Phase 2 — ML models (pooled):**
- Ridge regression (alpha selected via RidgeCV)
- Lasso regression (alpha selected via LassoCV; performs automatic feature selection)
- Random Forest (grid search over depth and leaf size)
- Gradient Boosting (grid search over learning rate and depth)

**Phase 3 — K-Means stratified:**
All four ML models retrained within each of the 4 K-Means clusters.

**Phase 4 — Literature stratified:**
All four ML models retrained within each of the 4 literature clusters (East, West, South, North).

**Feature set (17 variables):** Two lags each of HPI growth, GDP growth, HICP inflation, unemployment, and building permits; contemporaneous and two lags of long-term interest rate; contemporaneous and one lag of parliament ideology score; eurozone membership flag.

---

### 5. `metrics.ipynb` — Evaluation and Comparison

Loads `predictions.csv` and produces the full comparative evaluation. Answers each research question systematically with tables, charts, and a final auto-generated RQ summary.

**Input:** `predictions.csv`
**Output:** `report_metrics_table.csv`

**RQ1 result:** Lasso is the best pooled model (RMSE = 2.23, −15% vs naive persistence). All pooled models have negative R2 on the 2022–2025 test period, reflecting a structural break caused by the ECB rate hiking cycle — the sharpest monetary tightening in 40 years, which no model trained on 2005–2021 data could fully anticipate.

**RQ2 result:** Literature stratification beats the pooled model in 6/8 RF and GB combinations (average improvement +9.6%). The North cluster (DK, FI, SE) benefits most: GB-North achieves RMSE = 1.44 and R2 = +0.38 — the only model with positive R2 in the study. K-Means stratification is net harmful on average (wins in only 3/8 cases), confirming that theory-grounded clustering outperforms purely data-driven clustering for this application. The South cluster does not benefit from stratification due to its internal heterogeneity (Portugal behaves as a convergence economy; Italy is structurally stagnant; Cyprus is a boom-bust outlier).

**RQ3 result:** `lt_interest_rate_lag1` is the most important feature (importance = 0.167), with all three interest rate variables combined accounting for 30.8% of RF importance. Housing price momentum (HPI lags: 19.2%) ranks second. Inflation dynamics (HICP lags: 14.1%) rank third. Parliament ideology (`parliament_rile` combined: 5.4%) shows non-trivial importance, supporting the inclusion of institutional variables. Eurozone membership (0.4%) is negligible once interest rates are included.

---

## Data Sources

| Source | Dataset | Variables |
|---|---|---|
| Eurostat | `prc_hpi_q` | House Price Index (target) |
| Eurostat | `prc_hicp_midx` | HICP inflation |
| Eurostat | `namq_10_gdp` | Real GDP index |
| Eurostat | `une_rt_q` | Unemployment rate |
| Eurostat | `sts_cobp_q` | Building permits index |
| Eurostat | `demo_r_d3dens` | Population density |
| Eurostat | `demo_pjanind` | Old-age dependency ratio |
| Eurostat | `irt_lt_mcby_q` | Long-term government bond yields |
| ECB API | MIR `A2C.AM.R` | Mortgage interest rates (eurozone) |
| MARPOR | Manifesto Project 2025a | Parliament ideology score (rile) |

---

## Key References

Andrews, D., Caldera Sanchez, A., & Johansson, A. (2011). Housing markets and structural policies in OECD countries. *OECD Economics Department Working Papers*, No. 836. https://doi.org/10.1787/5kgk8t2k9vf3-en

Calza, A., Monacelli, T., & Stracca, L. (2013). Housing finance and monetary policy. *Journal of the European Economic Association*, 11(S1), 101–122. https://doi.org/10.1111/j.1542-4774.2012.01095.x

Egert, B., & Mihaljek, D. (2007). Determinants of house prices in Central and Eastern Europe. *Comparative Economic Studies*, 49(3), 367–388. https://doi.org/10.1057/palgrave.ces.8100221

Geron, A. (2022). *Hands-on machine learning with Scikit-Learn, Keras, and TensorFlow* (3rd ed.). O'Reilly Media.

Hall, P. A., & Soskice, D. (2001). *Varieties of capitalism*. Oxford University Press.

Jolliffe, I. T., & Cadima, J. (2016). Principal component analysis: A review and recent developments. *Philosophical Transactions of the Royal Society A*, 374(2065). https://doi.org/10.1098/rsta.2015.0202

Kemeny, J. (1995). *From public housing to the social market*. Routledge.

---

## Requirements

```
pandas, numpy, matplotlib, seaborn, scikit-learn, scipy, statsmodels, openpyxl, requests
```

Install with:
```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy statsmodels openpyxl requests
```
