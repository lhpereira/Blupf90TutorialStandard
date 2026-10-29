---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Multiple-trait model with unequal design matrix
===============================================

Model
-----

When each trait has a different model, the design matrices are unequal. Even in this case, the matrix notation of the model and the mixed model equations are the same as described previously.

BLUPF90+ can handle a multiple-trait model with unequal design matrices in two ways. Both approaches provide the same results, so the difference is only a matter of user preference [^1].

[^1]: Gibbs sampling programs support only one of the two approaches.

Files
-----

The data shown in the textbook on p. 109 are provided in the file `mr63.dat`. This file contains two different class effects, and each trait uses one of them.

1. Animal ID (Cow)
2. Sire ID
3. Dam ID
4. HYS 1 (for Fat yield 1)
5. HYS 2 (for Fat yield 2)
6. Fat yield 1
7. Fat yield 2

The pedigree (`mr63.ped`) is from the previous example.

First, we show a parameter file that handles the unequal design matrices in one way.

~~~~~{language=blupf90 caption="mr63.par"}
DATAFILE
mr63.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6 7
WEIGHT(S)

EFFECTS:
4 5  2 cross
1 1  8 cross
RANDOM_RESIDUAL VALUES
65.0 27.0
27.0 70.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr63.ped
(CO)VARIANCES
35.0 28.0
28.0 30.0
OPTION solv_method FSPAK
~~~~~

The first line in `EFFECTS:` refers to two different columns: column 4 for trait 1 and column 5 for trait 2. Thus, you can combine different effects in a single statement. You should specify the maximum number of levels among the effects listed in this statement. For example, in this case, the maximum level in column 4 is 2, and the maximum level in column 5 is also 2, so you can specify 2 as the number of levels for this statement.


Solutions
---------

This is identical to the example shown in the textbook on p. 110. Note that the first fixed effect (effect 1) differs by trait, although the solutions are stored in the same effect slot.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1        175.73126996
   2   1         1        243.23908674
   1   1         2        219.61329399
   2   1         2        240.54972647
   1   2         1          8.96915914
   2   2         1          8.84028863
   1   2         2         -2.99914279
   2   2         2         -2.77728027
   1   2         3         -5.97001636
   2   2         3         -6.06300835
   1   2         4         11.75424243
   2   2         4         11.65758757
   1   2         5        -16.25295661
   2   2         5        -15.82350798
   1   2         6        -17.31429689
   2   2         6        -15.71912600
   1   2         7          8.69047372
   2   2         7          8.13764492
   1   2         8         22.70213948
   2   2         8         20.93068834
~~~~~


Another parameter file
----------------------

BLUPF90 supports a different way to handle unequal design matrices. Consider the following parameter file, which uses the same data and pedigree files.

~~~~~{language=blupf90 caption="mr63a.par"}
DATAFILE
mr63.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
6 7
WEIGHT(S)

EFFECTS:
4 0 2 cross
0 5 2 cross
1 1 8 cross
RANDOM_RESIDUAL VALUES
65.0 27.0
27.0 70.0
RANDOM_GROUP
3
RANDOM_TYPE
add_animal
FILE
mr63.ped
(CO)VARIANCES
35.0 28.0
28.0 30.0
OPTION solv_method FSPAK
~~~~~

In this parameter file, the fixed effect for each trait is defined separately. The column number is 0 for a trait that does not need the effect. In this case, the first line describes the effect $HYS_1$, and only the first trait needs this effect (so you should put 0 for the second trait). Note that the Gibbs sampling program (GIBBSF90+) accepts only this style.

You can immediately see that the results are identical to the previous ones. Some effects ($HYS_1$ for trait 2 and $HYS_2$ for trait 1) are not estimated and have solutions of 0 because they are not defined in the parameter file.


Remarks
-------

The GitHub repository contains another parameter file, `mr63uni.par`, which performs two independent single-trait analyses for the two traits simultaneously. This trick works because all covariance components are set to 0. The results are comparable to the univariate results shown in the textbook (p. 110).

Note that this trick may not necessarily reduce computation time or computational resource requirements, although it generally works for small-scale analyses.
