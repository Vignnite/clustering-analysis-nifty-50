---
name: nifty50-clustering-case-study
description: Orchestrator for the Nifty 50 unsupervised clustering case study (Datasets/Practice/Agglomorative_HeirarchialCLustering/Nifty_50/Nifty 50 List.csv plus yfinance-derived features, notebook at "Heirarchial_Clustering/unsupervised_heirarchial_nifty.ipynb"). Use whenever asked to build, extend, re-run, or reason about this case study end to end. Sequences reused regression-* skills (preprocessing mechanics generalize across supervised/unsupervised) with the new clustering-* skills in order.
---

# Nifty 50 Clustering Case Study: Orchestrator

Before doing anything else, read
`Datasets/Practice/Agglomorative_HeirarchialCLustering/Nifty_50/Knowledge_Base.md`
in full — verified schema, the derived-feature definitions, the data
quality findings (2 real missing rows, outlier counts), and every resolved
decision. No need to re-explore the raw CSV or re-derive those decisions.

## Working conventions

- Deliverable notebook:
  `Heirarchial_Clustering/unsupervised_heirarchial_nifty.ipynb`.
- Gemini key: reuse whichever `.env` (`GOOGLE_API_KEY`) this project's other
  case studies already use.
- This is **unsupervised** — no target column, no train/test split. Every
  phase below that was written for a supervised case study generalizes
  cleanly to the data-mechanics level (duplicates, missing values, IQR
  outliers, scaling/encoding) but drop any target-framed language from
  those skills' examples when applying them here.
- If a new data quirk or decision is discovered mid-analysis, update
  `Knowledge_Base.md` rather than only fixing it inline.

## Pipeline (execute in this order)

1. **Data acquisition** (dataset-specific, not a generalized skill): load
   `Nifty 50 List.csv`, fetch 3-year daily price/volume history per symbol
   via `yfinance` (ticker = `SYMBOL.NS`), cache the derived feature table to
   `nifty50_features_raw.csv` and skip re-fetching on subsequent runs if
   that cache exists. `.head()`/`.shape` checkpoint after the merge.
2. **`regression-eda-profiling`** (reused) — shape/dtype/null/duplicate
   profiling, format-issue checks, numeric distribution pass on the 6
   derived features, categorical cardinality pass on `Industry`. Drop any
   language about a "target distribution" — there isn't one.
3. **`regression-data-cleaning`** (reused) — the 2 genuinely-missing rows
   (`HDFC`, `TATAMOTORS`) via median imputation, explicitly named root
   cause for each (merger / corporate restructuring); `np.log1p` on
   `Avg_Daily_Volume` for its right-skew; IQR-based outlier capping on the
   remaining numeric features (cap, don't drop — every entity must keep a
   row to receive a cluster label).
4. **`regression-visualization`** (reused, subset that applies) — numeric
   feature distributions, correlation heatmap across the 6 features,
   outlier boxplots. Skip the target-specific chart types (feature-vs-
   target scatter, high/low performer split) since there's no target.
5. **`regression-encoding-multicollinearity`** (reused, subset that
   applies) — build the one shared `ColumnTransformer`/`Pipeline`:
   `StandardScaler` on the 6 numeric features, `OneHotEncoder` on
   `Industry`. Skip the VIF sub-workflow (that skill's multicollinearity
   framing is target-driven; note explicitly why it's skipped here).
6. **`clustering-agglomerative`** (new) — dendrogram, candidate k from the
   largest merge-distance jump, confirmed against the silhouette-vs-k
   curve, final `AgglomerativeClustering` fit + PCA visualization.
7. **`clustering-kmeans`** (new) — KMeans and KMeans++ each across the same
   k sweep, elbow (WCSS) + silhouette-vs-k curves, Dunn index, final fits +
   PCA visualization for both.
8. **`clustering-evaluation-visualization`** (new) — the cross-algorithm
   comparison table (silhouette, Dunn index, cluster-size balance), cross-
   tab of cluster agreement between algorithms, per-cluster profiling in
   original units for the winning algorithm.
9. **`classification-gemini-narration`** (reused, not rebuilt) — the shared
   `generate_insight` helper, set up early (right after imports), used for
   substantive remark cells throughout (data-acquisition findings, cleaning
   decisions, each clustering model's results, the final cross-algorithm
   comparison and business interpretation of the winning clusters).

Final deliverable beyond the notebook itself: the winning algorithm's
cluster assignments saved as a CSV
(`Heirarchial_Clustering/nifty50_cluster_assignments.csv`) mapping each
company to its final cluster label, plus a one-paragraph profile per
cluster.
