---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Common environmental effects as random
======================================

Model
-----

In this example, the author considers a simple maternal model in which the maternal effects ($c$) are not correlated with
one another. The variance components include the animal genetic variance $\sigma_a^2 = 20$, the maternal variance $\sigma_c^2 = 15$, and the residual variance $\sigma_e^2 = 65$.

Files
-----

The data file (`mr52.dat`) contains the complete table shown in the textbook (p. 95). The columns are as follows.

1. Animal ID (piglet)
2. Sire ID
3. Dam ID
4. Sex (1 = male and 2 = female)
5. Weaning weight (kg)

The pedigree (`mr52.ped`) can be derived from the above data.

The parameter file should contain two random effects.

~~~~~{language=blupf90 caption="mr52.par"}
DATAFILE
mr52.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS: POSITIONS_IN_DATAFILE NUMBER_OF_LEVELS TYPE_OF_EFFECT  [EFFECT NESTED]
4  2 cross
1 15 cross
3  5 cross    # for maternal environmental effect
RANDOM_RESIDUAL VALUES
65.0
RANDOM_GROUP  # first random effect
2
RANDOM_TYPE
add_animal
FILE
mr52.ped
(CO)VARIANCES
20.0
RANDOM_GROUP  # second random effect
3
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
15.0
OPTION solv_method FSPAK
~~~~~

Solutions
---------

The results are identical to those in the textbook.
