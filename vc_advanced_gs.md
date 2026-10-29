---
title: Variance component estimation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Advanced features in Gibbs sampling programs
============================================

Heterogeneous residual variances
--------------------------------

GIBBSF90+ supports heterogeneous residual variances defined by class. Here, we demonstrate an analysis with heterogeneous residual variances.

### Files ###

We use the same data and pedigree file as before.

- [`simdata2.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/adv_gibbs/simdata2.txt) : data file
- [`simped2.txt`](https://github.com/masuday/data/blob/master/tutorial/examples/variance_components/adv_gibbs/simped2.txt) : pedigree file

The pedigree file contains three columns: animal, sire, and dam. The data file has 14 columns, as described below.

Column Item          type      description
------ ---------     -------   ----------------------------------
1      Animal ID     integer   Same as in pedigree (4641 animals)
2      Sire ID       integer   Same as in pedigree
3      Dam ID        integer   Same as in pedigree
4      Weight        real      Not used here
5      Mu            integer   All 1: not used here
6      Farm          integer   Class effect (155 levels)
7      Sex           integer   Class effect (2 levels)
8      Year          integer   Class effect (11 levels)
9      Obs. 1        real      Phenotype for trait 1
10     Obs. 2        real      Phenotype for trait 2
11     Obs. 3        real      Phenotype for trait 3
12     Obs. 4        real      Phenotype for trait 4
13     Covariate     real      Not used
14     Class         integer   Heterogeneous residual class

The 14th column contains the heterogeneous-residual-variance class with 3 levels, which is used in this section.

The parameter file has one option to define the heterogeneous residual class.

~~~~~{language=blupf90 caption="gibbs2.txt"}
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
OPTION hetres_int 14 3
~~~~~

The option has two arguments.

- `OPTION hetres_int` = defines the heterogeneous residual class with two values: 1) the position of the class in the data file and 2) the maximum level of the class.

In this case, we specify that the 14th column has 3 levels.

Using 20,000 samples, saving every 10th sample, and discarding 10,000 samples as burn-in, we obtain the following results.

~~~~~{language=output}
 ave G
   39.845
 SD G
   3.9917
 ave R
   72.313
 SD R
   3.9497
 ave R
   78.248
 SD R
   4.4558
 ave R
   80.646
 SD R
   4.3320
~~~~~

There are 3 `ave_R` (and `SD_R`) blocks. The first corresponds to the variance for heterogeneous level 1, the second to level 2, and so on. Compare the above estimates with the results from BLUPF90+ shown in the previous section.


Restart the sampling
--------------------

After sampling, you may find that more samples are needed. GIBBSF90+ supports restarting the sampling from the endpoint of the previous run.

When GIBBSF90+ finishes sampling, four files are created. All of these files are required to continue sampling.

- `binary_final_solutions` = Posterior means and SDs for location parameters
- `last_solutions` = The last samples for location parameters
- `fort.99` = Values needed for DIC
- `gibbs_samples` = Sampled variance components

If you want to restart sampling, you must add an option to the parameter file. This option requires the number of samples drawn in the previous run. If you had 10,000 samples, the option should be

~~~~~{language=blupf90}
OPTION cont 10000
~~~~~

where 10000 is the number of samples obtained previously. Run the Gibbs sampler with a parameter file containing the above option, and the program will restart sampling from 10001 in this example. GIBBSF90+ will ask you for the number of samples, burn-in, and interval; it is recommended that you enter the following values.

- The number of samples: the additional samples you want to generate in this run.
- The number of burn-in samples: the number you entered in the previous run.
- Interval: the number you entered in the previous run.


Extraction of solutions from binary final solutions
---------------------------------------------------

The file `binary_final_solutions` contains posterior means for location parameters (that is, solutions for fixed or random effects). This file is saved in a non-text format. The following Fortran program can extract the solutions and print them to a file `final_solutions.txt`. You can compile the program using a Fortran compiler such as gfortran and run it in a directory containing the binary final solutions. The output format is equivalent to that of BLUPF90+ with `OPTION sol se`.

~~~~~{language=Fortran caption="\url{binsol_to_textsol.f90}"}
program binsol_to_textsol
	implicit none
	integer :: io, t, e, l
	double precision :: v, sol, se
	open(10, file='binary_final_solutions', form='unformatted', &
		status='old', iostat=io)
	if(io /= 0) stop
	open(20, file='final_solutions.txt')
	write(20,'(" trait / effect level solution               s.e.")')
	do
		read(10, iostat=io) t,e,l,sol,se
		if(io /= 0) exit
		write(20, '(2i4,i10,2f20.8)') t,e,l,sol,se
	end do
	close(10)
	close(20)
end program binsol_to_textsol
~~~~~

Output of POSTGIBBSF90
----------------------

The POSTGIBBSF90 program provides many diagnostics for the Gibbs samples. Here, we give a brief description of the information provided.

- `MCE` = Monte Carlo error, corresponding to the "standard error" of the posterior mean of a parameter ($\hat{\mu}-\mu$).
- `Mean` = Posterior mean of a parameter.
- `HPD` = Highest posterior density interval at 95%, similar to a "95% confidence interval" in the frequentist approach.
- `Effective sample size` = Number of samples after accounting for autocorrelation among samples.
- `Median` = Posterior median of a parameter.
- `Mode` = Posterior mode of a parameter; this is only an approximation.
- `Independent chain size` = Effective number of independent samples.
- `PSD` = Posterior standard deviation of a parameter.
- `Mean` = the same as the mean described above.
- `PSD Interval (95%)` = Lower and upper bounds of $\operatorname{Mean} \pm 1.96\,\operatorname{PSD}$.
- Geweke diagnostic = Convergence diagnostic; values below $1.0$ may indicate convergence (according to the official manual, this diagnostic is almost useless because it is below $1.0$ in almost all cases).
- `Autocorrelations` = Lag correlations at lags 1, 10, and 50, calculated for the saved samples.
- `Independent # batches` = The effective number of blocks after accounting for autocorrelation among samples.
