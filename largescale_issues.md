---
title: Large-scale genetic evaluation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Issues in a large scale analysis
================================

General issues
--------------

Recent computers can complete genetic evaluations or variance-component estimation in a practical amount of time, even with relatively large data sets and complicated models that could not be handled in the past. Even with modern computers, a genetic evaluation can be challenging in terms of both computing time and memory requirements. BLUPF90+ and GIBBSF90+ are designed to store the system of mixed-model equations (MME) in memory, and they can handle up to 1 million pedigree animals in a single-trait model. This is not sufficient for a large-scale analysis involving more animals and effects. Issues specific to genomic data are considered later.

There are several issues to be solved in large-scale genetic evaluation.

- Solving the mixed-model equations.
- Calculating the accuracy or reliability of individual EBVs.
- Estimation of variance components.

The mixed-model equations may be too large to store in memory for a large-scale analysis. Therefore, we need a way to solve the equations without explicitly creating them. In this case, we cannot directly calculate the inverse of the left-hand side, and the prediction-error variance (PEV) of an animal's EBV is not available. Even if the mixed-model equations are small enough to fit in memory, the computational cost of calculating the inverse can be extremely high, making this calculation impractical in many cases. We usually use subsets of the full data for variance-component estimation. Nevertheless, the computation can take a very long time or fail to finish if the data set is large or the model is complicated.

We have several options for large-scale analysis. The BLUP90IOD3 (formerly BLUP90IOD2) program supports the iteration-on-data technique. This technique is combined with an iterative method for solving the equations. The algorithm partially builds the equations while reading the data and pedigree files and indirectly updates the solutions. When the files have been read, one round of iteration is complete. BLUP90IOD3 implements the Preconditioned Conjugate Gradient (PCG) method as its iterative method. See Tsuruta et al. (2001) for details.

The ACCF90 program approximates the accuracy or reliability of an animal's EBV. This program creates approximate elements of the left-hand-side matrix for each animal. After collecting all the information, it inverts these elements to obtain the approximate prediction-error variance (PEV). This is an iterative method, but it usually converges quickly. The basic idea comes from Misztal and Wiggans (1988) and Strabel et al. (2001).

AI-REML is a primary method for variance-component estimation. A typical AI-REML computation involves inverting the left-hand side of the mixed-model equations. The FSPAK package quickly calculates selected elements of the inverse needed for AI-REML. It can still perform well for small-scale analyses, but it is impractical for large-scale analyses. The package was written more than 20 years ago, and its fundamental routine was written in the early 1980s. The newer YAMS (Yet Another MME Solver) package can remove this bottleneck. Although results from YAMS are compatible with those from the previous software, YAMS supports parallel computing and other modern techniques for highly efficient computation. BLUPF90+ can use YAMS through an option. See Masuda et al. (2014) and Masuda et al. (2015) for details.

Genomic-specific issues
-----------------------

The issues above are more severe when we have many genotypes. A genomic data set is generally large and dense, and a genomic model usually requires more memory and storage than a pedigree model. A large-scale genomic evaluation has the following additional issues.

- Computation of a genomic relationship matrix and/or related matrices.
- Slow convergence with the genomic data in solving the mixed model equations.

Animal breeders and quantitative scientists have developed specialized models, algorithms, and computer programs to overcome these issues. The APY (Algorithm for Proven and Young) method is one of the methods to avoid full computation of $\mathbf{G}^{-1}$ (Misztal, 2016). A sparse-matrix operation is useful to represent $\mathbf{A}_{22}^{-1}$ indirectly (Masuda et al., 2015). Although the problems remain and there is room for improvement, most have been solved or sufficiently mitigated for routine genomic evaluation. We need state-of-the-art, highly optimized, specialized software for this purpose, and the BLUPF90 family provides such a suite, including BLUP90IOD3 and ACCF90GS/ACCF90GS2 (Lourenco et al., 2026).

We will not cover the details of genomic tools for large-scale ssGBLUP. The most recent information is available in the official DokuWiki and research articles.
