# Nifty 50 Market-Behavior Clustering

Unsupervised clustering of all 50 NSE Nifty 50 index constituents, grouped
by how they actually trade rather than by their static GICS-style industry
label. Two companies in the same labeled industry can behave very
differently in the market, and this analysis groups them by real financial
behavior instead.

## Data

The source list (`Nifty 50 List.csv`) has only company identifiers and an
industry label, no numeric columns. Three years of daily price/volume
history per NSE symbol is fetched live via `yfinance`, and six features are
derived from each company's daily return series:

- `Annual_Return`, `Annual_Volatility`, `Beta` (vs. the Nifty index),
  `Avg_Daily_Volume`, `Max_Drawdown`, `Sharpe_Ratio`.

Two companies (`HDFC`, `TATAMOTORS`) have no usable price history due to
real corporate actions (the 2023 HDFC/HDFC Bank merger and Tata Motors'
2024 restructuring) and are median-imputed rather than dropped.

## Methodology

- Cleaning: median imputation for the two missing companies, a log
  transform on the heavily right-skewed trading-volume feature, IQR-based
  outlier capping on the rest.
- Preprocessing: one shared `ColumnTransformer` — `StandardScaler` on the
  six numeric features, `OneHotEncoder` on `Industry`.
- Three clustering algorithms, compared on identical terms: **Agglomerative
  Hierarchical Clustering** (dendrogram-driven k selection), **KMeans**, and
  **KMeans++**, each evaluated with silhouette score and a from-scratch
  Dunn index implementation.
- Cluster count (k=3) chosen from the dendrogram's largest merge-distance
  gap, cross-checked against the elbow/silhouette sweeps for both KMeans
  variants.

## Results

| Model | Silhouette | Dunn Index |
|---|---|---|
| Agglomerative (Ward) | 0.207 | **0.241** |
| KMeans (random init) | **0.217** | 0.171 |
| KMeans++ | **0.217** | 0.171 |

KMeans++ is selected as the winning model on silhouette (this project's
primary clustering metric); Agglomerative posts the higher Dunn index, and
that disagreement is reported directly rather than hidden. All three
algorithms substantially agree on the same 3-way split (confirmed via a
cross-tab of cluster assignments), which is the stronger evidence that the
groups reflect real structure in how these companies trade.

## Files

- `Heirarchial_Clustering/unsupervised_heirarchial_nifty.ipynb` — the full
  analysis notebook.
- `Heirarchial_Clustering/nifty50_cluster_assignments.csv` — final
  company-to-cluster mapping.
- `Datasets/Practice/Agglomorative_HeirarchialCLustering/Nifty_50/` —
  source list, cached derived-feature fetch, and a knowledge base documenting
  every data/methodology decision.
- `.claude/skills/` — the reusable analysis skills this notebook was built
  from (clustering-specific plus the generalized preprocessing/narration
  skills reused from this project's other case studies).
