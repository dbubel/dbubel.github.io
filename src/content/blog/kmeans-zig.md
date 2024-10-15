---
author: dbubel
pubDatetime: 2024-09-18T15:22:00Z
title: K-means clustering in Zig
slug: k-means-clustering-zig
featured: false
draft: false
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

# Applications

- Customer segmentation\*
- Image compression
- Document clustering

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

# Customer Segmentation

- Task
  - Divide a customer base into distinct groups with similar characteristics to tailor marketing strategies and improve customer satisfaction.
- Benefits
  - **Personalized Marketing**: Target specific segments with tailored campaigns and promotions.
  - **Product Recommendations**: Suggest products based on the preferences of each customer segment.
  - **Customer Retention**: Identify at-risk segments and develop strategies to retain them.
  - **Resource Allocation**: Allocate marketing resources more effectively by focusing on high-value segments.

# Very simple example

- **Original Data**:
  - Customer 1: Age=30, Spending Habits=500, Purchase Frequency=10, Customer Type='VIP'
  - Customer 2: Age=25, Spending Habits=300, Purchase Frequency=15, Customer Type='new'
- **Normalized and Encoded Data**:
  - Customer 1: Age=0.6, Spending Habits=0.8, Purchase Frequency=0.4, Customer Type='VIP' -> [0.6, 0.8, 0.4, 0, 0, 1]
  - Customer 2: Age=0.5, Spending Habits=0.5, Purchase Frequency=0.6, Customer Type='new' -> [0.5, 0.5, 0.6, 1, 0, 0]
