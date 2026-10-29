---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Cholesky transformation
=======================

Model
-----

A canonical transformation is useful for reducing the dimensionality of data. However, there are several restrictions on the transformation, including the requirement that no observations be missing. Although missing observations can theoretically be handled in the canonical transformation, doing so is usually costly in practice.

A second option is a Cholesky transformation, which naturally handles missing observations. It converts correlated residuals into uncorrelated residuals, but genetic correlations among traits remain. In some models, zero residual covariances may be advantageous for computation.

We do not show the formulas or theoretical background of the Cholesky factorization here. Please see the textbook for details.

Files
-----

BLUPF90+ does not support the Cholesky transformation. Therefore, the user should prepare the transformed observations manually before the analysis. The transformed data set `mr72.dat` and the pedigree file `mr72.ped` are available through the GitHub repository.

Given these data, the parameter file should be as follows.

~~~~~{language=blupf90 caption="mr72.par"}
DATAFILE
mr72.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
7 8
WEIGHT(S)

EFFECTS:
2 2 2 cross
1 1 9 cross
RANDOM_RESIDUAL VALUES
1 0
0 1
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr72.ped
(CO)VARIANCES
0.500000 0.380539
0.380539 1.171972
OPTION solv_method FSPAK
~~~~~

Solutions
---------

The results on the transformed scale are identical to those in the textbook (p. 129). As with the canonical transformation, you must back-solve the solutions manually to obtain them on the original scale.
