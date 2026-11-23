# Nifty 50 Clustering Case Study: Knowledge Base

## Source data

- `Nifty 50 List.csv` — 50 rows, 5 columns: `Company Name`, `Industry`,
  `Symbol`, `Series` (constant `EQ`, no information), `ISIN Code` (unique
  identifier, drop). No numeric columns, no duplicates.
- `Industry` has 13 distinct values, several singleton categories
  (`SERVICES`, `TELECOM`, `CONSTRUCTION`, `FERTILISERS & PESTICIDES` each
  have exactly 1 company). Kept as-is per explicit instruction to one-hot
  encode categoricals rather than collapsing rare categories.

## Derived numeric features (not present in the source CSV)

The source list has no numeric columns at all, so there is nothing to
cluster on as-is. Confirmed with the user: fetch real historical daily
price/volume history per NSE symbol via `yfinance` (ticker = `SYMBOL.NS`),
3-year lookback, and derive per-company features from daily returns:

| Feature | Definition |
|---|---|
| `Annual_Return` | mean daily return × 252 |
| `Annual_Volatility` | std of daily return × sqrt(252) |
| `Beta` | cov(stock return, `^NSEI` index return) / var(index return) |
| `Avg_Daily_Volume` | mean daily traded volume (log-transformed before scaling — heavily right-skewed, ~3 orders of magnitude spread) |
| `Max_Drawdown` | min of (cumulative return / running-max cumulative return − 1) |
| `Sharpe_Ratio` | `Annual_Return / Annual_Volatility` (risk-free rate treated as 0) |

Raw fetch cached at
`Datasets/Practice/Agglomorative_HeirarchialCLustering/Nifty_50/nifty50_features_raw.csv`
(merged with the source list). The notebook's fetch cell checks for this
cache file and only re-hits `yfinance` if it's missing, for fast re-runs.

## Verified data issues

- **2 genuinely missing rows** (all 6 derived features NaN), both with a
  real, nameable cause rather than a random fetch glitch:
  - `HDFC` — Housing Development Finance Corporation merged into HDFC Bank
    in 2023; the standalone ticker no longer trades/has no recent history.
  - `TATAMOTORS` — Yahoo Finance returns "quote not found" for this ticker
    currently, consistent with Tata Motors' 2024 corporate restructuring.
  - Remedy: per-column median imputation (same policy as
    `regression-data-cleaning`'s missing-value matrix).
- **No duplicate entities.**
- **IQR-based outlier counts** (after median imputation, before scaling):
  `Annual_Return` 1, `Annual_Volatility` 2, `Log_Avg_Daily_Volume` 1,
  `Max_Drawdown` 1, `Beta` 0, `Sharpe_Ratio` 0. Capped at the standard
  `Q1 - 1.5*IQR` / `Q3 + 1.5*IQR` bounds, consistent with prior case
  studies' default remedy (cap, don't drop) — dropping would remove an
  entity from the clustering deliverable entirely.
- **`Avg_Daily_Volume` is heavily right-skewed** (range ~3.5e4 to ~3.5e7,
  ~3 orders of magnitude) — treated with `np.log1p` before scaling, not as
  an outlier-capping problem, since capping would discard real information
  about which companies are more liquid.

## Resolved decisions

- **Models to build and compare**: KMeans (random init), KMeans++ (smart
  init), Agglomerative Hierarchical Clustering (ward linkage) — no other
  algorithms.
- **Preprocessing**: one shared `ColumnTransformer` — `StandardScaler` on
  the 6 numeric features, `OneHotEncoder` on `Industry` — explicitly
  confirmed with the user. Built as a reusable sklearn `Pipeline`
  (`clustering_preprocessor`), not ad hoc per-model scaling.
- **No train/test split** — this is unsupervised; every company gets a
  cluster assignment from the full dataset, there is no held-out
  evaluation set.
- **k selection for Agglomerative**: dendrogram (ward linkage) read for the
  largest jump in merge (cophenetic) distance among the last several
  merges, confirmed against the silhouette-vs-k curve (k = 2..10).
  Silhouette is this model's primary metric.
- **k selection for KMeans/KMeans++**: elbow method on WCSS/inertia across
  k = 2..10, confirmed against the same silhouette-vs-k convention. Dunn
  index (custom implementation — no sklearn built-in) reported as a
  secondary metric for every model built in the case study, not just
  KMeans.
- **Final comparison**: cross-algorithm table (silhouette, Dunn index,
  cluster-size balance) + 2D PCA scatter per algorithm + per-cluster
  profiling in original units (mean/median per numeric feature, dominant
  `Industry` per cluster) for the winning algorithm.
- **Gemini narration**: reuse `classification-gemini-narration` as-is.
- Notebook: `Heirarchial_Clustering/unsupervised_heirarchial_nifty.ipynb`.
