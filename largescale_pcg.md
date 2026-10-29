---
title: Large-scale genetic evaluation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Iteration on data with preconditioned conjugate gradient (PCG)
==============================================================

Algorithm
---------

Preconditioned conjugate gradient (PCG) is an iterative method for solving linear equations. This method is well suited to the "iteration-on-data" technique. The intermediate state is stored in four vectors, and one iteration is performed by updating these vectors. BLUP90IOD3 is a program that implements this algorithm. Here, we introduce the basic ideas needed to understand how the program works. See Stranden and Lidauer (2000) and Tsuruta et al. (2001) for the detailed algorithm.

The mixed model equations can be written as
$$
\mathbf{Cx} = \mathbf{b}
$$
where $\mathbf{C}$ is the left-hand-side matrix, $\mathbf{x}$ is the solution vector, and $\mathbf{b}$ is the right-hand-side vector. If we have a matrix $\mathbf{M}$ that approximates $\mathbf{C}$, the equations above are equivalent to
$$
\mathbf{M}^{-1}\mathbf{Cx} = \mathbf{M}^{-1}\mathbf{b}.
$$
This matrix $\mathbf{M}$ is called the preconditioner. If $\mathbf{M}=\mathbf{C}$, the equations are solved immediately. BLUPF90 uses $\mathbf{M} = \mathrm{diag}(\mathbf{C})$, so its inverse is easily calculated.


The residual is defined as
$$
\mathbf{r} = \mathbf{b} - \mathbf{Cx}
$$
and the algorithm attempts to reduce it using quantities calculated from the residual. The convergence criterion for the current solution $\hat{\mathbf{x}}$ is
$$
\varepsilon = \frac{||\mathbf{b} - \mathbf{C}\hat{\mathbf{x}}||^2}{||\mathbf{b}||^2}
$$
where $||\cdot||$ denotes the norm. The default value of $\varepsilon$ in BLUPF90IOD2 is $10^{-12}$.

Note that some other software, such as MiX99, uses $\sqrt{\varepsilon}$. Users should be careful about the definition of the convergence criterion when switching to BLUPF90IOD3.

This section was originally written for the previous version, BLUP90IOD2. The author believes that the content is still applicable to BLUP90IOD3.

Programs
--------

BLUP90IOD3 is the current program for performing iteration on data with PCG. It combines CBLUP90IOD2 (for a threshold model or threshold-linear models with one threshold trait), BLUP90MBE (for multibreed models with external information; Legarra et al., 2007), and other programs. BLUP90IOD3 also implements a general multivariate model with both threshold and linear traits (Lourenco et al., 2026).

In this example, we use the former version, BLUP90IOD2.

<!--
A parallel version of BLUPF90IOD2 is now available. BLUP90IOD2OMP1 is a program supporting parallel processing in reading data and pedigree files using OpenMP. This program is useful especially for very large data set with a complicated model (like the multiple-trait model). There is no advantage to use this program for small or moderate data set. The usage of this program is the same as BLUP90IOD2.
-->

Files and analysis
------------------

Here, we will use the same sample files as those used for REML estimation.

- [`simdata.txt`](https://github.com/Masuday/data/blob/master/tutorial/simdata.txt) : data file
- [`simped.txt`](https://github.com/Masuday/data/blob/master/tutorial/simped.txt) : pedigree file

We will apply a 4-trait animal model to this data set with the following parameter file.

~~~~~{language=blupf90 caption="iodparam1.txt"}
DATAFILE
simdata.txt
NUMBER_OF_TRAITS
4
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
9 10 11 12
WEIGHT(S)

EFFECTS:
6 6 6 6   155 cross
7 7 7 7     2 cross
8 8 8 8    11 cross
1 1 1 1  4641 cross
RANDOM_RESIDUAL VALUES
 63.568  35.276  26.535  13.533
 35.276  84.627  37.831  23.306
 26.535  37.831  75.156  28.079
 13.533  23.306  28.079  46.839
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
simped.txt
(CO)VARIANCES
 37.150  19.471  23.885  24.246
 19.471  16.128  19.571  22.239
 23.885  19.571  31.315  33.782
 24.246  22.239  33.782  51.706
~~~~~

Run BLUP90IOD2 with the parameter file. It requires 185 rounds to meet the convergence criterion.

~~~~~{language=output}
round =   183  eps =  0.1707E-11  time =    0.01
round =   184  eps =  0.1139E-11  time =    0.01
round =   185  eps =  0.9976E-12  time =    0.01
  7.1286485E-03 seconds per round
 * END iteration: 11-14-2016  17h 52m 05s 737
 solutions stored in file: "solutions"
~~~~~

Options
-------

~~~~~{language=blupf90}
OPTION conv_crit tol
~~~~~

This option defines the convergence criterion ($\varepsilon$) for stopping the iterations. The real value `tol` should be small. The default is `1.0E-12`. The criterion should be chosen carefully: the default value may be too loose, whereas a strict criterion may require too many rounds to converge. The best practice is to compare solutions obtained with different convergence criteria and determine a criterion that provides sufficient convergence.

~~~~~{language=blupf90}
OPTION blksize n
~~~~~

This option creates a block-diagonal preconditioner matrix ($\mathbf{M}$). The integer value `n` defines the block size. By default, the preconditioner is a diagonal matrix (that is, `n` is 1). The block size should be the same as the number of traits. This option reduces the number of iterations if the specified value is appropriate.
