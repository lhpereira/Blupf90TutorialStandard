---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Sire model
==========

Model
-----

Here, we apply a sire model to the data from the previous section. The mathematical model is the same as in the previous section, but the individual genetic effect is replaced with the sire genetic effect. In this model, we treat each observation as belonging to the sire of the animal rather than to the animal itself. Additive genetic variation is explained by sires. Therefore, the sire variance is $\sigma_s^2 = 0.25\sigma_a^2 = 5$, and the residual variance absorbs the remaining part of the genetic variance: $\sigma_e^{2*}=0.75\sigma_a^2+\sigma_e^2=55$. The phenotypic variance is the same (60) in the animal and sire models.

Files
-----

The files in the GitHub repository are all valid but slightly simplified. Here, we will use more straightforward modeling.

We can use the same data set as in the previous section (`mr42a.dat`). In this case, we use the 3rd column (sire ID) instead of the 1st column (animal ID) as the sire effect. This is a direct transcription from the textbook.

~~~~~{language=text caption="mr42a.dat"}
 4 1 1 0 4.5
 5 2 3 2 2.9
 6 2 1 2 3.9
 7 1 4 5 3.5
 8 1 3 6 5.0
~~~~~

The pedigree file used for the sire model in BLUPF90 is different from the one used in the animal model. This file (`mr42a.ped`) contains 3 columns:

1. The ID for sire.
2. The ID of the sire's sire.
3. The ID of the maternal grandsire (MGS) of the sire.

~~~~~{language=text caption="mr42a.ped"}
 1 0 0
 3 0 0
 4 1 0
~~~~~

In this case, all maternal grandsires are unknown, so the 3rd column should be 0 for all animals.

The parameter file is also modified. We add comments indicating the lines that must be changed.

~~~~~{language=blupf90 caption="mr42a.par"}
DATAFILE
mr42a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
2 2 cross
3 4 cross               # 3rd column = sire effect; 4 sires
RANDOM_RESIDUAL VALUES  # residual variance
55.0
RANDOM_GROUP
2
RANDOM_TYPE             # type changed
add_sire
FILE
mr42a.ped
(CO)VARIANCES           # sire variance
5.0
OPTION solv_method FSPAK
~~~~~

The value `add_sire` is a keyword for the sire model in BLUPF90. The pedigree file does not include sire 2 because it is missing, but the program handles this without issue.

Solutions
---------

The solutions are as follows.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1          4.33567107
   1   1         2          3.38198579
   1   2         1          0.02200220
   1   2         2          0.00000000
   1   2         3          0.01402640
   1   2         4         -0.04304180
~~~~~

The solution for sire ID 2 is 0.0. BLUPF90+ always produces a solution of 0.0 for a level missing from the data and pedigree files. Otherwise, the solutions are identical to those in the textbook (p. 75).

Note that the files in the GitHub repository (`mr42.dat`, `mr42.ped`, and `mr42.par`) omit ID 2 and renumber the IDs, so they produce the same solutions with different level coding.
