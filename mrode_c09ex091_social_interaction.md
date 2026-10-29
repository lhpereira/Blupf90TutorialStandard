---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Social interaction effects
==========================

Model
-----

When an animal is raised with a few other animals in a limited space (for example, a pen or cage), social interactions (for example, competition) occur among the animals. The animal's phenotype is affected by its own genetic and environmental effects, as well as by the contributions of its competitors. The competitors' effects appear as environmental contributions to the animal. This structure is similar to that of the maternal animal model introduced in the previous section. Some of the competitors' contributions may come from genes carried by the competitors, while the remainder comes from nongenetic factors. The genetic component can be correlated with the animal's direct genetic effect. A statistical model should therefore include a complicated covariance structure.

In this example, there are 3 pens, and each pen has 3 animals ($n = 3$). See the textbook for the detailed formulation.

The model contains sex as a fixed effect and the direct and associative genetic effects, pen (group), and common environmental (full-sib) effects as random effects. Given the data, the full-sib group can be viewed as the dam group. The author assumes $n = 3$, $\sigma_g^2 = 12.12$ (group variance), $\sigma_e^2 = 60.6$ (the original residual variance), $\sigma_e^{2*} = 60.6 - 12.12 = 48.48$ (the final residual variance), and $\sigma_c^2 = 12.5$ (common environmental variance), and
$$
\mathbf{G}_{0}
=
\left[
\begin{array}{ll}
25.70&2.25\\
2.25&3.60
\end{array}
\right].
$$


Files
-----

Note that the textbook and the files on the GitHub repository made some mistakes. The following descriptions correct the mistakes and give accurate values.

The data set is shown on p. 154 (`mr91a.dat`). We added 3 columns containing competitor IDs.

~~~~~{language=text caption="mr91a.dat"}
 7 1 4 1 1  5.50   8  9  1
 8 1 4 1 2  9.80   7  9  1
 9 2 5 1 2  4.90   7  8  2
10 1 4 2 1  8.23  11 12  1
11 2 5 2 2  7.50  10 12  2
12 3 6 2 2 10.00  10 11  3
13 2 5 3 1  4.50  14 15  2
14 3 6 3 2  8.40  13 15  3
15 3 6 3 1  6.40  13 14  3
~~~~~

Each column has the following information.

   1. Animal ID
   2. Sire ID
   3. Dam ID
   4. Pen (group)
   5. Sex (1 = male and 2 = female)
   6. Growth rate (10$\times$g/day)
   7. ID of competitor 1 in the same pen
   8. ID of competitor 2 in the same pen
   9. Half-sib code (common environment)

The pedigree (`mr91.ped` on the GitHub repository) is derived from the data set.

The model is as follows.
$$
y_{ijklm} = S_{i} + u_{j} + c_{k} + g_{l} + e_{ijklm}
$$
In this model, $y_{ijklm}$ is the phenotype, $S_i$ is the fixed sex effect, $u_j$ is the additive genetic effect, $c_k$ is the common environmental (half-sib family) effect, $g_l$ is the group (pen) effect, and $e_{ijklm}$ is the random residual effect. The parameter file resembles that of a maternal animal model. See the textbook for details, and make sure that the order of effects is correct.

~~~~~{language=text caption="mr91a.par"}
DATAFILE
mr91a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
6
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
 5  2 cross  # sex
 1 15 cross  # direct genetic
 7  0 cross  # associative genetic 1
 8 15 cross  # associative genetic 2
 9  3 cross  # common environmental
 4  3 cross  # random group (pen)
RANDOM_RESIDUAL VALUES
48.48
RANDOM_GROUP
2 3
RANDOM_TYPE
add_animal
FILE
mr91.ped
(CO)VARIANCES
25.7 2.25
2.25 3.60
RANDOM_GROUP
5
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
12.5
RANDOM_GROUP
6
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
12.12
OPTION solv_method FSPAK
~~~~~

The parameter file may look tricky; it uses the same technique found in the reduced animal model. The 3rd effect in `EFFECTS:` is defined with 0 levels. This works together with the 4th effect to put two elements on the same row in the incidence matrix $\mathbf{Z}_S$ (see p. 154). Each statement in `EFFECTS:` can put only one element in a row of the system of mixed model equations. If we define an effect with 0 levels, that effect is not recognized as a new effect but is combined with the next effect. In this case, the 3rd effect is processed, but the offset address is not incremented, so the 4th effect is placed in the same row of $\mathbf{Z}_S$.

We can see what happens when the program processes the data using those two statements (effects 3 and 4). Consider the first line in the data file.

     7   1   4   1    1   5.50    8   9   1

The two statements perform the following operations:

- The program reads the 7th column (the value is 8) and puts 1 (because of the cross effect) in column 8 of the 1st row of $\mathbf{Z}_S$.
- The program reads the 8th column (the value is 9) and puts 1 (because of the cross effect) in column 9 of the 1st row of $\mathbf{Z}_S$.

The resulting row in $\mathbf{Z}_S$ is
$$
\left[
\begin{array}{ccccccccccccccc}
0& 0& 0& 0& 0& 0& 0& 1& 1& 0& 0& 0& 0& 0& 0
\end{array}
\right].
$$

The textbook omits the first 6 columns of $\mathbf{Z}_S$. Thus, the row above is identical to the corresponding row in the textbook. Next, we consider the 2nd line in the data file.

    8     1   4   1   2   9.80     7   9   1

The two statements perform the following operations:

- The program reads the 7th column (the value is 7) and puts 1 (because of the cross effect) in column 7 of the 1st row of $\mathbf{Z}_S$.
- The program reads the 8th column (the value is 9) and puts 1 (because of the cross effect) in column 9 of the 1st row of $\mathbf{Z}_S$.
The resulting row in $\mathbf{Z}_S$ is
$$
\left[
\begin{array}{ccccccccccccccc}
    0& 0& 0& 0& 0& 0& 1& 0& 1& 0& 0& 0& 0& 0& 0
\end{array}
\right].
$$
This is identical to the 2nd row in $\mathbf{Z}_{S}$ in the textbook (again, the first 6 columns have been omitted in the textbook).

The specification of `RANDOM_TYPE` is also tricky. In the first `RANDOM_GROUP`, we specify only effects 2 and 3. Effect 4 should be omitted because it is the same effect as effect 3. Also, BLUPF90 accepts only consecutive effects in `RANDOM_GROUP`, so we put 2 and 3 here.


Solutions
---------

<!--
The solutions are very similar to the values in the textbook (p.126).
-->

The solutions in the textbook seem to be incorrect, probably because of a misspecification of the model in the parameter file, as seen in `mr91.par` in the GitHub repository.
