---
title: Large-scale genetic evaluation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Approximation of accuracy and reliability
=========================================

Algorithm
---------

The accuracy or reliability of an animal's EBV can be calculated from the prediction-error variance (PEV), obtained from the diagonal elements of the inverse of the left-hand side (LHS) of the mixed-model equations. For smaller data sets, the diagonal elements can be calculated using sparse inversion with FSPAK or YAMS. As expected, the computational cost of calculating the inverse is very high, and the LHS must be stored in memory. Therefore, this direct method cannot be applied to large-scale genetic evaluations.

There are two major approaches for approximating accuracy and reliability. One approximates the diagonal elements of the LHS after absorbing the nongenetic effects. This element represents the effective number of records for an animal. The other approach focuses on the effective number of progeny (daughters), originally derived from selection-index theory based on a progeny test. This approach is especially useful for parents.

ACCF90 implements the first approach. The method was described by Strabel et al. (2001) and Sanchez et al. (2008), who extended the idea of Misztal and Wiggans (1988) to more complicated models. The algorithm is iterative because too many unknown variables are involved in the final equations to obtain the approximate diagonal elements.

A series of ACCF90GS programs, including ACCF90GS2 and ACCF90GS3, support ssGBLUP (Lourenco et al., 2026). The original algorithm was described by Misztal et al. (2013) using $\mathbf{G}^{-1}$ and $\mathbf{A}_{22}^{-1}$. Other researchers have developed alternative algorithms. These programs work well for smaller data sets. For larger problems, ACCF90GS2 and ACCF90GS3 use methods that do not explicitly compute $\mathbf{G}^{-1}$ and $\mathbf{A}_{22}^{-1}$.

This tutorial uses ACCF90 to demonstrate reliability approximation with a small data set.

Files
-----

We will use the following files for a four-trait repeatability model.

- [`simdata_rep.txt`](https://github.com/Masuday/data/blob/master/tutorial/simdata_rep.txt) : data file
- [`simped.txt`](https://github.com/Masuday/data/blob/master/tutorial/simped.txt) : pedigree file

The parameter file includes several options for ACCF90.

~~~~~{language=blupf90 caption="accparam1.txt"}
DATAFILE
simdata_rep.txt
NUMBER_OF_TRAITS
4
NUMBER_OF_EFFECTS
5
OBSERVATION(S)
9 10 11 12
WEIGHT(S)

EFFECTS: POSITIONS_IN_DATAFILE NUMBER_OF_LEVELS TYPE_OF_EFFECT [ EFFECT NESTED ]
6 6 6 6   155 cross
7 7 7 7     2 cross
8 8 8 8    11 cross
1 1 1 1  4641 cross
1 1 1 1  4641 cross
RANDOM_RESIDUAL VALUES
 45.124  22.357  18.626  13.762
 22.357  44.210  22.690  18.016
 18.626  22.690  46.101  22.795
 13.762  18.016  22.795  45.274
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
simped.txt
(CO)VARIANCES
 41.967  22.512  24.058  26.907
 22.512  17.489  19.738  24.257
 24.058  19.738  28.775  34.741
 26.907  24.257  34.741  56.668
RANDOM_GROUP
5
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
 15.546  15.237  10.490  1.4105
 15.237  40.937  19.562  6.4840
 10.490  19.562  27.782  4.5379
 1.4105  6.4840  4.5379  2.4410
OPTION cg   1 1 1 1
OPTION anim 4 4 4 4
OPTION pe   5 5 5 5
OPTION acc_maxrounds 20
~~~~~

The options are explained below. ACCF90 reads the solutions file. Run BLUPF90 (or BLUPF90IOD2) with the parameter file above and then run ACCF90 with the same parameter file. The program creates the file `sol_and_acc` as follows (the first nine lines are shown).

~~~~~{language=text caption="solutions"}
trait/effect level  solution acc
   1   4         1         -3.43278933  0.8728
   2   4         1          1.63205564  0.8147
   3   4         1          1.86332893  0.8510
   4   4         1          5.42050028  0.9237
   1   4         2          1.35147071  0.8754
   2   4         2          1.75742269  0.8140
   3   4         2          6.60794020  0.8512
   4   4         2          5.09637833  0.9268
   1   4         3         -9.07606792  0.8820
~~~~~

This file is similar to `solutions`, but it contains only the estimated breeding values (EBVs). The last column contains the reliability of each EBV.


Options
-------

~~~~~{language=blupf90}
OPTION cg   [ position(s)]  # contemporary group effect
OPTION anim [ position(s)]  # additive genetic effect
OPTION pe   [ position(s)]  # PE effect
OPTION mat  [ position(s)]  # maternal genetic effect
OPTION hs   [ position(s)]  # herd by sire interaction effect
~~~~~

Each option defines the position(s) of a specific effect in the `EFFECT:` block. The positions should be specified for all traits. If the effect is missing for a trait, put `0`. The first two options, `cg` and `anim`, are mandatory; the others are optional. In the example above, the contemporary group effect is the first effect in the `EFFECT:` block. Because it is considered for all four traits, we specify `cg 1 1 1 1`. Similarly, the additive genetic effect is the fourth effect for all traits, so we specify `anim 4 4 4 4`; the permanent environmental effect is the fifth effect for all traits, so we specify `pe 5 5 5 5`.

~~~~~{language=blupf90}
OPTION acc_maxrounds n      # n = integer number
OPTION conv_crit x          # x = real small number
~~~~~

The first option defines the maximum number of iterations (the default is 10), and the second defines the convergence criterion (the default is `1.0e-8`). The iterations stop when the program reaches either limit.

The program needs only a few iterations. Too many iterations may add bias to the approximate reliability. We recommend using the default convergence criterion. However, the default number of iterations may not be sufficient, so you should specify a larger value for `acc_maxrounds`.

~~~~~{language=blupf90}
OPTION type x   # x = 1.0 or 0.5
~~~~~

This option defines the type of reliability. The final value is calculated as $(R^2)^x$. The default is $x=1.0$, which corresponds to $R^2$ (that is, reliability). If you set $x=0.5$, the output is $(R^2)^{0.5} = R$, that is, accuracy. Traditionally, the beef industry has used accuracy ($x=0.5$), whereas the dairy industry has used reliability ($x=1.0$). The default value is 1.0.

~~~~~{language=blupf90}
OPTION parent_avg
~~~~~

With this option, the program calculates the parent average for each animal. The values are saved in an additional column in `sol_and_acc`.

~~~~~{language=blupf90}
OPTION original_id
~~~~~

This option adds the original ID (the tenth column in the renumbered pedigree file) to `sol_and_acc`.
