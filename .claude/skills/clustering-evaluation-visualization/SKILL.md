---
name: clustering-evaluation-visualization
description: Generalized phase skill for any unsupervised clustering case study — the cross-algorithm comparison table (silhouette, Dunn index, cluster sizes), the shared PCA-scatter visualization convention, and per-cluster profiling of the original features. Use as the closing phase, after Agglomerative/KMeans/KMeans++ have each produced a final labeling.
---

# Clustering Case Study: Evaluation & Comparison

## Guardrails

- **One shared `evaluate_clustering()` helper**, called for every algorithm's
  final labeling (Agglomerative, KMeans, KMeans++) so the comparison table
  has identical columns: silhouette score, Dunn index, number of clusters,
  cluster size distribution (min/max/count per cluster — a clustering that
  dumps 48 of 50 entities into one cluster and 2 singletons is a real
  failure mode worth catching here, not hidden behind a single scalar
  metric).
- **A clustering comparison has no "best by accuracy" answer** the way a
  supervised comparison does — present the table candidly and let higher
  silhouette + higher Dunn + balanced cluster sizes argue for a winner, but
  say explicitly if the algorithms substantially agree or disagree on
  structure (e.g. via a cross-tab of one algorithm's labels against
  another's) rather than only comparing scalar metrics in isolation.
- **Visualize every final clustering the same way**: a 2D PCA scatter
  colored by cluster label, consistent color palette convention across all
  three algorithms' plots so a reader can visually compare cluster shapes
  at a glance; state the 2-component explained variance ratio once and
  reuse it as the caveat for every such plot.
- **Per-cluster profiling in the original (unscaled) units** — for the
  winning algorithm's clusters, show the mean/median of each original
  numeric feature per cluster, and the categorical composition (e.g.
  dominant category per cluster), so the clusters get a human-readable
  story ("cluster 2 = high-growth, high-volatility, financial-services-
  heavy") rather than staying abstract integers.
- **No target to validate against** — don't frame any part of this as
  accuracy; the entire chapter is about internal validity (separation,
  compactness, balance) and interpretability of the groups found.
- Follow the project's shared notebook conventions (seaborn-only charts,
  motive cell before code, narrated takeaway after every chart/table).
