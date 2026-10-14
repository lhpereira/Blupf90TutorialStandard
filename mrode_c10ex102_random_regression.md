---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Random regression model
=======================

Model
-----

Assume longitudinal data measured over time. If the genetic correlation between observations taken at two time points is lower than 1, the repeatability model is inappropriate. In this case, a multiple-trait model, which treats observations measured in different time periods as separate traits, may be more appropriate. If the time interval is shorter, we have more traits. As the interval approaches 0, we ultimately have an infinite number of traits, and genetic correlations can be described using a function of two time points. Random environmental effects can have the same covariance structure. This is the basis of the random-regression model.

In this example, we fit second-order Legendre polynomials (3 coefficients, including the intercept) for both additive genetic and permanent environmental effects. In this model, each animal has 3 random coefficients for the additive genetic effect and 3 additional coefficients for the permanent environmental effect. See the textbook for details of the model.

The random-regression coefficients are correlated. This resembles a maternal model with a direct and maternal genetic covariance matrix. In this model, we have 3 random-regression coefficients for each random effect. Thus, the variance components are $3 \times 3$ matrices for the additive genetic ($\mathbf{G}_0$) and permanent environmental ($\mathbf{P}_0$) effects. The covariance matrices are
$$
\mathbf{G}_{0}
=
\left[
\begin{array}{rrr}
3.297&  0.594& -1.381\\
0.594&  0.921& -0.289\\
-1.381& -0.289& 1.005
\end{array}
\right]
\quad
\text{and}
\quad
\mathbf{P}_{0}
=
\left[
\begin{array}{rrr}
 6.872& -0.254& -1.101\\
-0.254&  3.171&  0.167\\
-1.101&  0.167&  2.457
\end{array}
\right]
$$
and $\sigma_e^2=3.710$.


Files
-----

We use the same data and pedigree files as in the previous section (with separate files in the GitHub repository, `mr102.dat` and `mr102.ped`).

The parameter file is an extension to the previous one.

~~~~~{language=blupf90 caption="mr102.par"}
DATAFILE
mr102.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
12
OBSERVATION(S)
4
WEIGHT(S)

EFFECTS:
 3 10 cross  # HTD
 5  1 cov    # Legendre polynomials (intercept) for fixed regression
 6  1 cov    # Legendre polynomials (1st order) for fixed regression
 7  1 cov    # Legendre polynomials (2nd order) for fixed regression
 8  1 cov    # Legendre polynomials (3rd order) for fixed regression
 9  1 cov    # Legendre polynomials (4th order) for fixed regression
 5  8 cov 1  # Legendre polynomials (intercept) for additive genetic effect
 6  8 cov 1  # Legendre polynomials (1st order) for additive genetic effect
 7  8 cov 1  # Legendre polynomials (2nd order) for additive genetic effect
 5  8 cov 1  # Legendre polynomials (intercept) for permanent environmental effect
 6  8 cov 1  # Legendre polynomials (1st order) for permanent environmental effect
 7  8 cov 1  # Legendre polynomials (2nd order) for permanent environmental effect
RANDOM_RESIDUAL VALUES
3.710
RANDOM_GROUP
7 8 9
RANDOM_TYPE
add_animal
FILE
mr102.ped
(CO)VARIANCES
 3.297  0.594 -1.381
 0.594  0.921 -0.289
-1.381 -0.289  1.005
RANDOM_GROUP
10 11 12
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
 6.872 -0.254 -1.101
-0.254  3.171  0.167
-1.101  0.167  2.457
OPTION solv_method FSPAK
# additional options for the random regression model
OPTION store_pev_pec 7 8 9
OPTION include_effects 7 8 9
OPTION minmaxrr 4 310
OPTION cumulative 6 310
OPTION include_pev
OPTION acctype rel
~~~~~

The additive genetic effect is represented by effects 7, 8, and 9, and the permanent environmental effect is represented by effects 10, 11, and 12. Both have similar definitions, so we explain only the additive effect. Now we consider effect 7:

    5    8 cov 1

This refers to column 5 as the intercept covariate for the regression. Because we have 8 animals in total, we estimate 8 intercepts. The above statement means that column 5 is treated as a covariate and the regression is nested within the animal (column 1). The remaining two statements can be interpreted similarly. The corresponding `RANDOM_GROUP` statement contains 3 effects: 7, 8, and 9, so we need a $3 \times 3$ genetic covariance matrix.


Solutions
---------

The equations are not full rank, so there are infinitely many solutions for the fixed effects. The solutions for the random effects are almost the same as those in the textbook.

The additive genetic random regressions are defined as effects 7, 8, and 9. For these effects, the number of levels is the maximum animal ID. In a random-regression model, estimated breeding values for an animal are represented by regression coefficients. In this case, each animal has 3 coefficients. For example, the breeding values for animal 4 are
$$
\mathbf{u}_4
=
\left[
\begin{array}{r}
0.34456493\\
0.00628267\\
-0.31641305
\end{array}
\right].
$$


Additional options
------------------

The parameter file shown above has extra options. Those options are not mandatory to perform the random regression model; rather they may help to obtain more information or accumulated breeding values with an aggregated Legendre polynomial covariates.

BLUPF90+ takes `OPTION store_pev_pec`, which calculates the prediction error covariance (PEC) matrix for a set of random regression coefficients for an animal. In this case, this option takes 3 numbers, `7 8 9`, corresponding to the `RANDOM_GROUP` for additive random-regression coefficients. BLUPF90+ calculates the PEC matrix (3 x 3 in this case) for each animal and store them to a file, `pev_pec_bf90`. These matrices can be useful to calculate more precise reliability of an aggregated breeding value (like a breeding value for 305-d yield).

A separate software, RRMEBVF90, can help computing the aggregated breeding value and its reliability based on the output of BLUPF90+ for a random regression model. The extra options in the parameter file shown above are used by RRMEBVF90. A blief explanation of the options is available below. Note that, in the below description, we focus on additive-genetic random regressions, but you can specify any random regression coefficients.

- `OPTION include_effects 7 8 9`: the positions of random regression coefficients to compute the aggregated value and its reliability.
- `OPTION minmaxrr 4 310`: the range of the original time scale for the Legendre polynomials (from day 4 to 310 as in the textbook).
- `OPTION cumulative 6 310`: the range for accumulated breeding values (from day 6 to 310 as in the textbook).
- `OPTION include_pev`: using `pev_pec_bf90` to compute the reliability/accuracy of the aggregated breeding value.
- `OPTION acctype rel`: realiability computation (`rel` for reliability and `acc` for accuracy) for the aggregated breeding values.

After running `blupf90+`, you can run `rrmebvf90` with the same parameter file. It produces a file, `solutions_cumulative`, which contains the aggregated breeding value and its reliability, as specified in the parameter file as follows.

~~~~~{language=text caption="solutions"}
         1         7         1         -12.37307706   0.0930
         1         7         2         -15.73278435   0.0375
         1         7         3          28.10586142   0.0664
         1         7         4          74.81569875   0.1167
         1         7         5         -98.41487528   0.1497
         1         7         6        -118.42716740   0.0586
         1         7         7         184.16805426   0.1039
         1         7         8          47.68897543   0.0485
~~~~~

The format is compatible with the regular `solutions` without the header line. The second right column is the aggregated breeding value and the right most column is the reliability. The reliability or accuracy is stored only if `OPTION acctype` is specified. The aggregated breeding values are almost identical to the values shown in the textbook (p.167). In addition, the reliability is identical to ones in the textbook (p.174).

The detailed usage of RRMEBVF90 is available in the official BLUPF90 manual and wiki pages.
