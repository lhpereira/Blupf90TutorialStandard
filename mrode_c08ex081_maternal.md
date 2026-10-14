---
title: Numerical examples from Mrode and Pocrnic (2014)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Animal model for maternal traits
================================

Model
-----

For some traits, especially those measured early in an animal's life, the phenotypes are largely affected by its dam. For example, weaning weight is expected to be associated with the dam's milk yield. The dam provides an environment in which her progeny grow (or lose) and develop their phenotypic values. The dam's contribution is assumed to consist of genetic and environmental factors. Thus, the animal model includes the dam's contribution, which has both genetic and nongenetic components.

In this example, the author uses a typical maternal model. In simple terms, the model has two correlated additive genetic effects: the direct genetic effect due to the animal's genes and the maternal genetic effect due to the dam's contribution. Assume the following model:
$$
\mathbf{y} = \mathbf{Xb} + \mathbf{Zu} + \mathbf{Wm} + \mathbf{Sp} + \mathbf{e}
$$
where $\mathbf{y}$ is a vector of observations, $\mathbf{b}$ is a vector of fixed effects, $\mathbf{u}$ is a vector of direct genetic effects, $\mathbf{m}$ is a vector of maternal genetic effects, $\mathbf{p}$ is a vector of maternal permanent-environmental effects, $\mathbf{e}$ is a vector of residual effects, and other matrices are incidence matrices. The covariance structure is typically assumed as
$$
\mathrm{var}
\left[
\begin{array}{l}
\mathbf{u}\\
\mathbf{m}\\
\mathbf{p}\\
\mathbf{e}
\end{array}
\right]
=
\left[
\begin{array}{llll}
\mathbf{A}\sigma_{d}^2&\mathbf{A}\sigma_{dm}&\mathbf{0}&\mathbf{0}\\
\mathbf{A}\sigma_{md}&\mathbf{A}\sigma_{m}^2&\mathbf{0}&\mathbf{0}\\
\mathbf{0}&\mathbf{0}&\mathbf{I}\sigma_{p}^2&\mathbf{0}\\
\mathbf{0}&\mathbf{0}&\mathbf{0}&\mathbf{I}\sigma_{e}^2\\
\end{array}
\right]
$$
where $\sigma_d^2$ is the direct genetic variance, $\sigma_m^2$ is the maternal genetic variance, $\sigma_{dm}$ is the covariance between direct and maternal genetic effects, $\sigma_p^2$ is the permanent environmental variance, and $\sigma_e^2$ is the residual variance. Note that this model contains a genetic covariance matrix ($\mathbf{G}_0$), even though it is a single-trait model. In this numerical example, the covariance matrix is
$$
\mathbf{G}_{0}
=
\left[
\begin{array}{ll}
\sigma_{d}^2&\sigma_{dm}\\
\sigma_{md}&\sigma_{m}^2
\end{array}
\right]
=
\left[
\begin{array}{ll}
150&-40\\
-40&90
\end{array}
\right]
$$
and $\sigma_p^2=40$ and $\sigma_e^2=350$.


Files
-----

The data set is shown on p. 138 (`mr81.dat`). The data available in the GitHub repository include a zero constraint on Herd 1, whose solution is 0.

   1. Animal ID (Calf)
   2. Sire ID
   3. Dam ID
   4. Herds
   5. Pen
   6. Birth weight (kg)

The pedigree is derived from the above data (`mr81.ped`).

The parameter file is as follows.

~~~~~{language=blupf90 caption="mr81.par"}
DATAFILE
mr81.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
5
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
 4  3 cross  # herd
 5  2 cross  # pen
 1 14 cross  # direct genetic effect
 3 14 cross  # maternal genetic effect
 3  7 cross  # maternal permanent effect
RANDOM_RESIDUAL VALUES
350.0
RANDOM_GROUP
3 4
RANDOM_TYPE
add_animal
FILE
mr81.ped
(CO)VARIANCES
150.0 -40.0
-40.0  90.0
RANDOM_GROUP
5
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
40.0
OPTION solv_method FSPAK
~~~~~

In this model, we consider five effects: herd, pen, direct genetic, maternal genetic, and maternal environmental effects. The first two effects are fixed. The third effect is the direct genetic effect, so the number of levels should be 14 (the number of animals found in the pedigree). The fourth effect is the maternal genetic effect; you can therefore use the position of the dam's ID. Note that the number of levels must be 14 because the maternal effect is correlated with the direct effect, which has 14 levels in the pedigree. In other words, each animal has both a direct breeding value and a maternal breeding value, and solutions are calculated for all 14 animals. The fifth effect is the maternal permanent environmental effect. This effect does not use the pedigree file; it is estimated only for dams with records in the data file. Therefore, the number of levels is only 7 because the maximum dam ID in the data is 7, although the position of this effect is the same as that of the previous
effect.[^2]

[^2]: In this case, even if you put 14 as the number of levels in the 5th effect, the program correctly works. The program can work when you put a larger number of levels than the actual number.


The direct genetic effect and the maternal genetic effect are correlated. These effects are listed together in `RANDOM_GROUP`, which contains `3 4`. This is still an animal model, so you can specify `add_animal` and the pedigree file. Although this is a single-trait model, it has two correlated genetic effects, so the covariance matrix ($\mathbf{G}_0$) should be a $2 \times 2$ matrix specified in `(CO)VARIANCES`.

We have another random maternal effect, which is an independent random effect (the maternal PE effect). You simply put 5 after `RANDOM_GROUP`. The type is `diagonal`. You do not need a pedigree file here. Only one variance component is required in `(CO)VARIANCES`.


Solutions
---------

You can confirm that the estimated direct and maternal breeding values are identical to those in the textbook (p. 140).

As noted in the textbook, these equations have a dependency among the fixed effects, so there are infinitely many solutions for the fixed effects. The above solutions are one such set. The provided files already impose a constraint on the fixed effect HYS. If you do not impose this constraint, you may obtain different solutions for the fixed effects, but they are also valid. The solutions for the random effects remain unchanged.
