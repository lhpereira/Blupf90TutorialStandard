---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Mixed linear model for computing SNP effects
============================================

Model
-----

When we have many SNP markers, we can expect that almost all additive genetic variation can be captured by the markers. In this case, the marker effects should be random because there are too many effects to estimate relative to the number of observations. With the random-marker model, we may not need a separate polygenic effect. In this example, the author assumes the following model:
$$
y_{i} = \mu + \sum_{k=1}^{m}z_{ik} g_{k} + e_{i} = \mathbf{1}\mu + \mathbf{Zg} + \mathbf{e}
$$
where $y_i$ is the observation for animal $i$, $\mu$ is the general mean, $m$ is the number of markers considered, $z_{ik}$ is the $k$th weighted marker genotype for animal $i$ (the $(i,k)$ element of $\mathbf{Z}$), $g_k$ is the $k$th random marker effect, and $e_i$ is the residual effect. The system of mixed model equations is
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z}\\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z}+\mathbf{I}/\sigma_g^2
\end{array}
\right]
\left[
\begin{array}{c}
\mathbf{\hat{b}}\\
\mathbf{\hat{g}}
\end{array}
\right]
=
\left[
\begin{array}{l}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{y} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{y} \\
\end{array}
\right].
$$
As described in the previous section, the textbook contains a typo: $\mathbf{R}^{-1}$ is the diagonal matrix of weights, whereas the textbook mistakenly states that $\mathbf{R}$ is the weight matrix. Again, this difference is critical. Combining $\sigma_e^2$ with a weight ($d_{ii}$ for EDC), the $i$th diagonal element of $\mathbf{R}^{-1}$ is $d_{ii}/\sigma_e^2$.

The residual variance is given as $\sigma_e^2 = 245.0$ in the textbook. The marker variance ($\sigma_g^2$) can be estimated from the additive genetic variance ($\sigma_a^2$) using $\sigma_a^2/\left[2\sum p_j(1-p_j)\right]$, where $p_j$ is the allele frequency for marker $j$. The genetic variance is taken from the previous section as $\sigma_a^2 = 35.241$. The author uses this equation and gives $2\sum p_j(1-p_j) = 3.5383$, so the variance components are $\sigma_g^2 = 35.241/3.5383 = 9.96$.


Files
-----

The data file (`mr112.dat`) now contains 10 columns corresponding to $\mathbf{Z}$ (p. 190).

1. Animal ID
2. Sire ID
3. Dam ID
4. General mean
5. EDC (weight)
6. Phenotype (Fat DYD)
7. Covariate for SNP 1
8. Covariate for SNP 2
9. Covariate for SNP 3
10. Covariate for SNP 4
11. Covariate for SNP 5
12. Covariate for SNP 6
13. Covariate for SNP 7
14. Covariate for SNP 8
15. Covariate for SNP 9
16. Covariate for SNP 10

We can use the same pedigree as before (`mr112.ped`).

The parameter file contains 10 SNP effects.

~~~~~{language=blupf90 caption="mr112.par"}
DATAFILE
mr112.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
11
OBSERVATION(S)
6
WEIGHT(S)
5
EFFECTS:
 4  1 cross   # general mean
 7  1 cov     # SNP effect 1
 8  1 cov     # SNP effect 2
 9  1 cov     # SNP effect 3
10  1 cov     # SNP effect 4
11  1 cov     # SNP effect 5
12  1 cov     # SNP effect 6
13  1 cov     # SNP effect 7
14  1 cov     # SNP effect 8
15  1 cov     # SNP effect 9
16  1 cov     # SNP effect 10
RANDOM_RESIDUAL VALUES
245.0
RANDOM_GROUP
2 3 4 5 6 7 8 9 10 11
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
9.96 0 0 0 0 0 0 0 0 0
0 9.96 0 0 0 0 0 0 0 0
0 0 9.96 0 0 0 0 0 0 0
0 0 0 9.96 0 0 0 0 0 0
0 0 0 0 9.96 0 0 0 0 0
0 0 0 0 0 9.96 0 0 0 0
0 0 0 0 0 0 9.96 0 0 0
0 0 0 0 0 0 0 9.96 0 0
0 0 0 0 0 0 0 0 9.96 0
0 0 0 0 0 0 0 0 0 9.96
OPTION solv_method FSPAK
~~~~~

In the above parameter file, we define 10 SNP effects as a group of random effects. The covariances among the effects are 0, so all the SNP effects are independent of one another. This specification is equivalent to defining each SNP effect separately (that is, using 10 `RANDOM_GROUP` blocks). A user can confirm that these two parameter files produce the same results.

For the unweighted analysis, you can remove `5` from `WEIGHT(S)`, i.e., leave an empty line for this keyword.

Solutions
---------

The solutions from both the weighted and unweighted analyses are identical to those in the textbook (p. 191).
