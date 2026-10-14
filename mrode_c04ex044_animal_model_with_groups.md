---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Animal model with unknown parent groups
=======================================

Short introduction to unknown parent groups
-------------------------------------------

Unknown parent groups (also known as genetic groups or phantom parent groups) are among the less well-understood concepts in animal breeding, in the author's opinion. An unknown parent group defines a nonzero mean for breeding values in a base population.

We usually assume that the mean breeding value in a base population is 0. The average breeding value by generation or birth year will increase (or decrease) through selection. If we have a complete pedigree back to the base population, complete phenotypic records used as the selection criterion, and an appropriate model to describe the factors affecting the observations, the mixed model can provide unbiased predictions of breeding values for all animals. However, reality is different. One of the biggest challenges is incomplete pedigree information. Many animals may have no pedigree information in a population undergoing selection. In this case, it is incorrect to assume that unknown parents belong to the base population. Under selection, animals in later generations tend to have significantly higher (or lower) genetic levels, so their parents should have a nonzero mean breeding value.

The groups can partially solve this problem. A group represents a real animal, and we assign the group to unknown parents that are expected to have the same genetic level. A group assigned to parents in a later generation should differ from a group for older parents because their genetic levels differ. In addition to generation (or birth year), a group may also be defined by sex, region of origin, and selection path. Because the group is a proxy for an animal in the base population, every real animal is a descendant of genetic groups.

The groups are typically treated as fixed effects. The expected breeding value of base animals is no longer 0; it is represented by the BLUE for the group effect. Therefore, the final prediction of an animal's breeding value is a combination of the BLUE for the group effect and the BLUP for the breeding value. Despite this complication, the mixed model equations with groups are surprisingly simple and resemble those of the standard animal model. For theoretical background, see the textbook in addition to Westell and Van Vleck (1987), Quaas (1988), and Masuda et al. (2022).

Model
-----

The mixed model with unknown parent groups can be written as follows:
$$
\mathbf{y} = \mathbf{Xb} + \mathbf{ZQg} + \mathbf{Zu} + \mathbf{e}
$$
where $\mathbf{Q}$ is a matrix that relates the groups to descendant animals and $\mathbf{g}$ is the fixed effect for the groups. We are interested in $\mathbf{u^*}=\mathbf{u} + \mathbf{Qg}$, that is, the prediction of an individual's breeding value adjusted for the group effect and accounting for the nonzero expectation of base breeding values. Assuming a single-trait model, the resulting system of mixed model equations is
$$
\left[
\begin{array}{lll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z} & \mathbf{0}\\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z} + \mathbf{A}_{nn}^{-1}/\sigma_a^{2} & \mathbf{A}_{np}^{-1}/\sigma_a^{2}\\
\mathbf{0}&\phantom{\mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z}+\,\,}\mathbf{A}_{pn}^{-1}/\sigma_a^{2} & \mathbf{A}_{pp}^{-1}/\sigma_a^{2}
\end{array}
\right]
\left[
\begin{array}{l}
\mathbf{\hat{b}}\\
\mathbf{\hat{u}^*}
\\
\mathbf{\hat{g}}\\
\end{array}
\right]
=
\left[
\begin{array}{l}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{y} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{y} \\
\mathbf{0}
\end{array}
\right]
$$
and
$$
\mathbf{A}^{-1}
=
\left[
\begin{array}{ll}
\mathbf{A}_{nn}^{-1} & \mathbf{A}_{np}^{-1} \\
\mathbf{A}_{pn}^{-1} & \mathbf{A}_{pp}^{-1}
\end{array}
\right]
$$
where the subscript $n$ in $\mathbf{A}^{-1}$ denotes real animals and the subscript $p$ denotes genetic groups. This $\mathbf{A}^{-1}$ can be calculated using Henderson's rule with a small extension (Quaas, 1988). The BLUPF90 programs support groups in a standard framework. BLUPF90 papers use unknown parent groups (UPG) rather than phantom parents or genetic groups.

Files
-----

We can use the same data file as before (`mr44.dat`). The pedigree file (`mr44a.ped`) must contain group codes rather than 0s for unknown parents. The pedigree file must contain 4 columns.

1. Animal ID (a positive integer).
2. Sire ID, which can contain a group code.
3. Dam ID, which can contain a group code.
4. Missing parent status: 1 = both parents known, 2 = one parent known, and 3 = both parents
   unknown.

~~~~~{language=text caption="mr44a.ped"}
1  9 10  3
2  9 10  3
3  9 10  3
4  1 10  2
5  3  2  1
6  1  2  1
7  4  5  1
8  3  6  1
~~~~~

A group code must be larger than the last ID for real animals. In this example, because we have 8 real animals, codes 9 and 10 are unknown parent group codes. Thus, we have 10 "animals" (8 real animals and 2 UPGs) in the pedigree. The 4th column is needed to calculate the Mendelian sampling variance for each animal in the program.

The parameter file is also similar to the previous one. You should make two changes.

~~~~~{language=blupf90 caption="mr44a.par"}
DATAFILE
mr44a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
2 2 cross
1 10 cross       # change the number of animals to 10 including groups
RANDOM_RESIDUAL VALUES
40.0
RANDOM_GROUP
2
RANDOM_TYPE      # use upg = unknown parent groups
add_an_upg
FILE
mr44a.ped
(CO)VARIANCES
20.0
OPTION solv_method FSPAK
~~~~~

Solutions
---------

The following solutions are obtained with the above files.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1          4.35899605
   1   1         2          3.21434490
   1   2         1          0.33201730
   1   2         2          0.17597293
   1   2         3          0.13611581
   1   2         4         -0.16882873
   1   2         5          0.00000000
   1   2         6          0.37131040
   1   2         7         -0.23933070
   1   2         8          0.33117127
   1   2         9          1.09907877
   1   2        10         -0.66967475
~~~~~

These values look very different from those in the textbook, and there is a clear reason why. The left-hand side of the mixed model equations is not full rank; the groups are treated as fixed, so the model contains two fixed group effects, and the fixed-effect estimates (BLUEs) are not unique. The nonunique BLUEs are combined with estimated breeding values and presented as individual solutions. You need to be careful when interpreting raw breeding-value solutions after adding groups to the pedigree.

To obtain a unique solution, we should calculate a contrast (precisely, an estimable function; here, the difference between solutions), as we do for BLUEs. For example, the difference in breeding values for animals 1 and 2 ($u_1 - u_2$) is 0.156, and it is consistent between this example and the textbook.

In the textbook, the author puts the zero constraint on group 1 (corresponding to animal 9 here), but BLUPF90+ has put the constraint on animal 5 in the above run. BLUPF90+ automatically detects linear dependencies in the equations and imposes a constraint on columns and rows when necessary. To impose a zero constraint on animal 9 in BLUPF90+, replace 9 with 0 in the pedigree file and run BLUPF90+ with the same parameter file. The files in the GitHub repository, such as `mr44.ped`, use this constraint. With this change, you will obtain the following results.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1          5.45807483
   1   1         2          4.31342367
   1   2         1         -0.76706148
   1   2         2         -0.92310585
   1   2         3         -0.96296296
   1   2         4         -1.26790750
   1   2         5         -1.09907877
   1   2         6         -0.72776838
   1   2         7         -1.33840948
   1   2         8         -0.76790750
   1   2         9          0.00000000
   1   2        10         -1.76875353
~~~~~

Be aware that the numerical error is large because of the very small data set. Considering rounding error, those values are similar to those in the textbook.

This analysis cautions us about breeding-value solutions from UPG models. Depending on the method or constraints, we may obtain infinitely many solutions because of a dependency between the group effects and other fixed effects. The raw breeding-value solutions already include the UPG effects, so we should compare the differences between solutions. A better way to express a breeding value is to adjust it relative to the average for animals in a similar condition. For example, in dairy cattle, the estimated breeding value may be adjusted using the average breeding value for cows born in a specific year. Such a reference population is called a genetic base. In a population undergoing selection, the average breeding value continuously increases (or decreases), so the genetic base should be revised every few years.

One more caution is necessary. When using UPG, the square root of the prediction error variance in the solution file (available with `OPTION sol se`) is not valid. This is because the prediction is a combination of BLUE and BLUP ($\mathbf{Qg}+\mathbf{u}$), and the corresponding element of the inverse of the left-hand-side matrix does not give the prediction error variance. If you want the PEV of the breeding value, you must use a workaround: use the non-UPG model to compute the PEV for the breeding values (only for $\mathbf{u}$).
