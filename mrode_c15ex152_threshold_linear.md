---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Joint analysis of quantitative and binary traits
================================================

Model
-----

We can consider a two-trait model with one threshold trait and one continuous trait. We have three options: Gibbs sampling (GIBBSF90+), a maximum a posteriori approach (the older CBLUP90THR), and a general multivariate threshold-linear framework using iterative methods.

When we use Gibbs sampling to draw from the posterior distribution of a location parameter, we can obtain a solution as the posterior mean. However, when we try to calculate the solutions without Gibbs sampling, the resulting equations become nonlinear, and a complicated strategy is needed to solve them. See Wang et al. (1997) for a detailed discussion. Recently, the general multivariate framework was derived and implemented (Bermann et al., 2026). This approach requires less computing time than Gibbs sampling for a large-scale data set.

For the threshold trait, the solutions obtained by any of these methods should be converted differently from those in the regular threshold model. The author cites Foulley et al. (1983) and explains how to perform this conversion.

In this section, we will simply use GIBBSF90+ to obtain the posterior mean of a location parameter. In this example, we have only one threshold, so the program fixes the threshold at 0 and the residual variance at 1.0. Then, we use BLUPF90+ with the Newton-Raphson method to calculate the solutions. 


Files and solutions
-------------------

### Common files

The data file is from Foulley et al. (1983) and shown below (`mr152.dat`).

~~~~~{language=text caption="mr152.dat"}
1 1 1 1 41.0 1
1 1 1 1 37.5 1
1 1 1 2 41.5 1
1 1 2 2 40.0 1
1 1 2 2 43.0 1
1 1 2 2 42.0 1
1 1 2 2 35.0 1
2 1 1 2 46.0 1
2 1 1 2 40.5 1
2 1 2 2 39.0 1
1 2 1 1 41.4 1
1 2 1 1 43.0 2
1 2 2 2 34.0 1
1 2 2 1 47.0 2
1 2 2 1 42.0 1
2 2 2 1 44.5 1
2 2 2 1 49.0 1
1 3 1 1 41.6 1
2 3 1 1 36.0 1
2 3 1 2 42.7 1
2 3 2 2 32.5 1
2 3 2 2 44.4 1
2 3 2 1 46.0 1
1 4 2 1 47.0 2
1 4 2 2 51.0 2
1 4 2 2 39.0 1
2 4 1 1 44.5 1
1 5 1 1 40.5 1
1 5 1 2 43.5 1
1 5 2 1 42.5 1
1 5 2 1 48.8 2
1 5 2 1 38.5 1
1 5 2 1 52.0 1
1 5 2 2 48.0 1
2 5 1 2 41.0 1
2 5 1 1 50.5 2
2 5 2 1 43.7 2
2 5 2 1 51.0 2
1 6 1 2 51.6 2
1 6 1 1 45.3 2
1 6 1 2 36.5 1
1 6 2 1 50.5 1
1 6 2 1 46.0 2
1 6 2 1 45.0 1
1 6 2 2 36.0 1
2 6 1 2 43.5 1
2 6 1 2 36.5 1
~~~~~

This file contains six columns.

1. Heifer origin
2. Sire ID
3. Season
4. Sex (1 = male and 2 = female)
5. BW (body weight)
6. CD (calving difficulty)

The pedigree file contains the sire, the sire of the sire, and the maternal grandsire.

~~~~~{language=text caption="mr152.ped"}
1 0 0
2 0 0
3 1 0
4 2 1
5 3 2
6 2 3
~~~~~

### GIBBSF90+: Gibbs sampling

As the author mentions, we should use the corrected genetic parameters ($\mathbf{G}_{c}$) shown on p. 278. The residual variance is 1.0 for CD and 20.0 for BW, and the residual covariance is 2.0527 because the residual correlation is 0.459. The parameter file is as follows. In this example, we reverse the order of the traits: trait 1 is CD (categorical trait) and trait 2 is BW.

~~~~~{language=blupf90 caption="mr152gs.par"}
DATAFILE
mr152.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
6 5
WEIGHT(S)

EFFECTS:      # l=H+M+E+s+e
1 1 2 cross   # origin
3 3 2 cross   # season
4 4 2 cross   # sex
2 2 6 cross   # sire
RANDOM_RESIDUAL VALUES
1.0000 2.0527
2.0527 20.000
RANDOM_GROUP
4
RANDOM_TYPE
add_sire
FILE
mr152.ped
(CO)VARIANCES
0.0300 0.0302
0.0302 0.7178
OPTION cat 2 0
OPTION fixed_var mean
~~~~~

Run GIBBSF90+ to draw 20,000 samples, saving a sample every 10 steps, and discard the first 10,000 samples as burn-in. The resulting solutions are shown below (in `final_solutions`).

~~~~~{language=text caption="final_solutions"}
trait/effect level  solution        SD
   1   1         1         -1.48878236          0.51632502
   2   1         1         41.59189006          1.36969324
   1   1         2         -1.77677982          0.62896928
   2   1         2         42.19413470          1.55335679
   1   2         1         -0.00914886          0.46477468
   2   2         1         -1.25154241          1.40166069
   1   2         2          0.00000000          0.00000000
   2   2         2          0.00000000          0.00000000
   1   3         1          1.18116775          0.51463927
   2   3         1          3.21833655          1.33080496
   1   3         2          0.00000000          0.00000000
   2   3         2          0.00000000          0.00000000
   1   4         1         -0.07367886          0.17080848
   2   4         1         -0.26275069          0.75696107
   1   4         2          0.04505988          0.17065197
   2   4         2          0.09935381          0.79633270
   1   4         3         -0.04894424          0.16757123
   2   4         3         -0.19468959          0.80784114
   1   4         4          0.04338392          0.17223417
   2   4         4          0.16514913          0.80657896
   1   4         5          0.02287885          0.17036015
   2   4         5          0.32991823          0.78445782
   1   4         6          0.03347718          0.17422194
   2   4         6          0.16621547          0.79187824
~~~~~

The estimated values are close to the reference values (p. 279). The small difference is due to the limited number of observations. If you impose zero constraints on the fixed effects, you may obtain values more similar to those in the textbook.


### BLUPF90+: Newton-Raphson method

Next, we use BLUPF90+ to calculate the solutions given variance components using the Newton-Raphson method. This method does not estimate the thresholds for traits with two or three categories. Instead, the user must provide fixed thresholds as parameters. In this example, two arbitrary thresholds are required. The solutions depend on the specified thresholds and may therefore be inaccurate if the thresholds are inappropriate.

This method involves two types of iteration: Newton-Raphson iterations and iterative solutions of the mixed model equations. At the beginning of each Newton-Raphson iteration, BLUPF90+ solves the mixed model equations using an iterative method (preconditioned conjugate gradient by default). The current solutions are then used to modify the equations for the next Newton-Raphson iteration. The algorithm stops when the current solutions are sufficiently close to those from the previous iteration.

The parameter file is similar to the one used for Gibbs sampling, except for the options. The following parameter file is used in this example.

~~~~~{language=blupf90 caption="mr152nr.par"}
DATAFILE
mr152.dat
NUMBER_OF_TRAITS
2
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
6 5
WEIGHT(S)

EFFECTS:      # l=H+M+E+s+e
1 1 2 cross   # origin
3 3 2 cross   # season
4 4 2 cross   # sex
2 2 6 cross   # sire
RANDOM_RESIDUAL VALUES
1.0000 2.0527
2.0527 20.000
RANDOM_GROUP
4
RANDOM_TYPE
add_sire
FILE
mr152.ped
(CO)VARIANCES
0.0300 0.0302
0.0302 0.7178
OPTION category 2
OPTION threshold_model nr
OPTION constant_thresholds 0.0 1.0
~~~~~

When using the linear-threshold model, all threshold traits must precede continuous traits in the `OBSERVATION(S)` line. In this case, position 6 corresponds to the threshold trait, and position 5 corresponds to the continuous trait. Therefore, you should put `6` first, followed by `5`. This rule applies regardless of the number of traits. If there are no continuous traits, you can simply list the threshold traits.

There are three options in this parameter file. Their meanings are as follows.

- `OPTION category 2`: This option specifies the number of categories for each threshold trait. In this case, there is one threshold trait with two categories, so you should specify `2`. Note that you do not need to specify `0` for continuous traits, unlike with `OPTION cat` for Gibbs sampling.
- `OPTION threshold_model nr`: This option is required to use the Newton-Raphson method.
- `OPTION constant_thresholds 0.0 1.0`: This option specifies fixed thresholds. In this case, the model assumes three categories with two thresholds. The values `0.0` and `1.0` are specified as the thresholds.

You will get the following results.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1         -1.37749171
   2   1         1         41.61100467
   1   1         2         -1.63228452
   2   1         2         42.18064998
   1   2         1         -0.00926256
   2   2         1         -1.24380536
   1   2         2          0.00000000
   2   2         2          0.00000000
   1   3         1          1.08135218
   2   3         1          3.20345953
   1   3         2          0.00000000
   2   3         2          0.00000000
   1   4         1         -0.06013922
   2   4         1         -0.26993705
   1   4         2          0.03870734
   2   4         2          0.06899079
   1   4         3         -0.04992779
   2   4         3         -0.21577141
   1   4         4          0.04183432
   2   4         4          0.13756829
   1   4         5          0.01720032
   2   4         5          0.28430884
   1   4         6          0.03221148
   2   4         6          0.13579249
~~~~~

The values are very similar to the reference solutions in the textbook (p. 279), although they differ slightly from those obtained using Gibbs sampling.

### CBLUP90THR: Comparison with the old software

For comparison, we show the results from CBLUP90THR using the same files as above. Please note that the latest binary of CBLUP90THR available from the official website does not work for this example. The following results were obtained using an older version of the software. The estimated threshold is $0.533$ after 100 rounds.

The results are very close to those obtained using the Newton-Raphson method.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
  1  1       1     -0.8440
  2  1       1     41.6110
  1  1       2     -1.0988
  2  1       2     42.1806
  1  2       1     -0.0093
  2  2       1     -1.2438
  1  2       2      0.0000
  2  2       2      0.0000
  1  3       1      1.0814
  2  3       1      3.2035
  1  3       2      0.0000
  2  3       2      0.0000
  1  4       1     -0.0601
  2  4       1     -0.2699
  1  4       2      0.0387
  2  4       2      0.0690
  1  4       3     -0.0499
  2  4       3     -0.2158
  1  4       4      0.0418
  2  4       4      0.1376
  1  4       5      0.0172
  2  4       5      0.2843
  1  4       6      0.0322
  2  4       6      0.1358
~~~~~
