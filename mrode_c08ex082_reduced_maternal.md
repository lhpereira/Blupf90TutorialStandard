---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Reduced animal model with maternal effects
==========================================

Model
-----

The reduced animal model can be extended to have the maternal effects. We do not explain the details of this model here. Please read the textbook for detail.

BLUPF90+ does not support the reduced animal model directly. However, there is a trick to construct the mixed model equations in BLUPF90+ as explained in the section of reduced animal model. Here, we demonstrate how BLUPF90+ performs the reduced animal model with maternal effects.

Files
-----

We will apply the same model as described in the previous section. However, we have to prepare the data for the reduced animal model. The terminal animals will be removed from the pedigree. The actual data and pedigree files (`mr82.dat` and `mr82.ped`) are available through the GitHub repository.

The parameter file is as follows.

~~~~~{language=blupf90 caption="mr82.par"}
DATAFILE
mr82.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
7
OBSERVATION(S)
6
WEIGHT(S)
10
EFFECTS:
4 3 cross
5 2 cross
8 0 cov 2
8 0 cov 3
9 9 cov 7
3 9 cross
3 7 cross
RANDOM_RESIDUAL VALUES
350.0
RANDOM_GROUP
5 6
RANDOM_TYPE
add_animal
FILE
mr82.ped
(CO)VARIANCES
150.0 -40.0
-40.0  90.0
RANDOM_GROUP
7
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
40.0
OPTION solv_method FSPAK
~~~~~

The trick is in the EFFECT block, where some effect has 0 levels. The meaning of this trick has been explained in the section regarding the reduced animal model. With the maternal genetic effects, we should make a group including the animal (direct) and maternal effects (`RANDOM_GROUP` as `5 6`).

Solutions
---------

The results are the same as ones from the normal direct-maternal model. We can back-solve the estimated breeding values for animals excluded from the pedigree based on the solutions for their parents, as explained in the textbook (p.146).
