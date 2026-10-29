---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Multiple-trait model with no environmental covariance
=====================================================

Model
-----

Assuming a 2-trait model, there is a situation where some animals are recorded only in one trait and the other animals are recorded only in another trait. In other words, no animals have observations for both traits. This situation also happens when evaluating genotype by environment (G by E) interactions. Typically, animals in the environment 1 do not have records in environment 2, and vice versa. In this case, the phenotype in each environment is treated as a "trait", and we should assume that the residual covariances among the "traits" are exactly zero.

The author assumes dual-purpose sires in cattle. The male and female calves are raised in different feeding systems. Male calves are recorded for yearling weight and females for fat yield. Given a two-trait setup, they assume that the genetic covariance is nonzero but the residual covariance is zero.

<!--
Note that the results from BLUPF90+ must be different from the values in the textbook.
-->

Files
-----

The data set is shown in the textbook at p.113. It is safer to omit animal 4 from the data file because it has no observations for all the traits, although the mixed-model solutions will be valid with animal 4 in the data file (as in `mr64.dat`).

1. Animal ID (Calf)
2. Sex (1=male and 2=female)
3. Sire ID
4. Dam ID
5. HYS
6. Yearling weight (kg)
7. Fat yield (kg)

The pedigree (`mr64.ped`) is from the above data set.

The parameter file should be the following.

~~~~~{language=blupf90 caption="mr64.ped"}
DATAFILE
mr64.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6 7
WEIGHT(S)

EFFECTS:
5 5 3 cross
1 1 17 cross
RANDOM_RESIDUAL VALUES
77.0  0.0
 0.0 70.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr64.ped
(CO)VARIANCES
43.0 18.0
18.0 30.0
OPTION solv_method FSPAK
~~~~~

Solutions
---------

The solutions are compatible with the values shown in the textbook (p.114).

Remarks
-------

If you estimate variance components using the model and data shown in this section, you will not obtain estimates of the residual covariances, even if you use nonzero residual covariances as the starting values. Using Gibbs sampling, you may obtain nonzero residual covariance estimates, but they should be close to zero.

In theory, the residual covariance between two traits is estimated from records with nonmissing observations for both traits. Therefore, you should not expect to obtain residual covariance estimates under the model shown here.
