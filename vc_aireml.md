---
title: Variance component estimation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Restricted (residual) maximum likelihood with BLUPF90+
======================================================

REML software
-------------

### REML algorithms

REML (Restricted/Residual Maximum Likelihood) is a widely used method for estimating variance components in animal breeding. Several algorithms are available for REML, and BLUPF90+ supports the following two:

- Average Information (AI) algorithm
- Expectation-Maximization (EM) algorithm

The AI algorithm (AI REML) is an iterative method that requires initial values for the variance components. In the first iteration, the algorithm updates these values based on the provided starting point. In each subsequent iteration, new values are computed from the previous ones. After several iterations, the estimates are expected to converge to stable values. The EM algorithm (EM REML) is another iterative method, but it uses a different computational approach. In theory, both algorithms should yield the same estimates of variance components. For more information on the differences between these algorithms, see Misztal (2008).

BLUPF90+ serves as a unified program that supports solving the mixed model equations given variance components as well as estimating variance components with both EM and AI REML algorithms. Previously, the BLUPF90 family included two separate programs: AIREMLF90 for AI REML and REMLF90 for EM REML. This tutorial uses only BLUPF90+. If you are familiar with the older programs, please see a [separate page](./introduction_plus.md) for compatibility information. We will focus on the use of the AI REML algorithm, as it has long been the preferred tool for REML estimation and also provides (approximate) standard errors of genetic parameters. Additionally, BLUPF90+ can run the EM algorithm using an option specified in the parameter file.

### Invoking variance component estimation

You should add the following option to the parameter file to let BLUPF90+ estimate variance components by REML.

~~~~~{language=blupf90 caption="variance component estimation (AI REML)"}
OPTION method VCE
~~~~~

Without this option, BLUPF90+ will not estimate variance components. Instead, it will default to solving the mixed model equations using the variance components provided in the parameter file. Failing to include this option is a common mistake. Including it enables BLUPF90+ to operate in the same way as the old AIREMLF90. To use the pure EM algorithm (emulating REMLF90), add the following option to the parameter file, as explained in the subsequent section:

~~~~~{language=blupf90 caption="variance component estimation (EM REML)"}
OPTION method VCE
OPTION EM-REML
~~~~~

The number of maximum iterations is controlled by the following option.

~~~~~{language=blupf90 caption="the number of REML iterations"}
OPTION maxrounds 10000
~~~~~

Please refer to the official website and manual for a detailed explanation of variance component estimation in BLUPF90+.


Preparation
-----------

You can use the same parameter file as for solving the mixed model equations. BLUPF90+ reads the variance components as initial values (starting values) from the parameter file. In this section, we will use simulated files that have more observations than before. You can download the files from GitHub (<https://github.com/masuday/data>).

- [`simdata.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/aireml/simdata.txt) : data file
- [`simped.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/aireml/simped.txt) : pedigree file

The pedigree file contains three columns: animal, sire, and dam. The data file has 12 columns as described below.

Column       Item           type       description
------       ---------      -------    ----------------------------------
1            Animal ID      integer    Same as in pedigree (4641 animals)
2            Sire ID        integer    Same as in pedigree
3            Dam ID         integer    Same as in pedigree
4            Weight         real       Not used here
5            Mu             integer    All 1: not used here
6            Farm           integer    Class effect (155 levels)
7            Sex            integer    Class effect (2 levels)
8            Year           integer    Class effect (11 levels)
9            Obs. 1         real       Phenotype for trait 1
10           Obs. 2         real       Phenotype for trait 2
11           Obs. 3         real       Phenotype for trait 3
12           Obs. 4         real       Phenotype for trait 4


Single-trait analysis
---------------------

### Model ###

Here, we consider the following animal model:
$$
y_{ijkl} = F_{i} + S_{j} + Y_{k} + u_{l} + e_{ijkl}
$$
where $y_{ijkl}$ is an observation, $F_i$ is the fixed farm effect, $S_j$ is the fixed sex effect, $Y_k$ is the fixed year effect, $u_l$ is the additive genetic effect, and $e_{ijkl}$ is the random residual effect. The purpose of this analysis is to estimate the additive genetic variance ($\sigma_u^2$) and the residual variance ($\sigma_e^2$).

### Parameter file ###

Let us start with a single-trait analysis for phenotype 1 (column 9). The model contains farm, sex, and year as fixed cross-classified effects and the additive genetic effect as a random effect. Place the following parameter file in a directory with the data and pedigree files. As mentioned repeatedly, the parameter file looks like a standard one used to solve the BLUP equations.

~~~~~{language=blupf90 caption="aireml1.txt"}
DATAFILE
simdata.txt
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
9
WEIGHT(S)
4
EFFECTS:
 6   155 cross
 7   2 cross
 8   11 cross
 1   4641 cross
RANDOM_RESIDUAL VALUES
100
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
simped.txt
(CO)VARIANCES
100
OPTION method VCE
~~~~~

The variance components in the parameter file will be used as the initial values ($\sigma_e^2 = 100$ and $\sigma_u^2 = 100$) in AI REML.


### Intermediate estimates ###

Running `blupf90+` with the above parameter file, the program reports the intermediate estimates for the current round. AI REML is an iterative method --- the variance estimates change in each round, and the updated values are expected to approach the final estimates. After several rounds, when the changes are very small, we can consider the estimates to have converged. The following messages are from the first round.

~~~~~{language=output}
-2logL =     34141.8991194978     : AIC =     34145.8991194978    logL convergence   0.997071027755737
Corrections made:    3 , final bending proportions of AI and EM     0.970000    0.030000
  In round            1  convergence=   189.251523744639
  delta convergence=   81.8578709853283
 new R
   29.508
 new G
   6.7765
~~~~~

Some information is available in the message.

- `-2logL` = negative twice the restricted log-likelihood evaluated with the current variance components (along with the BLUE and BLUP).
- `AIC` = $-2\log L + 2p$ where $p$ is the number of unique variance components
- `convergence` = the current convergence indicator (see below)
- `delta convergence` = another convergence indicator
- `new R` = updated residual variance ($\sigma_e^2$)
- `new G` = updated genetic variance ($\sigma_u^2$)

In each round, the program computes two convergence indicators. One convergence indicator ($C$) is calculated as
$$
C=\frac{\sum_{i}(\theta_{i}-\theta^{*}_{i})^2}{\sum_{i}\theta_{i}^2}
$$
where $\theta_i$ is the current estimate of the $i$th variance component and $\theta^{*}_{i}$ is the previous estimate of the $i$th variance component. This criterion is also used in the EM REML algorithm. The other indicator, called the delta convergence ($C_{\Delta}$), is calculated as
$$
C_{\Delta}=\frac{\sum_{i}|\theta_{i}-\theta^{*}_{i}|}{n}
$$
where $n$ is the number of unique (co)variance components. This indicator represents the average absolute change across the estimates.

The iteration stops if $C<\varepsilon$ or $C_{\Delta}<\sqrt{\varepsilon}$ where $\varepsilon$ is the convergence criterion and its default is $\varepsilon=10^{-12}$ (`1.0E-12`). When one indicator approaches the criterion, the other one usually approaches the criterion as well.

You can also monitor the log-likelihood (`-2logL`), which is expected to decrease from one round to the next. The log-likelihood is not a direct convergence criterion, but it can be useful for checking whether the iterations are proceeding normally.


### Final estimates ###

If either convergence indicator meets the criterion, the iteration stops and the final estimates are shown on the screen. For the current data and initial values, 8 rounds are needed for convergence.

~~~~~{language=output}
-2logL =     33423.4510659477     : AIC =     33427.4510659477    logL convergence   0.241636111076790E-12
  In round            9  convergence=  1.470965996000675E-013
  delta convergence=  1.194144151034722E-005
 new R
   62.691
 new G
   38.538
 * END ITERATION: 07-29-2026  21h 52m 43s 185
 solutions stored in file: solutions

Final Estimates
 Genetic variance(s) for effect  4
   38.538
 Residual variance(s)
   62.691
 inverse of AI matrix (Sampling Variance)
   13.471      -6.7551
  -6.7551       6.5499
 Correlations from inverse of AI matrix
   1.0000     -0.71914
 -0.71914       1.0000
 SE for G
   3.6703
 SE for R
   2.5593
 *** Statistical Method: VCE
~~~~~

The program ends with `Final Estimates` showing the variance estimates at convergence. The same information is saved in the file `blupf90+.log`. The final estimates are $\hat{\sigma}_u^2 = 38.538$ and $\hat{\sigma}_e^2 = 62.691$, so the heritability estimate is $\hat{h^2} = 0.3807$. The program does not show the heritability by default, and the user should calculate it manually.

The program saves the solutions of the mixed model equations using the most recent variance estimates in the file `solutions`. Note that the solutions do not come from the final variance estimates, but from those in the round before convergence. In this example, the solutions are from the variance estimates in round 7. When the algorithm works well and converges correctly, the final variance estimates are almost the same as those used in the previous round, so the difference between the two solutions is negligible. If the convergence trajectory fluctuates, the two solutions could differ substantially. In such a case, the variance estimates themselves are unreliable. At the end of this section, we will consider convergence problems.


### Approximate standard error ###

The approximate standard error (sampling variance) for a variance estimate is available from the inverse of the AI matrix. Suppose the parameter vector is $\theta$ and the AI matrix is $\mathcal{I}(\theta)$. The sampling variance of
$\hat{\theta}$ is
$$
\mathrm{var}(\hat{\theta}) = \left[\mathcal{I}(\theta)\right]^{-1}
$$
where $\left[\mathcal{I}(\theta)\right]^{-1}$ is the inverse of the AI matrix. In this example, $\theta=\left[\begin{array}{ll}\sigma_u^2&\sigma_e^2\end{array}\right]'$. The square roots of the diagonal elements of $\left[\mathcal{I}(\theta)\right]^{-1}$ are shown as the `SE for G` ($\sqrt{13.471}$) and `SE for R` ($\sqrt{6.5499}$) in the last lines of the output.

Unfortunately, the program does not calculate the standard error of the heritability by default. There are at least two approaches to obtain the SE of variance components or heritability (i.e., a function of variance components).

First, you can obtain an approximate SE for a function of the variance components (for example, heritability) using the delta method by hand. See the documentation on the official wiki (<http://nce.ads.uga.edu/html/projects/AI_SE_revised.pdf>) for details. Here we just show a way of calculating the standard error for the heritability estimate. The heritability estimate is 0.3807. Using the approximation, the standard error is
$$
\begin{aligned}
\mathrm{SE}\left(h^2\right)&=\left(\frac{h^2}{\sigma_u^2}\right)\left[ (1-h^2)^2\mathrm{var}(\hat{\sigma}_u^2)-2(1-h^2)h^2\mathrm{cov}(\hat{\sigma}_u^2,\hat{\sigma}_e^2)+(h^2)^2\mathrm{var}(\hat{\sigma}_e^2) \right]\\
&=\left(\frac{0.3807}{38.54}\right)\left[ (1-0.3807)^2\times 13.471-2(1-0.3807)\times 0.3807\times (-6.755)+(0.3807)^2\times 6.550 \right]\\
&=0.007
\end{aligned}
$$

Another way to evaluate the SE for the heritability estimate is the Monte Carlo method suggested by Meyer and Houle (2013). You can find a do-it-yourself description of the method at (<http://artadia.blogspot.fr/2016/05/standard-error-of-variance-components.html>). It uses the AI matrix at convergence. BLUPF90+ can conduct this method using options in the parameter file. You can add the following line to the bottom of the parameter file and run `blupf90+`.

~~~~~{language=blupf90}
OPTION se_covar_function h2 G_4_4_1_1/(G_4_4_1_1+R_1_1)
~~~~~

This option has two arguments: label and formula. The label can contain arbitrary characters. The formula describes an objective function for which the standard error should be calculated ($\sigma_u^2/(\sigma_u^2 + \sigma_e^2)$ in this case). The formula is described using special labels for variance components. For a genetic covariance component, use `G_effect1_effect2_trait1_trait2`; for a residual covariance, use `R_trait1_trait2`. In this single-trait case, the genetic variance $\sigma_u^2$ is defined as effect 4 in the parameter file, so we refer to $\sigma_u^2$ as `G_4_4_1_1`. Similarly, we refer to $\sigma_e^2$ as `R_1_1`. Spaces are not allowed in the formula. See the manual for details on this option.

After the main iterations have converged, the program calculates an approximate standard error using the Monte Carlo technique. In this case, you can see the following output on the screen.

~~~~~{language=output}
h2  - Function: g_4_4_1_1/(g_4_4_1_1+r_1_1)
  Mean:   0.38070
  Sample Mean:   0.38002
  Sample SD:   0.30426E-01
 elapsed time  6.2208176E-03
~~~~~

In the output, `Sample Mean` should be almost identical to the estimate. `Sample SD` is an approximate
standard error, which is 0.03 (`0.30426E-01` is $0.30426 \times 10^{-1}$ that is 0.030426).

Do not worry if you forgot to add the option to the parameter file before estimating the variance components; you can obtain the SE after finishing the BLUPF90+ run. Prepare a parameter file with the option `se_covar_function` and the variance components at convergence. When you run BLUPF90+ again, the program will stop within 1 or 2 rounds and calculate the approximate standard error of the parameter. However, this approach does not always work, and you may see a series of iterations, particularly if there were convergence issues, as mentioned below.


Convergence problems
--------------------

The AI algorithm does not guarantee convergence. Sometimes convergence fails in four ways:

- Divergence. The estimates suddenly jump to nonsense values and the program stops.
- Random walk. The estimates go back and forth, and the convergence trajectory is not stable.
- No move. The estimates do not change, and the results are almost the same as the arbitrary initial values.
- Slow convergence. The estimates change very little from round to round (several hundred rounds or more may be needed for convergence).

If you see nonsense estimates or suspicious convergence behavior, you should not take those values.

These problems may have several causes.

1. Mistakes in the parameter file, data, and/or pedigree.
2. Wrong or unstable initial values.
3. A model that is too complicated relative to the limited amount of data and/or pedigree information.
4. Too many variance components to be estimated.
5. Extreme parameter values, such as nearly zero variance or extremely high genetic correlations.
6. Extremely unbalanced data: such as too many small groups or excessive missing observations.

If your files are correct and changing the initial values does not resolve the issues, you should simplify your model. If the issue is caused by an extreme data structure, you may need to abandon the analysis in its current form.

BLUPF90+ tries to correct nonsensical estimates (outside the parameter space) using a technique described by Jensen et al. (1996). The method blends the AI matrix from the regular algorithm ($\mathcal{I}_{\mathrm{AI}}$) with the AI matrix from the EM algorithm ($\mathcal{I}_{\mathrm{EM}}$), that is, $\mathcal{I}(\theta)= (1-w)\mathcal{I}_{\mathrm{AI}} + w\mathcal{I}_{\mathrm{EM}}$, with a weight $w$. If $w = 1$, the algorithm is equivalent to the regular EM algorithm, which always provides estimates within the parameter space. If an estimate is not positive definite (that is, a nonsensical value), BLUPF90+ sets a small $w$, blends the two AI matrices, modifies the blended AI matrix to be valid, and recalculates the estimates with the modified AI matrix. The program repeats this blending, increasing $w$ until the estimates make sense, as you see in the warning message.

~~~~~{language=output}
*** Warning *** corrected Covariance Matrix 5 times
~~~~~

If the correction does not resolve the ill-conditioned matrix, BLUPF90+ will fail and stop with a message. In such a case, the printed estimates are most likely nonsensical, and you should not consider them to be valid estimates.

The blending is a band-aid method and is successful only when the initial AI matrix is recoverable. If the blending happens every round, you can still obtain estimates, but you need many more rounds to converge, or you may eventually see divergence.

EM REML algorithm
-----------------

The EM (expectation-maximization) REML algorithm is much more stable than the AI algorithm and very robust to poor initial estimates. It can also provide good starting values for the AI algorithm because convergence is fast enough when the current estimates are far from the converged values. The EM algorithm is a pure statistical method, and it has several desirable features, such as estimates remaining inside the parameter space. However, the EM algorithm is _much_ slower; that is, it needs more rounds to converge. The convergence rate decreases greatly as the estimates approach convergence. Nevertheless, some users decide to use EM REML; for some very complex problems, only EM REML may converge.

You can run the full EM algorithm in BLUPF90+ using either of the following options:

~~~~~{language=blupf90}
OPTION EM-REML
OPTION EM-REML pure   # another option
~~~~~

With either of the above options, the AI matrix is never computed. This means that you cannot obtain the SE of the parameters using `OPTION se_covar_function` because it depends on the AI matrix at convergence.

If you want to evaluate the SE of the parameter estimates, you should switch from EM REML to AI REML after a certain number of iterations. BLUPF90+ supports this algorithm switch. A simple method is to run EM REML until convergence and then calculate the AI matrix (without AI REML iterations) using the following option.

~~~~~{language=blupf90}
OPTION EM-REML AI
~~~~~

Or, if you prefer to run EM REML for a certain number of rounds and then switch to AI REML until convergence, the following option is available.

~~~~~{language=blupf90}
OPTION EM-REML 10   # run 10 EM REML rounds then switch to AI REML
~~~~~

There are several more options to manage the REML iterations.
Please see the manual for details.


Constraints on variance components
----------------------------------

The user may want to keep some variance components fixed (constant) throughout the iterations. BLUPF90+ supports this type of general constraint on variance components through the following option.

~~~~~{language=blupf90}
OPTION constant_var <effect> <trait1> <trait2>
~~~~~

With this option, the program fixes (i.e., never updates) the variance components for effect `<effect>` between traits `<trait1>` and `<trait2>`.
For example, to fix the covariance for effect 5 between trait 1 and 2, the option is `OPTION constant_var 5 1 2`.

A more frequent constraint is that (co)variance components are fixed at zero. BLUPF90+ supports forcing some variance components to be 0. This is useful when such variance components will never be included in the model or when some covariance components should be 0 in a complicated model. When the initial variance component is 0, this component never changes throughout the iterations. For example, the following parameter file forces the direct and maternal genetic covariance to be fixed at 0.

~~~~~{language=blupf90}
DATAFILE
renf90.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
 4  3 cross
 5  2 cross
 1 14 cross  # direct genetic effect
 3 14 cross  # maternal genetic effect
RANDOM_RESIDUAL VALUES
350.0
RANDOM_GROUP
3 4
RANDOM_TYPE
add_animal
FILE
renadd01.ped
(CO)VARIANCES  # fixed to be zero for covariance
150  0
 0  90
OPTION method VCE
~~~~~

When the initial values are 0, BLUPF90+ automatically excludes such variance components from the estimation process. More precisely, BLUPF90+ does not include such parameters in the AI matrix, and therefore, the excluded parameters will not be calculated in any way. On the one hand, you may expect faster computation and more stable convergence because fewer parameters will be estimated. On the other hand, if the constrained variance component is not truly zero, the other estimates may be biased.

Note that, when an estimate is outside the parameter space and BLUPF90+ fails to fix it using the AI-EM combination, some variances may accidentally become 0. Once this happens, those variances will not return to nonzero values. This is not because of the constraint mentioned above. It is simply because the information needed for variance estimation will be lost if the variance is 0.

Summary
-------

- BLUPF90+ calculates REML estimates of variance components.
- BLUPF90+ accepts the same parameter file as the BLUP prediction and uses the variance components as initial values.
- The final estimates are shown on screen and saved in the file `blupf90+.log`.
- The solutions are also calculated using variance component estimates from the round before convergence.
- There are a couple of methods to approximate the standard error of a genetic parameter.
- The AI REML algorithm does not guarantee convergence, even though it tries to keep the estimates within the parameter space.
- If the algorithm diverges, use a simpler model.
- For complex problems, you can use the EM algorithm instead of AI REML.
- If the initial variance component is 0, it will be fixed at 0 throughout the iterations.
