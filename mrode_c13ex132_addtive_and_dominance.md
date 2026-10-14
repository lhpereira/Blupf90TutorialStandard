---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Total genetic merit: additive + dominance effects
=================================================

Model
-----

There is a unified method for predicting an animal's total genetic merit, defined as the sum of its additive and dominance effects. We can back-solve the additive and dominance-effect solutions from the total genetic merit and the relationship matrices. Please see the textbook for details.

BLUPF90+ does not natively support this model. Instead, the user should prepare all required relationship matrices and plug them into the equations. The user should also back-solve the solutions if needed.

Files
-----

We will use the same data file as before (`mr132.dat`). The user should calculate the inverse of the combined matrix, $\mathbf{M}^{-1} = (\mathbf{A}\sigma_a^2 + \mathbf{D}\sigma_d^2)^{-1}$, and store its elements in a text file (`mr132.apd`).

The parameter file is as follows.

~~~~~{language=blupf90 caption="mr132.par"}
DATAFILE
mr132.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
4  2 cross
1 12 cross
RANDOM_RESIDUAL VALUES
1
RANDOM_GROUP
2
RANDOM_TYPE
user_file
FILE
mr132.apd
(CO)VARIANCES
0.008333
OPTION solv_method FSPAK
~~~~~

Solutions
---------

You will obtain the same solutions as those in the textbook (p. 229). Again, BLUPF90+ does not back-solve the additive breeding values and dominance effects, so the user must perform this back-solving manually.
