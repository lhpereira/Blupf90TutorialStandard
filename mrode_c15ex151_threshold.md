---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

The threshold model
===================

Model
-----

Some traits show categorical responses that can potentially be ordered by degree, such as calving difficulty (easy, slightly difficult, or difficult) and disease status (no treatment needed, treated, or died). For such traits, although the phenotype is not continuous, we can still assume that both genetic and environmental factors affect the trait and that a polygenic effect is present. In this case, it is reasonable to assume that each animal has an unobservable, hypothetical random variable, analogous to the phenotype for a continuous trait, that contributes to the categorical response. Various factors can contribute to this underlying variable. When the variable crosses a certain threshold, the animal falls into a different response category. This is the basic idea of the threshold model: the variable is called the liability, and the boundary is called the threshold. The liability is usually assumed to be normally distributed on an underlying scale. We can apply a linear model to this underlying variable. In this framework, the number of thresholds is equal to the number of categories minus 1.

The author cites a well-known data set originally provided by Gianola and Foulley (1983). The trait is calving ease with 3 categories. The sire model for liability is as follows.
$$
l_{ijkl} = H_i +C_j + s_k + e_{ijkl}
$$
where $l_{ijkl}$ is the liability, $H_i$ and $C_j$ are fixed effects, $s_k$ is the random sire effect, and $e_{ijkl}$ is the residual. The variance components are $\sigma_s^2 = 1/19 = 0.0526$ and $\sigma_e^2 = 1.0$.

Both BLUPF90+ and GIBBSF90+ now support the full threshold model, with any number of threshold traits correlated with any number of continuous, normally distributed traits (Bermann et al., 2026). We will use both programs here. Note that the BLUPF90 family previously had several programs for handling threshold traits, and the Gibbs-sampling programs officially supported the threshold model.

Common files
------------

The pedigree file (`mr151.ped`) is prepared for the sire model.

~~~~~{language=text caption="mr151.ped"}
 1 0 0
 2 0 0
 3 1 0
 4 3 0
~~~~~

The items are different from the animal model.

1. Animal (sire) ID
2. Sire (sire of sire) ID
3. Maternal grand sire ID

The data file is shown below. All phenotypes should be written in the flat file. There are 28 observations from 28 calves (p. 266). Thus, the data file should contain one observation per record.

<!--
The data file in the GitHub repository has constraints on the both fixed effects (`mr151.dat`).  
-->

~~~~~{language=text caption="mr151a.txt"}
1 1 1 1   1
1 2 1 1   2
1 1 1 1   3
1 2 2 2   4
1 1 2 1   5
1 1 2 3   5
1 2 2 1   6
1 2 2 1   6
1 2 2 1   6
1 1 3 1   7
1 1 3 2   7
1 2 3 2   8
1 1 3 1   9
2 2 1 1  10
2 2 1 1  10
2 1 1 1  11
2 1 1 3  12
2 2 2 1  13
2 2 2 3  13
2 1 2 1  14
2 2 3 2  15
2 1 3 3  16
2 1 4 2  17
2 2 4 1  18
2 2 4 1  19
2 2 4 1  19
2 1 4 1  20
2 1 4 1  20
~~~~~
<!--
~~~~~{language=text caption="mr151.dat"}
0 0 1 1 0 0 1 2
0 2 1 1 0 0 1 2
0 0 1 1 0 0 1 2
0 2 2 0 1 0 1 1
0 0 2 1 0 1 2 2
0 2 2 3 0 0 3 4
0 0 3 1 1 0 2 2
0 2 3 0 1 0 1 1
0 0 3 1 0 0 1 2
2 2 1 2 0 0 2 3
2 0 1 1 0 0 1 2
2 0 1 0 0 1 1 1
2 2 2 1 0 1 2 2
2 0 2 1 0 0 1 2
2 2 3 0 1 0 1 1
2 0 3 0 0 1 1 1
2 0 4 0 1 0 1 1
2 2 4 1 0 0 1 2
2 2 4 2 0 0 2 3
2 0 4 2 0 0 2 3
~~~~~
-->

The fourth column is included only to check the original row in the table.

1. Herd ($H_i$); level 1 is the zero-constrained level
2. Sex ($S_j$); 0 = male and 2 = female; level 1 is the zero-constrained level
3. Sire of calf ($s_k$)
4. The row in the original table corresponding to the data line. The parameter file is similar to the regular one except options.

Gibbs sampling by GIBBSF90+
---------------------------

### Parameter file

First, we use GIBBSF90+ for Gibbs sampling. The parameter file (`mr151gs.par`) is similar to the standard one, except for several options.

~~~~~{language=blupf90 caption="mr151gs.par"}
DATAFILE
mr151a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
4
WEIGHT(S)

EFFECTS:    # l=H+C+s+e
1 2 cross   # Herd
2 2 cross   # Sex
3 4 cross   # sire
RANDOM_RESIDUAL VALUES  # residual variance = 1
1.0
RANDOM_GROUP
3
RANDOM_TYPE             # sire model
add_sire
FILE
mr151.ped
(CO)VARIANCES           # sire variance = 1/19
0.0526
OPTION cat 3
OPTION fixed_var mean
~~~~~

The following 2 options are required to perform the threshold analysis with GIBBSF90+.

- `OPTION cat` = Defines the number of categories. This is a single-trait model, so specify one value (3 categories for trait 1). If you apply a multiple-trait model, supply as many integers as there are traits. The value 0 specifies that a trait is continuous (that is, not categorical).
- `OPTION fixed_var mean` = Performs sampling only for the location parameters (that is, the solutions), using the fixed variance components supplied in the parameter file. This is useful when only the solutions are of interest.

You can run GIBBSF90+ with many samples. In this example, we use 10,000 samples with a 5,000-sample burn-in and save all samples after burn-in. This may be more than necessary, but it helps ensure that the results have converged.

Note that GIBBSF90+ does not estimate the thresholds in this case. If there are 2 thresholds (3 categories), the program assumes that the two thresholds are fixed at 0 and 1. If there is 1 threshold (2 categories), the program fixes its value at 0.

### Solutions

The solutions are stored in `final_solutions`.

~~~~~{language=text caption="final_solutions"}
trait/effect level  solution        SD
   1   1         1          9.52944466          8.75831636
   1   1         2          9.85646244          8.76150641
   1   2         1         -9.88169090          8.75776616
   1   2         2        -10.30709508          8.74507553
   1   3         1         -0.05309676          0.21738782
   1   3         2          0.06929745          0.21682631
   1   3         3          0.03496753          0.21566398
   1   3         4         -0.08110327          0.22041841
~~~~~

$$
\begin{aligned}
P_{11} &= \Phi(t_1 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1) = 0.758\\
P_{12} &= \Phi(t_2 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1) - \Phi(t_1 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1) = 0.197\\
P_{13} &= 1 - \Phi(t_2 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1) = 0.044
\end{aligned}
$$

where $\Phi(x)$ is the cumulative distribution function of the standard normal distribution, such as `pnorm()` in R. The reference values are $P_{11} = 0.800$, $P_{12} = 0.129$, and $P_{13} = 0.071$. The results are apparently different from those in the textbook. Why? There are two possible reasons why the results do not match.

The first reason, which is minor, is overconstrained equations. The author imposed two zero constraints on the equations. Although the left-hand side (LHS) of the equations has two very small eigenvalues, they are not small enough to reduce the rank of the LHS. The problem is in the linear model: the author imposed the same constraints on the corresponding linear-model analysis, but this is inappropriate.

The second reason is the use of fixed thresholds with $\sigma_e^2 = 1.0$. When the number of categories ($c$) is 2, that is, for a binary trait, there is only one threshold, and its value can be fixed arbitrarily. When $c = 3$, there are two ways to parameterize the thresholds: (1) one threshold is fixed and the other is estimated, or (2) both thresholds are fixed and $\sigma_e^2$ is estimated (that is, $\sigma_e^2$ is no longer fixed at 1.0). With the latter parameterization, we need to estimate $c-3$ threshold parameters in addition to $\sigma_e^2$. GIBBSF90+ uses this parameterization, so an appropriate value of $\sigma_e^2$, rather than 1.0, is needed for the fixed thresholds. In the example above, the residual variance was inappropriate. One may ask why the software implements the model in this way. In variance-component estimation using Gibbs sampling for a threshold model with 3 or more categories (2 or more thresholds), the convergence rate for a threshold parameter is very slow. Sampling the residual variance is a better strategy when 2 or more thresholds are assumed in the model. See Sorensen et al. (1995) or Wang et al. (1997) for a detailed discussion. Thus, `OPTION fixed_var mean` may not be a good choice. You may explore this issue yourself.


Newton-Raphson iterations by BLUPF90+
-------------------------------------

### Parameter file

Next, we use BLUPF90+ for Newton–Raphson iterations in a general threshold-linear framework (Bermann et al., 2026). The parameter file (`mr151nr.par`) has additional options for supporting the threshold model.

~~~~~{language=blupf90 caption="mr151nr.par"}
DATAFILE
mr151a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
4
WEIGHT(S)

EFFECTS:    # l=H+C+s+e
1 2 cross   # Herd
2 2 cross   # Sex
3 4 cross   # sire
RANDOM_RESIDUAL VALUES  # residual variance = 1
1.0
RANDOM_GROUP
3
RANDOM_TYPE             # sire model
add_sire
FILE
mr151.ped
(CO)VARIANCES           # sire variance = 1/19
0.0526
OPTION category 3
OPTION threshold_model nr
OPTION constant_thresholds 0.0 1.0
~~~~~

The following 3 options are required to perform the threshold analysis with BLUPF90+.

- `OPTION category` = Defines the number of categories. Its usage is the same as that of `OPTION cat` in GIBBSF90+ (see above).
- `OPTION threshold_model nr` = Performs Newton–Raphson iterations to solve the nonlinear equations resulting from a multivariate threshold-linear model. BLUPF90+ also supports the EM algorithm (`em`), which is more stable but slower.
- `OPTION constant_thresholds 0.0 1.0` = Fixes the thresholds. BLUPF90+ needs to fix two thresholds for a three-category trait. Here, we use the constants 0 and 1 to align with GIBBSF90+.

BLUPF90+ repeatedly constructs and solves the equations during the iterative steps. The final solutions are stored in `solutions`.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1         -0.35218672
   1   1         2         -0.03022119
   1   2         1          0.01636063
   1   2         2         -0.39109144
   1   3         1         -0.04791721
   1   3         2          0.06891131
   1   3         3          0.04201164
   1   3         4         -0.07645558
~~~~~

The solutions for the sire effect are similar to those from Gibbs sampling. The conversion from the underlying scale to the linear scale is the same as that explained above for the Gibbs-sampling case.


### Extra test with fixed thresholds

In the analyses above, the thresholds are fixed at 0 and 1. Fixed thresholds simplify the computation, which is a great advantage for a complicated model with a large data set. If we specify more reasonable thresholds, we will obtain solutions closer to those in the textbook.

In the textbook, the thresholds are estimated as 0.4378 and 1.0675. We can use these values in the parameter file with `OPTION constant_thresholds 0.4378 1.0675`. With this option, BLUPF90+ gives the following solutions.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1         -0.16065139
   1   1         2          0.11667363
   1   2         1          0.16067717
   1   2         2         -0.19826923
   1   3         1         -0.04340437
   1   3         2          0.05917751
   1   3         3          0.04120522
   1   3         4         -0.06596450
~~~~~

The random-effect solutions are identical to those in the textbook (p. 271).


Alternative solutions
---------------------

In this subsection, we will find ways to obtain the same solutions as those in the textbook. First, we try an older program, CBLUP90THR, which is available from our web server. We do not explain how to use this program here. This program supports the computation of two or more thresholds in a single-trait threshold model. It also supports a multiple-trait model with one threshold trait and many linear traits.

The program implements Hoeschele et al. (1995) which generalized Foulley et al. (1983) and Janss and Foulley (1993) with some unpublished work by Richard Quaas (presented at a meeting of Beef Improvement Federation).

[^3]: The current binary on the UGA server does not seem to run well. The following description is from an older version of CBLUP90THR.

The following results are from the program using the same parameter file with enough iterations to achieve convergence (100 rounds here).

~~~~~{language=text caption="solutions"}
trait/effect level  solution
  1  1       1     -0.8942
  1  1       2     -0.6168
  1  2       1      0.4014
  1  2       2      0.0425
  1  3       1     -0.0434
  1  3       2      0.0592
  1  3       3      0.0412
  1  3       4     -0.0660
~~~~~

The thresholds are $t_1 = -0.0550$ and $t_2 = 0.5748$. The difference between the thresholds is $t_2 - t_1 = 0.6298$, which is the same as in the reference analysis. With the solutions above, $P_{11} = 0.800$, $P_{12} = 0.130$, and $P_{13} = 0.070$. The results are nearly identical to the reference values in the textbook.

Next, we run GIBBSF90+ with a different residual variance. Let $\sigma_e^2 = 3.0$. To keep the same variance ratio, $\sigma_s^2$ should be 0.158, so that $\sigma_e^2/\sigma_s^2 = 19.0$. The thresholds are fixed at $t_1 = 0.0$ and $t_2 = 1.0$. The final solutions from GIBBSF90+ are as follows.

~~~~~{language=text caption="final_solutions"}
trait/effect level  solution        SD
   1   1         1         16.34777283         15.17002602
   1   1         2         16.82596758         15.17639696
   1   2         1        -17.16324150         15.16948659
   1   2         2        -17.80476157         15.14659841
   1   3         1         -0.08323076          0.37743431
   1   3         2          0.10108613          0.37648625
   1   3         3          0.05924918          0.37371633
   1   3         4         -0.12015570          0.38254070
~~~~~

When $\sigma_e^2 \neq 1.0$, we should standardize the liability predictor by the residual standard deviation before applying the cumulative normal function to calculate the probabilities. See the following calculations.

$$
\begin{aligned}
P_{11} &= \Phi\left( \frac{t_1 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1}{\sigma_e} \right) = 0.813\\
P_{12} &= \Phi\left( \frac{t_2 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1}{\sigma_e} \right) - \Phi\left( \frac{t_1 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1}{\sigma_e} \right) = 0.116\\
P_{13} &= 1 - \Phi\left( \frac{t_2 - \hat{h}_1 - \hat{c}_2 - \hat{u}_1}{\sigma_e} \right) = 0.071
\end{aligned}
$$

Again, $\sigma_e$ is the residual standard deviation ($\sqrt{\sigma_e^2}$) because the formula is based on standardizing the latent variable. The probabilities are very similar to the reference results for all sires (p. 229).
