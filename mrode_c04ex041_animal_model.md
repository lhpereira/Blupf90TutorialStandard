---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Animal model
============

Model
-----

<!--
Mrode (2014) considered the pre-weaning gain (WWG) as a target trait and applied the following
linear model to the trait.
$$
y_{ij} = p_i + a_j + e_{ij}
$$
where $y_{ij}$ is the observation of j-th calf of the i-th sex, $p_{i}$ is the fixed effect of the i-th sex, $a_j$ is the
random effect (additive genetic effect) of the j-th calf and $e_{ij}$ is the random error effect. Assume
that the genetic variance was $\sigma_a^2$ = 20 and the residual variance was $\sigma_e^2= 40$. This is a typical
animal model which have been already introduced in the previous chapter.
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z} + \mathbf{A}^{-1}/\sigma_a^{2}
\end{array}
\right]
\left[
\begin{array}{c}
\mathbf{\hat{b}}\\
\mathbf{\hat{u}}
\end{array}
\right]
=
\left[
\begin{array}{l}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{y} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{y}
\end{array}
\right]
$$
where $\mathbf{R} = \mathbf{I}\sigma_e^2$. BLUPF90 always create the equation with this formula. The textbook, however,
creates a simplified version of the equations with a variance ratio $\alpha=\sigma_e^2/\sigma_a^2$.
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{X} & \mathbf{X}'\mathbf{Z} \\
\mathbf{Z}'\mathbf{X} & \mathbf{Z}'\mathbf{Z} + \alpha\mathbf{A}^{-1}
\end{array}
\right]
\left[
\begin{array}{c}
\mathbf{\hat{b}}\\
\mathbf{\hat{u}}
\end{array}
\right]
=
\left[
\begin{array}{l}
\mathbf{X}'\mathbf{y} \\
\mathbf{Z}'\mathbf{y}
\end{array}
\right]
$$
The two types of equations provide the identical solutions. In this section, first we build the equation
in a regular way with BLUPF90 (the first equation). Then we will demonstrate that BLUPF90 can
create the second type of equations.
-->

Mrode and Pocrnic (2023) considered pre-weaning gain (WWG) a target trait and applied a linear mixed model with a fixed effect (sex), a random effect (animal), and a residual effect. Assume that the genetic variance was $\sigma_a^2 = 20$ and the residual variance was $\sigma_e^2 = 40$. This is a typical animal model that has already been introduced in the previous chapter.

There are two possible systems of equations: 1) each variance component is explicitly included in the equations, or 2) $\alpha$, the variance ratio, is used. The two systems of equations should produce the same solutions.

Files
-----

<!--
~~~~~{language=text caption="mr41.dat"}
 4 1 1 0 4.5
 5 2 3 2 2.9
 6 2 1 2 3.9
 7 1 4 5 3.5
 8 1 3 6 5.0
~~~~~
-->

We now prepare the data file (`mr41.dat`). It contains 5 observations from 5 animals.
The data file has 5 columns as follows. This is the exact copy of the original table in the textbook.

1. Animal ID (calves)
2. Sex (1 for male and 2 for female)
3. Sire ID
4. Dam ID
5. Observations (WWG, kg)

Columns 3 and 4 are not actually used in this analysis.

A pedigree file (`mr41.ped`) is also prepared. The first column is the animal ID, the second column is the sire ID, and the third column is the dam ID.

<!--
~~~~~{language=text caption="mr41.ped"}
1 0 0
2 0 0
3 0 0
4 1 0
5 3 2
6 1 2
7 4 5
8 3 6
~~~~~
-->

The parameter file is as follows. To obtain the exact solutions, we include `OPTION solv_method FSPAK`. With this option, we can calculate the reliability of a solution using the diagonal elements of the inverse of the left-hand side of the mixed model equations, which are prediction error variances (PEVs). The additional option `OPTION sol se` is needed to calculate PEVs.

~~~~~{language=blupf90 caption="mr41.par"}
DATAFILE
mr41.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
2 2 cross
1 8 cross
RANDOM_RESIDUAL VALUES
40.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr41.ped
(CO)VARIANCES
20.0
OPTION solv_method FSPAK
OPTION sol se
~~~~~

Note that the parameter file stored in the GitHub repository does not have `OPTION sol se`, but this option is needed to reproduce the numbers shown in the textbook.

Solutions
---------

When we invoke BLUPF90 with the above parameter file, we immediately see the solutions in the file `solutions`.

~~~~~{language=text caption="solutions"}
trait/effect level  solution          s.e.
   1   1         1          4.35850233          4.88082357
   1   1         2          3.40443010          5.66554023
   1   2         1          0.09844458          4.34094096
   1   2         2         -0.01877010          4.43664612
   1   2         3         -0.04108420          4.27297922
   1   2         4         -0.00866312          4.13608581
   1   2         5         -0.18573210          4.13814812
   1   2         6          0.17687209          4.20610397
   1   2         7         -0.24945855          4.20407502
   1   2         8          0.18261469          4.11029997
~~~~~

The solutions are identical to those shown in the textbook (p. 39). The above `s.e.` is the square root of a diagonal element of the inverse of the left-hand side. Note that the above `s.e.` is actually SEP (standard error of prediction) in the textbook (p. 73), not PEV. This occurs because BLUPF90 creates general mixed model equations that explicitly contain $\sigma_e^2$ and $\sigma_a^2$, rather than the variance ratio $\alpha=\sigma_e^2/\sigma_a^2$. Therefore, we do not need to multiply the inverse element by an additional $\sigma_e^2$ to obtain PEV. Below, we will demonstrate the same left-hand side shown in the textbook and confirm that its inverse equals the results in the textbook.

Direct calculation of accuracy
------------------------------

BLUPF90+ can calculate the reliability of an estimated breeding value. The reliability is defined as $1-\mathrm{PEV}_i/\left[(1+F_i)\sigma_a^2\right]$, where $\mathrm{PEV}_i$ is the PEV and $F_i$ is the inbreeding coefficient of animal $i$. The following option is supported.

~~~~~{language=blupf90}
OPTION store_accuracy 2
~~~~~

Note that, contrary to its name, this option calculates the reliability (squared accuracy, $r^2$) of an estimated breeding value.
The number `2` above specifies the effect number, that is, the position of the target effect for the reliability computation. In this case, the second effect is the additive effect, so `2` is appropriate for this option.

This option creates two new files: `diag_nrm` for the inbreeding coefficients (the diagonal elements of the numerator relationship matrix) and `acc_bf90` for the solutions and reliabilities. The latter is shown below.

~~~~~{language=text caption="solutions"}
trait/effect level  solution acc
   1   2         1          0.09844458          0.05781158
   1   2         2         -0.01877010          0.01580856
   1   2         3         -0.04108420          0.08708243
   1   2         4         -0.00866312          0.14463969
   1   2         5         -0.18573210          0.14378651
   1   2         6          0.17687209          0.11543447
   1   2         7         -0.24945855          0.11628765
   1   2         8          0.18261469          0.15527171
~~~~~

It contains the solutions (estimated breeding values) and their reliabilities for the specified effect.
These reliabilities are comparable with the numbers in the textbook (p. 73).


Alternative parameter file
--------------------------

The textbook uses a simplified form of the mixed model equations in the single-trait analysis. BLUPF90 can handle this form in a tricky (not recommended) way. In this form, only the variance ratio matters. The ratio is $\alpha=\sigma_e^2/\sigma_a^2=2.0$, which is equivalent to assuming $\sigma_e^2=1.0$ and $\sigma_a^2=0.5$. The parameter file has these "equivalent" variance components.

This parameter file is not shown in the GitHub repository.

~~~~~{language=blupf90 caption="mr41a.par"}
DATAFILE
mr41.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
2 2 cross
1 8 cross
RANDOM_RESIDUAL VALUES
1.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr41.ped
(CO)VARIANCES
0.5
OPTION solv_method FSPAK
OPTION sol se
OPTION store_accuracy 2
~~~~~

The solutions are the following.

~~~~~{language=text caption="solutions"}
trait/effect level  solution          s.e.
   1   1         1          4.35850233          0.77172597
   1   1         2          3.40443010          0.89580057
   1   2         1          0.09844458          0.68636303
   1   2         2         -0.01877010          0.70149535
   1   2         3         -0.04108420          0.67561734
   1   2         4         -0.00866312          0.65397259
   1   2         5         -0.18573210          0.65429867
   1   2         6          0.17687209          0.66504343
   1   2         7         -0.24945855          0.66472263
   1   2         8          0.18261469          0.64989549
~~~~~

This parameter file will provide the same solutions as before. An advantage of this method is that it may reduce numerical error because $\sigma_e^2=1$, which avoids adding scaling to the equations. We square the `s.e.` to obtain a diagonal element of the inverse of the left-hand-side matrix. You can calculate the reliability or accuracy of the estimated breeding value by hand.

For example, the diagonal element for animal 1 is $d_1 = 0.68636303^2 \approx 0.471$, which is the same as the reference value in the textbook (p. 73). The $\mathrm{PEV}$ for animal 1 is $\mathrm{PEV}_1 = d_1\sigma_e^2 = 0.471 \times 40 = 18.84$. The standard error of prediction is $\mathrm{SEP}_1 = \sqrt{\mathrm{PEV}_1} = \sqrt{18.84} = 4.341$, which is also the same as in the textbook. The reliability for animal 1 is $r^2 = 1 - \mathrm{SEP}_1^2/\sigma_a^2 = 1 - \mathrm{PEV}_1/\sigma_a^2 = 1 - 18.84/20 = 0.058$. If the animal has a nonzero inbreeding coefficient, you must account for it when calculating reliability or accuracy. You can confirm that this rule also applies to the other animals.
