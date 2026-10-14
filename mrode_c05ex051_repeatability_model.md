---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Repeatability model
===================

Model
-----

A repeatability model is an animal model applicable when an animal has more than one observation. In such a case, we can split the residual effect into two: a random "temporary" environmental (TE) effect and a permanent environmental (PE) effect associated with the animal. Thus, the model contains two kinds of animal effects (additive genetic and permanent environmental effects), both of which are treated as random effects.

The author uses a repeatability model in Example 5.1 for fat yield in dairy cattle. The model has fixed effects for parity and Herd-Year-Season, an additive genetic effect for the animal, a permanent environmental effect for the animal, and a random residual effect (including the TE effects). The author assumes that the genetic variance is 20, the residual variance is 28, and the permanent-environmental variance is 12.

Files
-----

The data file (`mr51.dat`) contains only the table shown on p. 91 in the textbook. This file has 6 columns. Columns 2 and 3 are not actually used in this analysis.

1. Animal ID (cow)
2. Sire ID
3. Dam ID
4. Parity (1 or 2)
5. HYS (from 1 to 4)
6. Fat yield (kg)

The pedigree file (`mr51.ped`) can be prepared as a usual 3-column file.

This model contains two random effects in addition to the random error. The permanent environmental effects are not related to one another, so you should use `diagonal` in the `RANDOM_TYPE` keyword. The position of the permanent environmental effects is, in this case, the same as that of the additive genetic effects because both are individual animal effects; the only difference is the covariance structure. The parameter file (`mr51.par`) is as follows.

~~~~~{language=blupf90 caption="mr51.par"}
DATAFILE
mr51.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
5 4 cross
4 2 cross
1 8 cross  # for additive genetic effect
1 8 cross  # for permanent environmental effect
RANDOM_RESIDUAL VALUES
28.0
RANDOM_GROUP   # additive genetic effect
3
RANDOM_TYPE
add_animal
FILE
mr51.ped
(CO)VARIANCES
20.0
RANDOM_GROUP  # permanent environmental effect
4
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
12.0
OPTION solv_method FSPAK
~~~~~

Solutions
---------

BLUPF90+ calculates the solutions (not shown here) with the above parameter file. The left-hand side of the mixed model equations is not full rank, and two fixed-effect solutions are set to 0. The positions of the zero constraints differ from those in the textbook (p. 92; parity 1 and 2 here versus HYS 1 and 3 in the textbook). As before, a linear contrast is the same in both cases. The BLUPs for the random effects are the same as those in the textbook. The PE solutions for animals 1 to 3 are 0 (the expected value for a random effect with no information) because they have no observations. In this case, effect code 1 is shared by the additive genetic and permanent environmental effects.

As shown in the PE solutions, BLUPF90+ accepts missing levels and animals in the data and pedigree files. The number of levels specified in the parameter file does not necessarily have to equal the actual number of levels; it should instead be equal to the maximum level or animal code in the files. Therefore, as long as it is equal to or greater than the maximum code in the files, BLUPF90+ runs without errors and provides valid solutions, with zeros for missing levels. For example, in the above parameter file, you can replace `8` with a larger number, such as `100`, for the additive (the 3rd effect) and/or PE (the 4th effect), and BLUPF90+ will still run successfully. Of course, such a redundant specification is inefficient in terms of memory but may provide flexibility when constructing the parameter file.


Putting arbitrary zero-constraints manually
-------------------------------------------

There is a trick for manually imposing user-defined zero constraints on the mixed model equations. You can replace the target effect code in the data file with 0. This technique was introduced in the previous section for unknown parent groups. The alternative data file (`mr51a.dat`) contains the constraint for the HYS effect (in the 5th column), as shown below.

~~~~~{language=text caption="mr51a.dat"}
4 1 2 1 0 201
4 1 2 2 0 280
5 3 2 1 0 150
5 3 2 2 4 200
6 1 5 1 2 160
6 1 5 2 0 190
7 3 4 1 0 180
7 3 4 2 0 250
8 1 7 1 2 285
8 1 7 2 4 300
~~~~~

The 5th column in the above file has 0 for HYS 1 and 3, corresponding to the textbook's application of the zero constraint to HYS 1 and 3. Then, rewrite the parameter file to read the new data file by changing only the `DATAFILE` block. When you run BLUPF90+ with the modified parameter file, you will see different solutions.

Please be careful when using such a manual constraint. You may accidentally impose the constraint incorrectly. Also, if you impose too many constraints on the equations, the solutions will make no sense.
