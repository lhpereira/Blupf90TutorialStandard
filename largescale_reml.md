---
title: Large-scale genetic evaluation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

REML estimation with large data
===============================

Background
----------

In typical REML computations, there are several bottlenecks. First, the exact solutions of the mixed-model equations for the current variance components are needed. This means that we must use Gaussian elimination or a similar direct method. Cholesky factorization is often used to solve the equations. The LHS of the mixed-model equations generally contains many zero elements, so the matrix is sparse. Although we can store only the nonzero elements to save memory, factorization requires very complicated operations.

Second, REML algorithms use the first derivative of the restricted log-likelihood, which contains selected elements of the inverse of the LHS. In the animal model, the derivative contains the following trace term:
$$
\mathrm{tr}(\mathbf{A}^{-1}\mathbf{C}^{uu})
$$
where $\mathbf{A}^{-1}$ is the inverse of the numerator relationship matrix and $\mathbf{C}^{uu}$ is the submatrix of the inverse of the left-hand side (LHS) of the mixed model equations corresponding to the solutions for the additive genetic effect ($\hat{\mathbf{u}}$). Simply speaking, the REML equations require the inverse of the LHS. The inverse of a sparse matrix is usually dense (non-sparse). You can easily imagine that its computational and storage costs are extremely high.

In this calculation, we do not actually need the entire inverse. We need only the selected elements of the inverse corresponding to the nonzero elements in the original $\mathbf{C}$. This selected subset is called the sparse inverse in the animal-breeding literature. An algorithm called the Takahashi algorithm can calculate the sparse inverse using the Cholesky factor of the LHS.

FSPAK is a successful computer package for sparse operations, including factorization and sparse inversion of the LHS of the mixed-model equations. This package is called by BLUPF90+ using `OPTION solv_method FSPAK`. It is still useful for small equations, although its back-end subroutines were written more than 20 years ago. Unfortunately, its older design is inefficient for larger equations with many dense blocks. For example, a multiple-trait model (or random-regression or maternal model) contains many genetic and residual covariance matrices in the equations. Although each covariance matrix is small, these matrices are combined into large dense blocks during the sparse operations. This is especially important in a genomic model that includes the inverse of a genomic relationship matrix, which is large and dense.

YAMS is a replacement for FSPAK. This package implements several advanced algorithms, including supernodal factorization and the multifrontal inversion approach, to handle dense blocks in sparse matrices efficiently. YAMS makes extensive use of BLAS and LAPACK. For technical details, see Masuda et al. (2014) and Masuda et al. (2015).

BLUPF90+ provides several options to accelerate computation and stabilize the estimation process. We introduce these options here.

Note that YAMS needs the matrix to be stored in memory, which is a limiting factor for REML in genomic models. Instead of using a direct solver, inversion-free sampling methods may be feasible for variance-component estimation in a large-scale genomic model. Sampling-based REML (Monte Carlo REML) has been suggested for genomic models (Matilainen et al., 2012). The program `blup90iod3+` now implements a sampling algorithm suitable for ssGBLUP (Bermann et al., 2025).

In this section, we will see how YAMS works using a numerical example. The user can use BLUPF90+ to run the example.

Files
-----

We will use the following files.

- [`simdata_rep.txt`](https://github.com/Masuday/data/blob/master/tutorial/simdata_rep.txt) : data file
- [`simped.txt`](https://github.com/Masuday/data/blob/master/tutorial/simped.txt) : pedigree file

We will use a four-trait repeatability model with the following parameter file.

~~~~~{language=blupf90 caption="complicatedparam1.txt"}
DATAFILE
simdata_rep.txt
NUMBER_OF_TRAITS
4
NUMBER_OF_EFFECTS
5
OBSERVATION(S)
9 10 11 12
WEIGHT(S)

EFFECTS: POSITIONS_INDATAFILE NUMBER_OF_LEVELS TYPE_OF_EFFECT  [ EFFECT NESTED ]
6 6 6 6   155 cross
7 7 7 7     2 cross
8 8 8 8    11 cross
1 1 1 1  4641 cross
1 1 1 1  4641 cross
RANDOM_RESIDUAL VALUES
100 80 80 80
80 100 80 80
80 80 100 80
80 80 80 100
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
simped.txt
(CO)VARIANCES
100 80 80 80
80 100 80 80
80 80 100 80
80 80 80 100
RANDOM_GROUP
5
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
100 80 80 80
80 100 80 80
80 80 100 80
80 80 80 100
OPTION use_yams
OPTION EM-REML 10
~~~~~

Here we specify two options.

~~~~~{language=blupf90}
OPTION use_yams
~~~~~

This option switches the sparse library from FSPAK to YAMS. If the program has multithreaded BLAS and LAPACK, the computations will be parallelized. This option is also effective in BLUPF90 in combination with `OPTION solv_method FSPAK`: it uses YAMS instead of FSPAK.

~~~~~{language=blupf90}
OPTION EM-REML n   # default :0
~~~~~

This option forces the program to perform EM-REML (equivalent to REMLF90) during the first $n$ rounds. The default is 0; that is, no EM rounds are performed, and AI rounds start from round 1. For complicated models, AI-REML often diverges in the first round. Although this option can help in this situation, it does not always prevent the program from diverging.

We have several additional options to accelerate the computations.

~~~~~{language=blupf90}
OPTION fact_once x
~~~~~

This option avoids recalculating the Cholesky factor. It saves the Cholesky factor in temporary memory (if `x` is `memory`) or in a temporary file (if `x` is `file`). If you have enough memory, `memory` is preferable because it provides faster computation.

~~~~~{language=blupf90}
OPTION approx_loglike
~~~~~

This skips the computation of the exact log-likelihood. If you do not need the exact value, we recommend using this option to speed up the computation.


Results
-------

The following results are available after 17 rounds.

~~~~~{language=output}
 new R
   45.124       22.357       18.626       13.762
   22.357       44.210       22.690       18.016
   18.626       22.690       46.101       22.795
   13.762       18.016       22.795       45.274
 new G
   41.967       22.512       24.058       26.907
   22.512       17.489       19.738       24.257
   24.058       19.738       28.775       34.741
   26.907       24.257       34.741       56.668
 new G
   15.546       15.237       10.490       1.4105
   15.237       40.937       19.562       6.4840
   10.490       19.562       27.782       4.5379
   1.4105       6.4840       4.5379       2.4410
~~~~~

You can try an alternative parameter file without `use_yams` and `EM-REML`. Without YAMS, the computations will be slow. Without EM-REML, the estimates may diverge in the first round. You can also try `fact_once` and `approx_loglike`. Although you may not be able to determine whether these options accelerate computation in this small example, they can substantially reduce computing time in a larger analysis.
