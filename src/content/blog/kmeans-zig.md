---
author: dbubel
pubDatetime: 2024-09-18T15:22:00Z
title: K-means clustering in Zig
slug: k-means-clustering-zig
featured: false
draft: true 
tags:
  - k-means
description: Zig implementation of k-means clustering.
---

# k-Means overview

- **Definition**:
  - k-Means Clustering is a popular unsupervised machine learning algorithm used for partitioning a dataset into k distinct, non-overlapping clusters.

- **Key Concepts**:
  - **Centroids**: Each cluster is represented by its center point, called a centroid.
  - **Iterations**: The algorithm iteratively refines the location of the centroids until the "mean" position changes less than a defined constant Epsilon (E).
  - Massively sensitive to which centroids are chosen initially

# Pseudo algoritm
```
Dataset D = [(1, 2), (3, 4), (5, 6), (8, 8)]
Number of clusters K = 2

1. Initialize centroids randomly:
   Centroids = [(1, 2), (8, 8)]

2. Repeat until convergence:

   2.1. Assignment Step:
       - Calculate distances and assign points to clusters:
         - (1, 2) -> closest to (1, 2)
         - (3, 4) -> closest to (1, 2)
         - (5, 6) -> closest to (8, 8)
         - (8, 8) -> closest to (8, 8)
       - Cluster assignments:
         - Cluster 1: [(1, 2), (3, 4)]
         - Cluster 2: [(5, 6), (8, 8)]

   2.2. Update Step:
       - Recalculate centroids:
         - New centroid for Cluster 1: ( (1+3)/2, (2+4)/2 ) = (2, 3)
         - New centroid for Cluster 2: ( (5+8)/2, (6+8)/2 ) = (6.5, 7)

   - Repeat the assignment and update steps with the new centroids until convergence.
```

