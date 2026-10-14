---
title: Variance component estimation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Gibbs sampling and post-Gibbs analysis
======================================

Gibbs sampling software
-----------------------

<!--
The BLUPF90 family provides a single program, GIBBSF90+, for conducting Gibbs sampling for linear mixed models and threshold models. This software is currently recommended for all users. However, several Gibbs sampling programs were previously available in the BLUPF90 family. The following description is provided for reference.

In the past, users were encouraged to use GIBBS2F90 or THRGIBBS1F90 for linear mixed models. THRGIBBS1F90 supported both threshold and linear models and was typically faster than GIBBS2F90, especially for large datasets. GIBBS3F90 was useful in cases where heterogeneous residual variances by class were assumed. These programs differed in their capabilities and performance.

**Notice**: This tutorial was originally written using the older software, and THRGIBBS1F90 is used throughout the examples, even though threshold models are not applied. If you are using GIBBSF90+, you can simply replace THRGIBBS1F90 with GIBBSF90+ without any additional options.
-->

Gibbs sampling is often used for variance component estimation, although it is a more general statistical algorithm for parameter estimation. Gibbs sampling is completely different from REML, both conceptually and computationally. We do not discuss the theoretical background of this method here; see Sorensen and Gianola (2002) or Misztal (2008) for details. Roughly speaking, Gibbs sampling draws samples from the posterior distribution by generating random numbers based on the information available at a specific point. In one round, the Gibbs sampler solves the mixed model equations with the current variance components and adds a small random number (noise) to each solution. Then, the sampler generates a variance estimate as a random number given the solutions and the other variance components. A single sample of solutions and variance components from one round is not informative. By repeating this process over many rounds and collecting many samples, we can estimate the posterior distribution of a variance component, for example, as a histogram of the samples. You can calculate the average of the samples (the posterior mean) as a point estimator of a variance component. You can also calculate the standard deviation of the samples (the posterior SD), which corresponds to the standard error in a frequentist approach (for example, REML).

GIBBSF90+ is the Gibbs sampler in the BLUPF90 family. This program assumes, by default, noninformative (also known as vague or flat) priors for location parameters as well as variance components. You can control the strength of the prior distribution for each variance component using an option in the parameter file.

Gibbs sampling requires post-Gibbs analysis. This can be done using the separate software POSTGIBBSF90 or by processing the samples manually. The basic workflow of a Gibbs sampling analysis is as follows:

1. use GIBBSF90+ to generate samples for variance components and
2. use POSTGIBBSF90 to summarize the samples.

Both programs accept the same parameter file as BLUPF90+, but ask additional questions to control the sampling details.


Preparation
-----------

In this section, we will use the same simulated files that were introduced in the previous section for REML.

- [`simdata.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/gibbs/simdata.txt) : data file
- [`simped.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/gibbs/simped.txt) : pedigree file

The parameter file is the same as the one shown in the previous section for AI REML. Here, we give this file a different name.

~~~~~{language=blupf90 caption="gibbs1.txt"}
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
 6     155 cross
 7     2 cross
 8     11 cross
 1     4641 cross
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
~~~~~

The variance components in the parameter file will be used as initial values, but their impact on the results will be small. Therefore, you can use any values in the parameter file.


Sampling strategy
-----------------

When you invoke GIBBSF90+, the program asks for the name of the parameter file.

~~~~~{language=output}
 name of parameter file?
~~~~~

After you type the name, the program asks another question.

~~~~~{language=output}
 number of samples and length of burn-in
~~~~~

You should enter two integers here.

- The total number of samples to be generated.
- The number of burn-in samples, which are the first samples to be discarded.

The two numbers should be separated by a space or a newline (that is, the Enter key). The number of burn-in samples should be smaller than the total number of samples. If you are not sure how many burn-in samples to use, enter 0. After sampling, you can manually discard any number of burn-in samples using POSTGIBBSF90. If you generate 5,000 samples and discard the first 1,000 samples as burn-in, you can enter the following (no commas are allowed).

~~~~~{language=shell}
5000 1000
~~~~~

You will be asked one more question.

~~~~~{language=output}
 Give n to store every n-th sample ? (1 means store all samples )
~~~~~

You should enter one integer here.

- The interval at which to save the samples.

For example, if you want to save a sample every 10 rounds, enter 10 here.

~~~~~{language=shell}
 10
~~~~~

If you want to save all the samples, type `1` instead. After this, the program starts sampling.

What is the appropriate number of samples? What is the best interval? The answer depends on the model and data. We will consider these questions later in this section.


Constraints on variance components
----------------------------------

As explained previously in the AI REML section, the program supports forcing some variance components to be 0. When the initial variance component is 0, this component never changes throughout the iterations. See the previous section for an example of initial values equal to 0.

There is a difference between BLUPF90+ and GIBBSF90+ in how constraints are handled. Whereas BLUPF90+ never computes estimates for variance components specified as fixed, the Gibbs sampler actually calculates all variance components and then replaces the constrained parameters with 0. This means that the Gibbs sampler has the same computational cost with constraints as without constraints.

Like BLUPF90+, GIBBSF90+ tries to correct estimated variance components that are outside the parameter space. During this process, some variances may be set to 0 numerically. As with BLUPF90+, once this happens, such variances may never return to nonzero values simply because no information remains to estimate the variance parameters. If you see such accidental zeros during the iterations, your model may be inadequate, or the amount of data may be insufficient to estimate the variance components.

Results shown on screen
-----------------------

### Intermediate results ###

During the sampling, the program always prints the sampled variances in each round.

~~~~~{language=output}
        5000  rounds
 G
   36.265
 R
   61.996
~~~~~

### Final posterior mean and SD ###

The program also calculates samples of the solutions in each round. At the end of sampling, it shows several statistics on the screen. You do not always need all the information, so we break it down here.

~~~~~{language=output}
 cpu time per iteration          1.5550610E-03  : total   7.775305
 wall-clock time per iteration   1.4231800E-03  : total   7.115900
 * End of iteration 07-29-2026  21h 16m 26s 587
 ave G
   39.035
 SD G
   3.5789
 ave R
   62.582
 SD R
   2.5062
~~~~~

- `ave G and SD G`: Posterior mean and SD for $\sigma_u^2$. Average and standard deviation of saved samples after removing burn-in.
- `ave R and SD R`: Posterior mean and SD for $\sigma_e^2$. Average and standard deviation of saved samples after removing burn-in.

The number of burn-in samples was specified at the beginning of the program run.


### Final status of sampling ###

These are not essential for a regular analysis.

~~~~~{language=output}
 * Last seeds =  2101474043  1031647242
 * Number of samples kept =        4000
 solutions stored in binary file: "last_solutions"
~~~~~

- `Last seeds`: Seeds of random number generator. Usually, the values are not crucial.
- `Number of samples kept`: The total number of samples after burn-in.
- `solutions stored in a binary file: "last_solutions"`: The last sampled solutions are stored in `last_solutions`. Usually, the file is not essential.


### DIC and related information ###

DIC (Deviance Information Criterion) can be used to compare two or more models, as with AIC (smaller is better). In each round, the program calculates partial information to form the DIC. In general notation, we can write a likelihood function as $\log L(\theta)$ with parameters $\theta$. In the above model, assume that $N$ is the number of observations and $\hat{\sigma}_{e[k]}^2$ (that is, $\hat{\theta}_k$) is the sampled residual variance in round $k$ after burn-in. The deviance $D_k$ in round $k$ is defined as
$$
D_k = -2\log L(\hat{\theta}_k) = -2\sum_{i=1}^{N}\left[C+\frac{\hat{e}_i^2}{\hat{\sigma}_{e[k]}^2}\right]
$$
where $C$ is $\log(2\pi\hat{\sigma}_{e[k]}^2)$ and $\hat{e}_i$ is an observed residual for observation $i$ (that is, $y - \hat{y}$: the observation minus the sampled solution). With $m$ samples after burn-in, $\bar{D}$, an estimate of the expectation of $D$, is defined as
$$
\bar{D}=\frac{\sum_{k=1}^{m}D_k}{m}
$$
Similar statistics are also calculated using the posterior mean of $\sigma_e^2$. We calculate the posterior deviance $D(\bar{\theta})$ using the above formula for $D_k$, but replace $\hat{\sigma}_{e[k]}^2$ (that is, $\hat{\theta}_k$) with its posterior mean $\bar{\sigma}_{e}^2$, that is, $\bar{\theta}$, as follows:
$$
D(\bar{\theta})=-2\log L(\bar{\theta})=-2\sum_{i=1}^{N}\left[\bar{C}+\frac{\hat{e}_i^2}{\bar{\sigma}_{e}^2}\right]
$$
where $\bar{C}=\log(2\pi\bar{\sigma}_{e}^2)$.
These statistics are calculated using the samples stored after burn-in. GIBBSF90+ prints the following quantity as DIC.
$$
\mathrm{DIC}=2\bar{D}-D(\bar{\theta})
$$
GIBBSF90+ saves every $D_k$ to a file `fort.99`.

The output corresponds to the formulas above.

~~~~~{language=output}
****************************************
 ***************** DIC ******************
 ****************************************
 detR   62.5820466970184
 # stored samples         400
 D-bar   32364.1379063698
 D(theta-bar)   30968.7777134920
 DIC = 2*D-bar - D(theta-bar) =   33759.4980992475
 Effective number of parameters =   1395.36019287776
 solutions stored in file: "binary_final_solutions"
 *** End program *** 07-29-2026  21h 16m 26s 597
~~~~~

- `detR`: Determinant of posterior mean of the residual covariance matrix ($|\mathbf{R}_{0}|$ in the general case and $|\sigma_e^2|$ in this case).
- `# stored samples`: The total number of samples saved in a file ($m$ above).
- `D-bar`: $\bar{D}$
- `D(theta-bar)`: $D(\bar{\theta})$
- `DIC = 2*D-bar - D(theta-bar)`: $2\bar{D}-D(\bar{\theta})$
- `Effective number of parameters`: $\bar{D} - D(\bar{\theta})$
- `solutions stored in file: "binary_final_solutions"`: posterior means for sampled solutions after burn-in.

The estimated genetic variance is $\hat{\sigma}_u^2 = 39.03 (\pm 3.579)$, and the estimated residual variance is $\hat{\sigma}_e^2 = 62.58 (\pm 2.506)$. If you believe that the numbers of samples and burn-in samples are appropriate, you can use the above posterior means and SDs as the final results. Otherwise—and in most cases—you should run POSTGIBBSF90 to perform the post-Gibbs analyses and confirm that the numbers of samples and burn-in samples were sufficient to estimate posterior statistics for the variance parameters.


Generated files
---------------

GIBBSF90+ generates several files, including the following:

- `gibbs_samples`: Samples of variance components in specific rounds after burn-in.
- `fort.99`: Deviance in specific rounds after burn-in.
- `last_solutions`: The last sampled solutions saved in a binary form.
- `binary_final_solutions`: Posterior means for sampled solutions from specific rounds after burn-in saved in binary form.

The samples and calculated statistics are based on samples from every $n$th round after burn-in, where $n$ is the interval that you enter at the beginning of the program.

The file `gibbs_samples` is used by POSTGIBBSF90 to perform post-Gibbs analyses. The number of samples stored in this file depends on your answers to the initial questions. For example, using 5,000 samples with 1,000 burn-in samples and a sampling interval of 10, the file will contain 400 samples, which is $(5000 - 1000)/10$. The second file, `fort.99`, is optional and is used in the post-Gibbs analyses. The `last_solutions` file is useful for continuing the analysis from the last round; you do not usually need to inspect its contents. The last file, `binary_final_solutions`, contains samples of solutions for fixed and random effects. You may want to inspect these solutions, but the file is not a text file. In the next section, we will consider advanced techniques, including extracting solutions from this file.


Post Gibbs analyses
-------------------

### Invoking POSTGIBBSF90 ###

POSTGIBBSF90 performs post-Gibbs analyses by reading the parameter file, `gibbs_samples`, and `fort.99`. You have to run POSTGIBBSF90 in a directory containing these files. When you invoke the program, it prints the following message.

~~~~~{language=output}
 name of parameter file?
~~~~~

Type the name of the parameter file used in this analysis. Then, the program checks whether `gibbs_samples` was generated from that parameter file. If successful, you will see information about the total number of saved samples. POSTGIBBSF90 then shows the number of samples and asks another question.

~~~~~{language=output}
  # parameters in gibbs_samples =           2
 Read         400  samples from round        1010  to        5000
 Burn-in?
~~~~~

This is a tricky question. You should *not* enter the same number as you entered at the beginning of GIBBSF90+. The burn-in samples have already been discarded. This question asks for the number of *additional* samples to discard as burn-in. The answer is

- `0` if you do not discard any more samples as burn-in;
- any integer if you need to discard additional samples as burn-in.

After this, you will be asked the last question.

~~~~~{language=output}
 Give n to read every n-th sample? (1 means read all samples)
~~~~~

Before typing `1` to keep all the samples, please wait for a moment. This question asks for the thinning interval that you entered when running GIBBSF90+. If you want to keep all the samples you have drawn, you must enter the same number as before. Alternatively, you can enter a multiple of the original number. For example, in this case, you can enter 10, 20, 30, and so on because the original interval was 10. If you enter an inappropriate number, the program will stop with a suspicious message.

Post statistics
---------------

We will see the following table with many statistics when we enter `0` as the additional burn-in and `10` as the thinning interval in our example.

~~~~~{language=widetext}
# samples after burn-in =         400
# samples after burn-in used =         400

                          ********   Monte Carlo Error by Time Series   ********
 Pos. eff1 eff2 trt1 trt2        MCE        Mean             HPD             Effective       Median        Mode      Independent
                                                        Interval (95%)       sample size                             chain size
   1    4    4    1    1      0.36091       39.035       32.030       45.270         98.1         39.060       39.175            6
   2    0    0    1    1      0.19923       62.582       57.640       67.440        157.8         62.640       63.362            4

                          ********   Posterior Standard Deviation   ********
 Pos. eff1 eff2 trt1 trt2        PSD        Mean             PSD             Geweke            Autocorrelations      Independent
                                                        Interval (95%)       diagnostic   lag: 1       10       50    # batches
   1    4    4    1    1       3.5788       39.035       32.020       46.049        -0.09      0.660   -0.076    0.017        66
   2    0    0    1    1       2.5062       62.582       57.670       67.494         0.09      0.406   -0.023    0.070       100
~~~~~

The output has two main tables. Each table contains statistics for each variance component. Variance components can be identified using the columns `Pos. eff1 eff2 trt1 trt2`.

- `Pos.` is the position index of a parameter.
- `eff1` and `eff2` are the effects for which the variance is defined. Without a maternal genetic effect or random regressions (i.e., correlated random effects), `eff1` equals `eff2`.
- `trt1` and `trt2` are corresponding traits. In a single-trait model, `trt1` equals `trt2`.

In this case, we have two variances: one ($\sigma_{u}^{2}$) is the genetic variance defined for the 4th effect, and the other is the residual variance ($\sigma_e^2$). This is a single-trait model, so the trait number is 1. Finally, we find that position 1 corresponds to $\sigma_u^2$ and position 2 corresponds to $\sigma_e^2$.

As you can see, this program displays a great deal of information. We select only the essential numbers here. The remaining items will be covered in the next section.

- `Mean`: Posterior mean.
- `HPD Interval (95%)`: 95% highest posterior density interval.
- `Median`: Posterior median.
- `Mode`: Posterior mode. It would provide a better estimate than posterior mean if the posterior distribution is highly skewed.
- `Independent chain size`: The "effective" number of samples after adjusting autocorrelations among samples.
- `PSD`: Posterior standard deviation.
- `Autocorrelations`: Lag-correlation between two samples at a specific interval.

This table shows several "point estimates," including the posterior mean, median, and mode. A reasonable estimate depends on the skewness of the posterior distribution. If the distribution is highly skewed and the mean differs substantially from the mode, the posterior mode or median could be a better choice. Note that the mode is only approximated from the 50 interval classes calculated by this program.

The 95% highest posterior density interval (HPD95) is often cited as a statistic for evaluating the accuracy of parameter estimates. This is a type of interval estimate for a parameter. The values are the lower and upper bounds obtained by discarding the largest 2.5% and smallest 2.5% of the sorted samples.

You can evaluate whether the sampling interval is sufficient using the "independent chain size" and "autocorrelations." Two adjacent samples are usually highly correlated because the current sample is drawn based on the previous one. When the correlation remains high between distant samples, the dependence is also high, and successive samples may be very similar. This means that the samples do not provide much information about the posterior distribution. The autocorrelation indicates how many intervals between samples are needed to reduce the correlation substantially. The independent chain size represents the number of effectively independent samples. If the independent chain size is 3, the statistics (that is, the posterior mean and SD) are effectively calculated using only 3 independent samples, which is too small. This indicates whether you need more samples.


### Time-series plot and histogram ###

After showing the above table, the program waits for your input with the following message.

~~~~~{language=output}
 Choose a graph for samples (= 1) or histogram (= 2); or exit (= 0)
~~~~~

POSTGIBBSF90 can draw a time-series plot or histogram of samples. It uses the external software Gnuplot to display the graph. To obtain a plot, you must have Gnuplot installed on your system. You can type `1` to draw a time-series plot or `2` to draw a histogram of the samples. Type `0` to quit the program.

The time-series plot visually indicates whether the burn-in period and total sampling are sufficient. If the initial samples appear to retain their initial values, you need more burn-in. If the samples remain for a long time in a specific value range, the posterior mean may not represent the full posterior distribution, and you should draw more samples. The histogram provides a visual representation of the posterior distribution. Usually, it is unimodal but may be somewhat skewed, with a long right tail. If your distribution has two peaks or is very flat, you should check the parameter file for human error or draw more samples.


Output files from POSTGIBBSF90
------------------------------

POSTGIBBSF90 creates seven files derived from the post-Gibbs statistics described above.

- `postout`: The same content as that shown on the screen.
- `postind`: Relation of a position index to a variance component.
- `postmean`: Posterior means shown in a user-friendly format.
- `postmeanCorr`: Posterior means for correlations shown in a user-friendly format.
- `postsd`: Posterior standard deviations.
- `fort.998`: $\log L$ calculated from the file `fort.99`
- `postgibbs_samples`: The final samples after removing burn-in and applying thinning.

The `postout` file contains detailed statistics from the post-Gibbs analysis. You can check the position index with `postind`. Several files contain posterior means and SDs for variance components. The file `fort.998` contains $\log L$ needed to compute the Bayes factor.

The file `postgibbs_samples` is useful especially if you calculate other statistics by yourself. It contains samples after burn-in. The content looks like the following.

~~~~~{language=text}
1    1010    2   35.60       64.26
2    1020    2   36.98       63.53
3    1030    2   30.92       66.49
4    1040    2   34.93       65.60
5    1050    2   37.38       63.26
~~~~~

The meaning of each column is:

- Column 1: the sequential number for saved samples.
- Column 2: the actual round in which each sample was drawn.
- Column 3: the number of parameters.
- Column 4 or later: actual samples ordered by the position index.


Post-Gibbs Analysis for arbitrary function of variance components
-----------------------------------------------------------------

A frequently asked question is how to compute the posterior distribution of heritability or genetic correlation, each of which is a function of variance components. There are two ways to perform the post-Gibbs analysis for this purpose.

### Using the file postgibbs samples

The first option is simpler if you are familiar with R or Python scripting. From the file `postgibbs_samples`, you can easily draw a graph and calculate additional statistics. For instance, you may compute the heritability at each sample as a function of the sampled variance components. The code in R would be something like this.

~~~r
a=read.table("postgibbs_samples",header=FALSE)
# tell what each thing is
colnames(a)=c("i","iter","varu","vare")
a$h2=a$varu/(a$varu+a$vare)
hist(a$h2)
~~~

In this way, you estimate the posterior distribution of $h^2$ from the samples of the variance components. This procedure can be applied to _any_ function of the variance components, for instance, genetic correlations. More sophisticated analyses can be performed with the R package `boa`.

### Using an option in the parameter file

The second option is to use the option `se_covar_function` in a parameter file. The user can define the function of the variance components on the option line, and POSTGIBBSF90 calculates a sample of the function in each sampling round. The program displays all post-Gibbs statistics on the screen as it does for standard variance components.

The use of this option was explained in the previous section on AI REML. See that section for details.


Remarks
-------

You must not run more than one GIBBSF90+ in the same directory at the same time. One program will overwrite `gibbs_samples` generated by the other program. If you want to share the data and pedigree files across multiple jobs, create a separate directory (or folder) and copy the files there, or create symbolic links to them.

There are several options for GIBBSF90+. For example, one supports continuing sampling from already completed results, starting from the last round of the previous run. Another controls the strength of prior information. Please see the manual for details.

The BLUPF90 family used to include several Gibbs samplers. Those programs have been consolidated into GIBBSF90+. For reference, the differences among the old programs are shown below.

- GIBBSF90: initial program
- GIBBS1F90: much faster for multiple-trait models
- GIBBS2F90: faster convergence for models with correlated effects (for example, maternal or random-regression models).
- GIBBS3F90: derived from GIBBS2; supports heterogeneous residual variances; slightly slower than GIBBS2.
- THRGIBBS1F90: derived from GIBBS2; supports threshold models and large dense matrices; faster than GIBBS2, especially for large datasets; requires more memory than GIBBS2.
- THRGIBBS3F90: derived from THRGIBBS1; supports heterogeneous residual variances.

GIBBSF90+ is always recommended. However, if you have to use the old programs, GIBBS2F90 and THRGIBBS1F90 may be alternatives.

<!--
Actually, recent improvements were done for GIBBS2F90 and THRGIBBS1F90. So we recommend a user to use these two programs if you are not interested in heterogeneous residual variances.
-->

Summary
-------

- GIBBSF90+ can perform Gibbs sampling for both regular linear mixed models and threshold models.
- GIBBSF90+ requires the total number of samples, the number of burn-in samples, and the interval at which samples are saved.
- GIBBSF90+ saves the samples to the file `gibbs_samples`.
- GIBBSF90+ reads `gibbs_samples` and calculates post-Gibbs statistics.
- GIBBSF90+ creates many files containing the statistics.
- You can check from the output whether the initial settings are suitable.
- You can perform post-Gibbs analyses of a function of variance components yourself using the file `postgibbs_samples`, or using the option `se_covar_function` with POSTGIBBSF90.
- If the initial variance component is 0, it will be fixed at 0 throughout the iterations.
