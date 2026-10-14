---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Multiple-trait model with equal design matrix and missing records
=================================================================

Model
-----

Here, we apply the same model described in the previous example to data with missing observations. The model description and mixed model equations are identical to those in the previous example.

When an observation is missing, $\mathbf{R}_0$ and its inverse should be modified. For example, assuming a two-trait model, if the observation for the first trait is missing, the first row and column of $\mathbf{R}_0$ should be set to zero. The corresponding inverse is the generalized inverse of this modified $\mathbf{R}_0$. Applying this to the previous example gives
$$
\mathbf{R}_{0}
=
\left[
\begin{array}{rr}
0&0\\
0&30
\end{array}
\right]
\quad
\text{and}
\quad
\mathbf{R}_{0}^{-}
=
\left[
\begin{array}{rr}
0&0\\
0&30
\end{array}
\right]^{-}
=
\left[
\begin{array}{rr}
0&0\\
0&1/30
\end{array}
\right].
$$

A generalized inverse of this zero-filled matrix is equivalent to the inverse of a matrix containing only the nonzero elements of the zeroed matrix (Searle, 1971). BLUPF90+ can detect a missing observation and construct an appropriate $\mathbf{R}_{0}$ and its generalized inverse.


Files
-----

One animal is added to the previous example, and two observations are marked as missing. A missing observation is indicated by `0`, which is the default missing code used in the BLUPF90 family (`mr62.dat`). You can change the missing code from 0 to an arbitrary numeric value, as described below.

In this example, we will use an extended pedigree file based on the previous one, with animal 9 added (`mr62.ped`).

The parameter file is also identical except for omitting an option for standard error calculations.

~~~~~{language=blupf90 caption="mr62.par"}
DATAFILE
mr62.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5 6
WEIGHT(S)

EFFECTS:
2 2 2 cross
1 1 9 cross
RANDOM_RESIDUAL VALUES
40.0 11.0
11.0 30.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr62.ped
(CO)VARIANCES
20.0 18.0
18.0 40.0
OPTION solv_method FSPAK
~~~~~

Solutions
---------

The results are identical to those in the textbook (pp. 107--108).

Remarks
-------

The default missing-phenotype code is `0` in the BLUPF90 programs, and you can change it to an arbitrary numeric value using an option in the parameter file. The following option changes the missing-observation code to $-999$ (you can use any numeric value).

~~~~~{language=blupf90}
OPTION missing -999
~~~~~

This option works for BLUPF90+, GIBBSF90+, and RENUMF90. With this option, an observation coded as `0` is no longer recognized as missing.

Note that the changed missing code applies only to _missing observations (phenotypes)_. Although the programs accept `0` as a general "missing" code in the data file, pedigree file, and effects in the parameter file, this `missing` option changes only the missing-phenotype code. The other "missing" codes remain `0`. A covariate with a value of 0 is still treated as a regular 0 value.

BLUPF90+ may accept a record entry with all observations missing without stopping. Although this may not cause a problem when solving the mixed model equations, it can cause problems in variance component estimation. When using REML or Gibbs sampling with BLUPF90+ or GIBBSF90+, the programs warn you to remove such records. Therefore, users should remove them before running BLUPF90+ or GIBBSF90+. RENUMF90 should detect such invalid records and issue a warning.

You may see extreme solutions or biased variance components if there are too many missing observations. Such results are not valid, and you should reconsider the model and data.
