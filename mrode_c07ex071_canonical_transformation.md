---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Canonical transformation
========================

Model
-----

Under some conditions in a multiple-trait model, we may reduce the computational cost of solving the mixed model equations by applying a dimension-reduction method. A canonical transformation is one such approach.

The canonical transformation is a numerical technique for converting correlated variables into uncorrelated variables on an underlying (canonical) scale. With this transformation, an $N$-trait model becomes $N$ independent single-trait models.
It is generally cheaper to repeat the single-trait analysis $N$ times (which can be done in parallel) than to perform a single $N$-trait model analysis. You can back-solve the solutions from the canonical scale to the original scale.

There is a set of strict conditions under which the original canonical transformation is applicable. These conditions include: 1) no missing observations; 2) the same model for all traits; and 3) one random effect in addition to the random residual effect. Subsequently, animal-breeding researchers relaxed all of these conditions (with several practical and theoretical restrictions). Furthermore, the canonical transformation was extended to variance component estimation, although some restrictions still apply.

Let $\mathbf{y}_i$ be a vector of phenotypes for animal $i$ across $N$ traits.
Assume a simple animal model with two random effects: an additive genetic effect and a residual effect.
Without inbreeding, the variance of $\mathbf{y}_{i}$ is $\mathbf{G}_0+\mathbf{R}_0$, where $\mathbf{G}_0$ is the additive genetic covariance matrix and $\mathbf{R}_0$ is the residual covariance matrix.
The canonical transformation determines a matrix $\mathbf{Q}$ that satisfies the following conditions:
$$
\mathbf{Q}\mathbf{R}_0\mathbf{Q}'=\mathbf{I}\quad\text{and}\quad\mathbf{Q}\mathbf{G}_0\mathbf{Q}'=\mathbf{D}
$$
where $\mathbf{D}$ is a diagonal matrix. The matrix $\mathbf{Q}$ is available through the eigenvalue decomposition, shown in the textbook.

Then, we define the transformed observation vector as $\mathbf{y}_{i}^{*}=\mathbf{Qy}_{i}$. In this way, $\mathrm{var}(\mathbf{y}_{i}^{*})=\mathbf{D}+\mathbf{I}$, implying that the transformed variables have no correlations among traits. We can use $\mathbf{y}_{i}^{*}$ as the transformed observations, calculate $\mathbf{\hat{u}}_{i}^{*}$ from independent single-trait analyses, and then back-solve for $\mathbf{\hat{u}}_{i}=\mathbf{Q}^{-1}\mathbf{\hat{u}}_{i}^{*}$. See the textbook for details.

BLUPF90+ does not support the canonical transformation. Therefore, to perform this transformation, the user must prepare the transformed data and back-solve the results manually. The textbook describes the procedure.

Files
-----

You should calculate $\mathbf{Q}$ from $\mathbf{G}_0$ and $\mathbf{R}_0$ manually before the analysis. The textbook gives $\mathbf{Q}$ as
$$
\mathbf{Q} = 
\left[
\begin{array}{rr}
0.1659 & -0.0792 \\
0.0168 & 0.1755
\end{array}
\right]
$$

The data set with the transformed observations is shown in the textbook on p. 125 and is available in the GitHub repository (`mr71.dat`).

1. Animal ID (Calf)
2. Sex (1 = male and 2 = female)
3. Sire ID
4. Dam ID
5. Yearling weight (kg): original scale
6. Fat yield (kg): original scale
7. Yearling weight (kg): transformed scale
8. Fat yield (kg): transformed scale

The pedigree (`mr71.ped`) is from the textbook.

The parameter file should be the following.

~~~~~{language=blupf90 caption="mr71.par"}
DATAFILE
mr71.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
7 8
WEIGHT(S)

EFFECTS:
2 2 2 cross
1 1 8 cross
RANDOM_RESIDUAL VALUES
1 0
0 1
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr71.ped
(CO)VARIANCES
0.3283 0.0000
0.0000 1.3436
OPTION solv_method FSPAK
~~~~~

The covariance components after the transformation include $\mathbf{I}$ for the residual covariance matrix and $\mathbf{D}$ for the additive genetic covariance matrix. This parameter file runs two independent single-trait analyses simultaneously. Alternatively, you may run two separate single-trait analyses and combine the results afterward.

Solutions
---------

The results on the canonical scale are identical to those in the textbook (p. 126). You must back-solve the solutions manually to obtain them on the original scale. For example, for the breeding values of animal 1, you need a solution vector $\mathbf{\hat{u}}_{1}^{*}$ and the inverse of the transformation matrix, $\mathbf{Q}^{-1}$, to obtain the original-scale solutions as $\mathbf{\hat{u}}_{1} = \mathbf{Q}^{-1}\mathbf{\hat{u}}_{1}^{*}$.
