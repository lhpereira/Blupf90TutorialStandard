---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Fixed regression model
======================

Model
-----

When repeated measurements are considered as the same traits over time, a repeatability model can be suitable to apply. The assumption in this situation is that the genetic correlation between observations measured at any two time-points is 1. Even if the assumption is correct, the actual observations would have an average curve by a group over time. This non-genetic, systematic change can be modeled by fixed regression curves (or distinct fixed class-effect defined over time).

The author shows the test day measurements of fat yield in dairy cattle. A repeatability model with a fixed regression curve is assumed. See the textbook for details for the details. The variance components are $\sigma_u^2 =5.521$, $\sigma_p^2 = 8.470$, and $\sigma_e^2 = 3.710$.

Usually, fixed regressions are nested within an environmental class. It models the fact that an average trajectory can be differentiated depending on the specific environment nested within, for instance, herd, region, season, age, parity, and so on. This example is small, so the nested fixed regressions are not considered.


Files
-----

The author shows the data set (`mr101.dat`). BLUPF90+ does not prepare the coefficients of the Legendre polynomials automatically. And therefore, The coefficients should be prepared as covariates by the user and saved in the data file. The following is the detail of this data.

~~~~~{language=text caption="mr101.dat"}
 4   4  1 17.0  0.7071 -1.2247  1.5811 -1.8708  2.1213
 4  38  2 18.6  0.7071 -0.9526  0.6442 -0.0180 -0.6205
 4  72  3 24.0  0.7071 -0.6804 -0.0586  0.7571 -0.7757
...
~~~~~

1.  Animal ID (cow)
2.  Days in milk (DIM)
3.  Herd-test-day class
4.  Test day fat yield
5.  0th order Legendre covariates $\phi_0$ (intercept)
6.  1st order Legendre covatiates $\phi_1$
7.  2nd order Legendre covariates $\phi_2$
8.  3rd order Legendre covariates $\phi_3$
9.  4th order Legendre covariates $\phi_4$


The pedigree is the same as Example 4.1 (`mr101.ped`).

The parameter file is shown below.

~~~~~{language=blupf90 caption="mr101.par"}
DATAFILE
mr101.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
8
OBSERVATION(S)
4
WEIGHT(S)

EFFECTS:
3 10 cross
5  1 cov
6  1 cov
7  1 cov
8  1 cov
9  1 cov
1  8 cross
1  8 cross
RANDOM_RESIDUAL VALUES
3.710
RANDOM_GROUP
7
RANDOM_TYPE
add_animal
FILE
mr101.ped
(CO)VARIANCES
5.521
RANDOM_GROUP
8
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
8.470
OPTION solv_method FSPAK
~~~~~

Solutions
---------

The equations are linearly dependent, so the solutions for the fixed effects are not unique, although the current solutions match those in the textbook. The solutions for the random effects agree closely with the textbook values (p. 162).
