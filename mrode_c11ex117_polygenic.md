---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Mixed linear models with polygenic effects
==========================================

Model
-----

See the textbook for details.

Files and solutions for SNP-BLUP
--------------------------------

First, we perform the SNP-BLUP analysis. The data file (`mr117a.dat`) is from the previous example, Example 11.1. We can use the same pedigree file as before (`mr117.ped`).

The parameter file contains 10 SNP effects and residual polygenic effects.

~~~~~{language=blupf90 caption="mr117a.par"}
DATAFILE
mr117a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
12
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
 4  1 cross   # general mean
 1 26 cross   # residual polygenic effects
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
RANDOM_GROUP  # polygenic effect
2
RANDOM_TYPE
add_animal
FILE
mr117.ped
(CO)VARIANCES
3.5241
RANDOM_GROUP  # jointly considering independent 10 SNP effects
3 4 5 6 7 8 9 10 11 12
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
8.9636 0 0 0 0 0 0 0 0 0
0 8.9636 0 0 0 0 0 0 0 0
0 0 8.9636 0 0 0 0 0 0 0
0 0 0 8.9636 0 0 0 0 0 0
0 0 0 0 8.9636 0 0 0 0 0
0 0 0 0 0 8.9636 0 0 0 0
0 0 0 0 0 0 8.9636 0 0 0
0 0 0 0 0 0 0 8.9636 0 0
0 0 0 0 0 0 0 0 8.9636 0
0 0 0 0 0 0 0 0 0 8.9636
OPTION solv_method FSPAK
~~~~~

You can check the solutions against the reference values in the textbook (pp. 199--200).

Files and solutions for GBLUP
-----------------------------

<!--
In this example with GBLUP, we use a text file for $\mathbf{G}_{w}^{-1}$ created with PREGSF90 in the previous section (the first approach). See the instruction and prepare the file.
-->

The author has already prepared a text file for $\mathbf{G}^{-1}$. The animal codes in the file are aligned with the original animal codes; therefore, you do not need to use any tricks with PREGSF90.

The data file (`mr117b.dat`) is the same as the previous one, except that the marker covariates have been removed. The pedigree file (`mr117.ped`) is the same as the one used for SNP-BLUP.

The parameter file is shown below.

~~~~~{language=blupf90 caption="mr117b.par"}
DATAFILE
mr117b.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
4  1 cross
1 26 cross
1 26 cross
RANDOM_RESIDUAL VALUES
245.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr117.ped
(CO)VARIANCES
3.5241
RANDOM_GROUP
3
RANDOM_TYPE
user_file
FILE
mr117b.Gi
(CO)VARIANCES
31.717
OPTION solv_method FSPAK
~~~~~

The solutions can be obtained by running BLUPF90+.
