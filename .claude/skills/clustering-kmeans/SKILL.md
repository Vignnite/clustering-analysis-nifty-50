---
name: clustering-kmeans
description: Generalized phase skill for any unsupervised clustering case study — building and comparing both KMeans (random centroid init) and KMeans++ (smart init) using the elbow method (WCSS/inertia) and the silhouette-vs-k curve to pick k, plus the Dunn index as a secondary metric. Use after the shared scaling/encoding pipeline is built, alongside the agglomerative phase.
---

# Clustering Case Study: KMeans and KMeans++

## Guardrails

- **Fit on the identical preprocessed feature matrix** used by every other
  clustering model in the case study (same `ColumnTransformer` output) —
  this is what makes the final cross-algorithm comparison fair.
- **Build both variants explicitly, not just the sklearn default.**
  `KMeans(init="random", n_init=10)` (plain KMeans) and
  `KMeans(init="k-means++", n_init=10)` (KMeans++, also sklearn's default)
  — fit both across the same k sweep so the comparison between them is a
  real side-by-side, not an assumption that ++ is simply better.
- **Elbow method on WCSS/inertia**: sweep k (e.g. 2 through 10), plot
  inertia vs. k for both variants, and read the elbow as the k where the
  marginal drop in inertia flattens — print the actual inertia values used
  to justify the chosen elbow point, not just a verbal "elbow around k=X".
- **Silhouette-vs-k curve as the second, corroborating signal** — same k
  sweep, same plot convention as the agglomerative phase, so all three
  algorithms' k-selection charts are visually comparable in the notebook.
  If the elbow and the silhouette peak disagree, show both curves and
  explain the final k choice rather than picking silently.
- **Dunn index as a secondary metric for every model built in this case
  study** (not just KMeans) — implement it directly (no sklearn built-in):
  `min inter-cluster distance / max intra-cluster diameter`, both computed
  from the same pairwise distance matrix on the preprocessed feature
  matrix. A higher Dunn index is better (well-separated, compact clusters);
  report it alongside silhouette in every comparison table, since the two
  metrics can disagree and that disagreement is informative, not noise.
- **No train/test split** — fit on the full preprocessed matrix; `.labels_`
  is the deliverable cluster assignment for every entity.
- **Visualize the final clustering** for both KMeans and KMeans++ at the
  chosen k: 2D PCA scatter colored by cluster, same convention as the
  agglomerative phase's visualization, so all three algorithms' final
  results can sit side by side in the notebook's closing comparison.
