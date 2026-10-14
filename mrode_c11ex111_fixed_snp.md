---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Fixed effect model for SNP effects
==================================

Model
-----

In this section, the author estimates individual SNP effects as fixed effects using 8 reference animals that have both genotypes and phenotypes (fat DYD). Only three SNP markers are considered in this small example. We also consider the usual additive genetic (polygenic) effects. The model is
$$
y_{i} = \mu + \sum_{k=1}^{3}z_{ik} g_{k} + u_{i} + e_{i}
$$
where $y_i$ is an observation for animal $i$, $\mu$ is the general mean, $z_{ik}$ is the $k$th weighted marker genotype for animal $i$ (the $(i,k)$ element of the $\mathbf{Z}$ matrix; see VanRaden, 2008), $g_k$ is the $k$th fixed SNP effect, $u_i$ is the additive genetic (polygenic) effect, and $e_i$ is the residual effect. The pedigree can include additional animals. The variance components are $\sigma_a^2 = 35.241$ and $\sigma_e^2 = 245.0$.

The current edition of the textbook contains a typo in equation (11.5). In this equation, $\mathbf{R}^{-1}=\mathbf{D}$, not $\mathbf{R}=\mathbf{D}$. This difference is critical because the weight $d_{ii}$ should contribute directly to the equations, without inversion. In the textbook, the subsequent numerical examples also assume the formula without inversion. Combining $\sigma_e^2$ with a weight ($d_{ii}$ for EDC), the $i$th diagonal element of $\mathbf{R}^{-1}$ is $d_{ii}/\sigma_e^2$.

Files
-----

In this model, the elements of $\mathbf{Z}$ are used as covariates, so they should be saved in the data file. The data file contains the first 3 columns of $\mathbf{Z}$; see p. 181 in the textbook. The weight used in this analysis is the inverse of EDC, so the values should be calculated and stored in the data file (`mr111.dat`).

~~~~~{language=text caption="mr111.dat"}
13  0  0 1 558  9.0  1.357 -0.357  0.286
14  0  0 1 722 13.4  0.357 -0.357 -0.714
...
~~~~~

The data file (`mr111.dat`) has 9 columns.

1. Animal ID
2. Sire ID
3. Dam ID
4. General mean
5. EDC (weight)
6. Phenotype (Fat DYD)
7. Covariate for SNP 1
8. Covariate for SNP 2
9. Covariate for SNP 3

The pedigree file includes all 26 animals (`mr111.ped`). Animals 1 through 14 have missing parents.

The parameter file for the weighted analysis is shown below.

~~~~~{language=blupf90 caption="mr111a.par"}
DATAFILE
mr111.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
5
OBSERVATION(S)
6
WEIGHT(S)
5
EFFECTS:
 4  1 cross   # general mean
 7  1 cov     # SNP effect 1
 8  1 cov     # SNP effect 2
 9  1 cov     # SNP effect 3
 1 26 cross   # additive genetic ( polygenic ) effect
RANDOM_RESIDUAL VALUES
245.0
RANDOM_GROUP
5
RANDOM_TYPE
add_animal
FILE
mr111.ped
(CO)VARIANCES
35.241
OPTION solv_method FSPAK
~~~~~

For the unweighted analysis, you can remove `5` from `WEIGHT(S)`, i.e., leave an empty line for this keyword.

If you want to construct the equations in the form shown in equation (11.5), with $\alpha$ and $\mathbf{R}^{-1}=\mathbf{D}$, you can use the following trick. First, calculate $\alpha=6.952$, then put its inverse, $1/\alpha=1/6.952=0.1438$, into `(CO)VARIANCES` and $1.0$ into `RANDOM_RESIDUAL VALUES`. You do not need to change the rest of the parameter file. This trick is explained in the Quick Tour section.

Solutions
---------

In the weighted and unweighted analyses, the solutions are the same as those in the textbook (p. 188).
