---
title: Variance component estimation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Advanced features in AI REML
============================

Heterogeneous residual variances
--------------------------------

There are situations in which residual variances vary across conditions. This is known as heterogeneous residual variance. There are two types of models for heterogeneous residual variances in BLUPF90+.

1. The residual variance is related to a covariate. For example, the residual variance for body weight increases with age in months. The residual variance can be described as a function of a covariate (age).
2. The residual variance differs by class. For example, in a test-day random-regression model, we usually assume a different residual variance for each lactation stage. The residual variances are independent of one another.

Although BLUPF90+ is designed to handle the first case, the program can also handle the second case with a trick. We will examine both cases using numerical examples with REML. GIBBSF90+ supports only the second case.

Note that, as stated in the manual, convergence will be very slow if you apply heterogeneous residual (co)variances to a multiple-trait model.


### Model ###

In this example, we assume the following model:
$$
y_{ijkl} = F_{i} + S_{j} + Y_{k} + u_{l} + e_{ijkl}
$$
where $y_{ijkl}$ is the observation, $F_i$, $S_j$, and $Y_k$ are the fixed effects, $u_l$ is the additive genetic effect, and $e_{ijkl}$ is the residual effect. The residual variance is defined by the following function:
$$
\sigma_e^2 = \exp(b_0 + b_1 x_{ijkl})
$$
where $b_0$ and $b_1$ are the regression coefficients used to model the residual variance, and $x_{ijkl}$ is the covariate measured with the observation $y_{ijkl}$. The model is based on Foulley and Quaas (1995). BLUPF90+ will estimate $b_0$ and $b_1$ using the user-supplied values of $x_{ijkl}$.


### Data ###

In this section, we will use simulated files similar to those in the previous section. You can download the files from GitHub.

- [`simdata2.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/adv_aireml/simdata2.txt) : data file
- [`simped2.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/adv_aireml/simped2.txt) : pedigree file

The pedigree file contains three columns: animal, sire, and dam. The data file has 14 columns, as described below.

Column  Item           type      description
------  ---------      -------   ----------------------------------
1       Animal ID      integer   Same as in pedigree (4641 animals)
2       Sire ID        integer   Same as in pedigree
3       Dam ID         integer   Same as in pedigree
4       Weight         real      Not used here
5       Mu             integer   All 1: not used here
6       Farm           integer   Class effect (155 levels)
7       Sex            integer   Class effect (2 levels)
8       Year           integer   Class effect (11 levels)
9       Obs. 1         real      Phenotype for trait 1
10      Obs. 2         real      Phenotype for trait 2
11      Obs. 3         real      Phenotype for trait 3
12      Obs. 4         real      Phenotype for trait 4
13      Covariate      real      Related to residual variance
14      Class          integer   (Used in the next subsection)

Column 13 contains a real-valued covariate. In these artificial data, the residual variance is expected to increase as the covariate increases. The 14th column contains the heterogeneous-residual-variance class with 3 levels, which will be used in the next subsection.


### Parameter file ###

The following parameter file is used.

~~~~~{language=blupf90 caption="aireml2.txt"}
DATAFILE
simdata2.txt
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
9
WEIGHT(S)
4
EFFECTS:
 6  155 cross
 7    2 cross
 8   11 cross
 1 4641 cross
RANDOM_RESIDUAL VALUES
100
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
simped2.txt
(CO)VARIANCES
100
OPTION method VCE
OPTION hetres_pos 13
OPTION hetres_pol 4.5 0.5
~~~~~

The options define the model for heterogeneous residual variances.

- `OPTION hetres_pos` = the position of covariate $x$ in the data file. The intercept $b_0$ is implicitly included in the program.
- `OPTION hetres_pol` = the starting values for $b_{0}$ and $b_{1}$.
    - You should supply the values for all coefficients including intercept.
	  - In this example, the initial value will be $\exp(4.5 + 0.5) = 148.4$ for $x = 1$.

In the above case, we define only one covariate in `hetres_pos`, but the program adds the intercept. If you put two numbers in `hetres_pos` and also put two initial values in `hetres_pol`, the program does not add the intercept. In such a case, the model contains no intercept and includes only the user-supplied covariates.

Note that these options do not work with `OPTION EM-REML`. The current version of BLUPF90+ will report this limitation when it encounters this condition.

### Results ###

You can run `blupf90+` with the above parameter file. The following output will be shown.

~~~~~{language=output}
Final Estimates
 Genetic variance(s) for effect  4
   39.384
 new R
           1 -th trait:           1 -th coefficient =   4.25520168816761
           1 -th trait:           2 -th coefficient =  5.886409958430982E-002
 inverse of AI matrix (Sampling Variance)
   16.773     -0.12514      0.77591E-02
 -0.12514      0.41370E-02 -0.16778E-02
  0.77591E-02 -0.16778E-02  0.10722E-02
 Correlations from inverse of AI matrix
   1.0000     -0.47507      0.57858E-01
 -0.47507       1.0000     -0.79664
  0.57858E-01 -0.79664       1.0000
 SE for G
   4.0955
 SE for R
  0.64319E-01
~~~~~

The output shows $\hat{b}_0 = 4.2552$ and $\hat{b}_1 = 0.058864$. This corresponds to $\hat{\sigma}_e^2 = \exp(4.26 + 0.0589 \times 0.5) = 72.9$ for $x = 0.5$, $\hat{\sigma}_e^2 = \exp(4.26 + 0.0589 \times 1.5) = 77.4$ for $x = 1.5$, and $\hat{\sigma}_e^2 = \exp(4.26 + 0.0589 \times 2.5) = 82.0$ for $x = 2.5$.

One drawback of this analysis is that the calculations of the standard error for the residual variance are not easy. The AI matrix contains information for $\hat{b}_0$ and $\hat{b}_1$ (the last 2 rows/columns).


### Another modeling by class ###

The heterogeneous residual variances are modeled by class. We assume that the class has 3 levels. The 14th column in the above data contains the level for each observation. BLUPF90+ with the AI REML function does not directly accept the level number, so we must convert it to a different format.

Having 3 levels is equivalent to defining 3 regression coefficients without an intercept. We now assume the following parameterization without an intercept.
$$
\sigma_e^2 = \exp(b_1 x_1 + b_2 x_2 + b_3 x_3 )
$$

If the observation has residual variance level 1, we have $x_1 = 1$, $x_2 = 0$, and $x_3 = 0$. For level 2, $x_1 = 0$, $x_2 = 1$, and $x_3 = 0$; for level 3, $x_1 = 0$, $x_2 = 0$, and $x_3 = 1$. We need 3 covariates with values of 1 or 0. We add 3 extra columns to the rightmost part of the data file.

- `simdata2a.txt`: extended data file

The first 10 rows are shown below.

~~~~~{language=text caption="simdata2a.txt"}
  1       0       0 1.00   1    67   1   1    85.0     0.0    92.1    91.0  0.46  1   1 0 0
  2       0       0 0.98   1   144   1   1   115.2   103.9    94.8    90.6  1.52  2   0 1 0
  3       0       0 1.04   1    92   1   1    93.0   107.4   114.9   107.0  0.00  1   1 0 0
  4       0       0 0.97   1    26   1   1    84.0    91.8    93.7   107.1  0.61  1   1 0 0
  5       0       0 0.96   1    83   1   1   100.9    87.9    88.9    91.5  0.55  1   1 0 0
  6       0       0 0.99   1    62   1   1    69.8    71.5    69.3     0.0  0.88  1   1 0 0
  7       0       0 0.97   1    40   1   1   101.2     0.0    92.1    88.6  0.27  1   1 0 0
  8       0       0 1.07   1    82   1   1    77.4    72.7    84.1   105.2  1.69  2   0 1 0
  9       0       0 1.01   1    71   1   1   107.8    87.0    97.8    76.0  2.89  3   0 0 1
 10       0       0 0.95   1   114   1   1    94.4   100.5   103.8    89.8  1.26  2   0 1 0
~~~~~

Columns 15, 16, and 17 contain 1 or 0. In column 15, the value is 1 if the level is 1 and 0 otherwise. In column 16, the value is 1 if the level is 2 and 0 otherwise. In column 17, the value is 1 if the level is 3 and 0 otherwise. With this data file, the parameter file is as follows.

~~~~~{language=blupf90 caption="aireml2a.txt"}
DATAFILE
simdata2a.txt
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
 7     2 cross
 8    11 cross
 1  4641 cross
RANDOM_RESIDUAL VALUES
100
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
simped2.txt
(CO)VARIANCES
100
OPTION method VCE
OPTION hetres_pos 15 16 17
OPTION hetres_pol 4.5 4.5 4.5
~~~~~

The results from the above parameter file are shown here.

~~~~~{language=output}
Final Estimates
 Genetic variance(s) for effect  4
   39.296
 new R
           1 -th trait:           1 -th coefficient =   4.28231170019646
           1 -th trait:           2 -th coefficient =   4.36065608498972
           1 -th trait:           3 -th coefficient =   4.38963108754592
 inverse of AI matrix (Sampling Variance)
   16.735     -0.11982     -0.11288     -0.10685
 -0.11982      0.32743E-02  0.75391E-03  0.69459E-03
 -0.11288      0.75391E-03  0.31352E-02  0.67693E-03
 -0.10685      0.69459E-03  0.67693E-03  0.29536E-02
 Correlations from inverse of AI matrix
   1.0000     -0.51184     -0.49281     -0.48059
 -0.51184       1.0000      0.23530      0.22335
 -0.49281      0.23530       1.0000      0.22245
 -0.48059      0.22335      0.22245       1.0000
 SE for G
   4.0909
 SE for R
  0.57221E-01
~~~~~

The estimate for level 1 is $\hat{\sigma}_e^2 = \exp(4.28) = 72.2$, for level 2 it is $\hat{\sigma}_e^2 = \exp(4.36) = 78.3$, and for level 3 it is $\hat{\sigma}_e^2 = \exp(4.39) = 80.6$. These values are very similar to the previous results obtained with the regression parameterization $\exp(b_0 + b_1 x)$ for $x = 0.5$, $1.5$, and $2.5$.


### Remarks ###

The methods explained here will not work well for multiple-trait models or complicated models with many variance components.

Likelihood Ratio Test
---------------------

REML methods allow statistical testing of variance components. For instance, imagine that we have to choose one of two competing models:

  - model 1 _not_ including the maternal permanent environmental effect
  - model 2 _including_ the maternal permanent environmental effect.

The Likelihood Ratio Test (LRT) checks whether including the extra random effect, with its associated variance component, provides a better model fit than excluding it. The model _without_ the extra random effect is the _null_ ($H_0$) model, whereas the model _with_ the extra random effect is the alternative model ($H_1$). The difference between the two likelihood values is positive and is the LRT statistic, which follows a mixture of $\chi^2$ distributions. The theory of the LRT can be found in standard books; a useful description is given in Sorensen and Gianola's book.

The output of BLUPF90+ under $H_0$ is $x=-2\log L$; the output under $H_1$ is $y=-2\log L$. These are positive numbers, so smaller is better; $y<x$ because $H_1$ is a more complex (and therefore more likely) model. The question is whether $x-y$ is "large enough." The LRT statistic is $LRT=x-y$, which follows a mixture of $\chi^2$ distributions. For one variance component, the p-value is calculated in R as follows:

~~~{language=r}
pchisq(x-y,1,lower.tail=FALSE)/2
~~~

Classical applications of the LRT in quantitative genetics include testing whether $h^2 > 0$ and testing QTL effects. An application to the association analysis of a multiallelic gene is described at <http://dx.doi.org/10.3168/jds.2013-6570>.

### Example for test of heritability in dairy sheep

I tested whether genetic effects ($\mathbf{u}$) should be included in an old data set from dairy sheep (80,000 records and 50,000 animals in the pedigree) using a model that includes permanent environmental ($\sigma^2_p$), genetic ($\sigma^2_u$), and residual ($\sigma^2_e$) variances.

Under the complete model $H_1$ ($\mathbf{y} = \mathbf{Xb} + \mathbf{Zu} + \mathbf{Zp} + \mathbf{e}$), the last lines of the BLUPF90+ output are:

~~~
-2logL =        825227.48
~~~

Then, we run the reduced model $H_0$ with a parameter file that does _not_ include the genetic effect ($\mathbf{y} = \mathbf{Xb} + \mathbf{Zp} + \mathbf{e}$):

~~~
 -2logL =    828095.614456413
~~~

The test in R is:

~~~{language=R}
pchisq(828095.614456413-825227.48,1,lower.tail=FALSE)/2

[1] 0
~~~


The p-value is so small that we cannot distinguish it from 0. We can take the logarithm and transform it to the scale $-\log_{10}(\mathrm{p\text{-}value})$, which is commonly used in GWAS:

~~~{language=R}
pval=pchisq(828095.614456413-825227.48,1,lower.tail=FALSE,log.p=TRUE)-log(2)
> pval
[1] -1438.967
-log10(exp(pval))
[1] 624.9355
~~~

Thus, the p-value is $p<10^{-624}$. The evidence against $H_0$ is very strong, and we can conclude that there is genetic variance in this data set.
