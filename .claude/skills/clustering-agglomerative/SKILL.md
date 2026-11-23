---
name: clustering-agglomerative
description: Generalized phase skill for any unsupervised clustering case study — the dendrogram-driven workflow for Agglomerative Hierarchical Clustering, how to read it to pick a candidate k, and confirming that choice against the silhouette-vs-k curve. Use after the shared scaling/encoding pipeline is built, as one of the clustering model phases.
---

# Clustering Case Study: Agglomerative Hierarchical Clustering

## Guardrails

- **Fit on the same preprocessed feature matrix every other clustering
  model in this case study uses** (`ColumnTransformer` output: scaled
  numeric + one-hot categorical) — never a different subset or an
  unscaled version just for this model.
- **Dendrogram first, model second.** Plot the dendrogram
  (`scipy.cluster.hierarchy.dendrogram` + `linkage`, `method="ward"` unless
  EDA gave a specific reason to prefer another linkage) before fitting
  `AgglomerativeClustering` at all — the dendrogram is the evidence a k
  choice is built on, not a decoration added after the fact.
- **Reading the dendrogram for k**: look at the merge (cophenetic) distances
  on the y-axis for the last several merges — print them alongside the
  plot, don't eyeball only. The candidate k is `(number of merges) -
  (index of the largest jump in merge distance)`, i.e. cut the tree where
  the vertical gap between consecutive merge heights is largest and stays
  roughly flat below that — state the actual distances used to make this
  call, not just the resulting number.
- **Confirm, don't just assert, the dendrogram's candidate k** against a
  silhouette-score-vs-k curve (k from 2 up through at least 10): plot it,
  and pick the k that is both a good local silhouette peak *and* consistent
  with where the dendrogram's descent flattens — if the two disagree, show
  both and explain the final choice rather than silently picking one.
- **Silhouette score is this model's primary metric** (matching the
  project's convention of one consistent metric per model family) — report
  it for the final chosen k, and keep the full k-sweep table/chart in the
  notebook so the choice is auditable.
- **No train/test split** — fit `AgglomerativeClustering(n_clusters=k)` on
  the full preprocessed matrix and assign every entity a cluster label;
  there is nothing to "predict" out of sample in this case study (unlike a
  supervised pipeline's held-out test set).
- **Visualize the final clusters**, not just report the k and score — at
  minimum a 2D PCA (or other dimensionality-reduction) scatter colored by
  cluster label, with the explained-variance-ratio of the 2 components
  stated so the plot's trustworthiness is clear.
