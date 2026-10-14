---
title: Quick tour of BLUPF90
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Trivial analyses for mixed models
=================================

In this section, we perform mixed-model analyses using BLUPF90+. We start with a simple mixed model with random effects that are unrelated to one another, and finally consider pedigree information.

To perform the simple mixed model (without a relationship matrix), the BLUPF90 programs need only two files (a data file and a parameter file), which are the same as those explained in the previous section. If you consider pedigree information, in addition to the data and parameter files, a pedigree file must be prepared. The same is true for a genomic model: you should prepare a genomic-marker file.

A simple mixed model
--------------------

We assume a model similar to that described in the previous section, except that the second effect is now treated as an independent random effect:
$$
y_{ijk} = A_i + S_j + \beta x_{ijk} + e_{ijk}
$$
where $y_{ijk}$ is an observation, $A_i$ is a fixed effect, $S_j$ is a random effect, $\beta$ is a regression coefficient, $x_{ijk}$ is a covariate, and $e_{ijk}$ is the residual effect. The model in matrix notation is
$$
\mathbf{y} = \mathbf{Xb} + \mathbf{Zu} + \mathbf{e}
$$
where $\mathbf{y}$ is a vector of observations, $\mathbf{b}$ is a vector of fixed effects, $\mathbf{u}$ is a vector of random effects, $\mathbf{e}$ is a vector of residuals, and $\mathbf{X}$ and $\mathbf{Z}$ are incidence matrices. In general, we can write $\mathrm{var}(\mathbf{u}) = \mathbf{G}$^[This $\mathbf{G}$ does not mean the genomic relationship matrix. This is the traditional notation describing a covariance structure of random effects.] and $\mathrm{var}(\mathbf{e}) = \mathbf{R}$, and the mixed model equations are
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z} + \mathbf{G}^{-1}
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
\right].
$$


For simplicity, we assume here that $\mathrm{var}(\mathbf{u}) = \mathbf{I}\sigma_s^2$ and $\mathrm{var}(\mathbf{e}) = \mathbf{I}\sigma_e^2$, so the mixed model equations reduce to
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{X}\sigma_e^{-2} & \mathbf{X}'\mathbf{Z}\sigma_e^{-2} \\
\mathbf{Z}'\mathbf{X}\sigma_e^{-2} & \mathbf{Z}'\mathbf{Z}\sigma_e^{-2} + \mathbf{I}\sigma_s^{-2}
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
\mathbf{X}'\mathbf{y}\sigma_e^{-2} \\
\mathbf{Z}'\mathbf{y}\sigma_e^{-2}
\end{array}
\right]
$$



The system of mixed model equations is often presented as
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{X} & \mathbf{X}'\mathbf{Z} \\
\mathbf{Z}'\mathbf{X} & \mathbf{Z}'\mathbf{Z} + \mathbf{I}\lambda
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
where $\lambda = \sigma_e^2/\sigma_s^2$. This model resembles a simple sire model with unrelated sires in animal breeding.


Note that BLUPF90+ always creates the equations with $\mathbf{R}^{-1}$; $\lambda$ is never calculated. More precisely, $1/\sigma_e^2$ is always calculated and accumulated on the left-hand side (LHS), and $1/\sigma_s^2$ is also added to the LHS.
Basically, the program needs explicit values for the variance components. Here we assume $\sigma_s^2 = 1.0$ and $\sigma_e^2 = 2.0$.

We can use exactly the same data file shown before. The file name is altered just for our convenience.

~~~~~{language=text caption="data2.txt"}
3.0  1  1  1.0
2.0  1  2  1.0
4.0  1  1  2.0
6.0  2  2  2.0
3.0  2  1  1.0
6.0  2  2  2.0
6.0  3  1  2.0
6.0  3  2  1.0
8.0  3  1  1.0
4.0  3  2  2.0
~~~~~

The parameter file is shown below.

~~~~~{language=blupf90 caption="param2.txt"}
DATAFILE
data2.txt
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
1
WEIGHT(S)

EFFECTS:
2 3 cross
3 2 cross
4 1 cov
RANDOM_RESIDUAL VALUES
2.0
RANDOM_GROUP
2
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
1.0
~~~~~

The first 7 keywords are the same as those used in the fixed-effect models. To define the random effect, we use 4 additional keywords^[The common 7 keywords build the design-matrix parts in the mixed model equations, i.e., $\mathbf{X}'\mathbf{X}$, $\mathbf{X}'\mathbf{Z}$, $\mathbf{Z}'\mathbf{X}$, and $\mathbf{Z}'\mathbf{Z}$. The additional 4 keywords provide the covariance structure and $\mathbf{G}^{-1}$] placed immediately after the common keywords.

  1. `RANDOM_GROUP` = shows which effect is random in the `EFFECTS:` block
     - 2 means that the second line (effect 2) in `EFFECTS:` is the random effect
  2. `RANDOM_TYPE` = covariance structure
     - diagonal specifies an identity covariance matrix, i.e., diagonal elements only
  3. `FILE` = additional file for the covariance structure
     - empty for no information needed here
  4. `(CO)VARIANCES` = variance component $\sigma_s^2$
     - 1.0 for the variance

If you have 2 or more random effects (e.g., a permanent environmental or maternal effect), you can add another 4 keywords to the parameter file. We will see such a case later. The solutions will differ substantially from those in the previous section, which assumed that effect 2 was a fixed effect.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1      2.30434783
   1   1         2      4.26086957
   1   1         3      5.28260870
   1   2         1      0.17391304
   1   2         2     -0.17391304
   1   3         1      0.47826087
~~~~~

The solution file contains both BLUEs for the fixed effects and BLUPs for the random effects. The first column is always 1 because this is a single-trait model; the 2nd column shows the effect number from 1 to 3; the 3rd column shows the level in each effect; and the 4th column contains the solutions. The first 3 solutions are for effect 1 ($A_i$), the next 2 solutions are for effect 2 ($S_j$), and the last solution is for the regression ($\beta$). The results are: $\hat{A}_1 = 2.30$, $\hat{A}_2 = 4.26$, $\hat{A}_3 = 5.28$, $\hat{S}_1 = 0.17$, $\hat{S}_2 = -0.17$, and $\hat{\beta} = 0.48$.

The animal model
----------------

The animal model specifically refers to a mixed model with an individual additive genetic effect as a random effect. Genetic covariances between animals are considered using a numerator relationship matrix. You need a pedigree file for this model. In this section, we assume the same model as above with additive genetic effects. An animal has either one observation or no observation (in which case the animal is only in the pedigree file). The mathematical model is as follows:
$$
y_{ijk} = A_i + S_j + \beta x_k + u_k + e_{ijk}
$$
where $u_k$ is the additive genetic effect for animal $k$. Here we treat $S_j$ as a fixed effect. The system of mixed model equations is
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{X}\sigma_e^{-2} & \mathbf{X}'\mathbf{Z}\sigma_e^{-2} \\
\mathbf{Z}'\mathbf{X}\sigma_e^{-2} & \mathbf{Z}'\mathbf{Z}\sigma_e^{-2} + \mathbf{A}^{-1}\sigma_u^{-2}
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
\mathbf{X}'\mathbf{y}\sigma_e^{-2} \\
\mathbf{Z}'\mathbf{y}\sigma_e^{-2}
\end{array}
\right]
$$

where $\mathbf{A}$ is a numerator relationship matrix, $\sigma_u^2$ is the additive genetic variance, and $\sigma_e^2$ is the residual variance.

We further extend the data table to add an animal's ID (code). We show the actual data file to be used in this analysis.

~~~~~{language=text caption="data3.txt"}
  3.0  1  1  1.0   6
  2.0  1  2  1.0   9
  4.0  1  1  2.0  12
  6.0  2  2  2.0   7
  3.0  2  1  1.0  10
  6.0  2  2  2.0  13
  6.0  3  1  2.0   8
  6.0  3  2  1.0  11
  8.0  3  1  1.0  14
  4.0  3  2  2.0  15
~~~~~

In the data file, the animal ID starts at 6; this means that animals 1 to 5 have no observations, so they are not included in the data file. The order of animal IDs is arbitrary, and you do not have to order the animals chronologically.

We show the pedigree file below. Pedigree information consists of 3 columns: animal ID, its sire's ID, and its dam's ID. If an animal's parent is unknown (missing), you should put `0`. The following is the pedigree in this analysis.

~~~~~{language=text caption="pedigree3.txt"}
  1   0   0
  2   0   0
  3   0   0
  4   0   0
  5   0   0
  6   0   0
  7   2   5
  8   1   4
  9   2   3
 10   7   6
 11   7   4
 12  11   8
 13  11  10
 14   9  13
 15  11  10
~~~~~

The individual ID should be an integer value, and it should correspond to the ID in the data file. If your data has non-integer animal IDs, you can use RENUMF90 to replace the characters with integer values in both the pedigree and data files. You do not have to record IDs in chronological order. Also, you do not need to sort the IDs in this file.

The basic format of the parameter file is the same as in the previous example. We assume $\sigma_u^2 = 0.5$ and $\sigma_e^2 = 2.0$.

~~~~~{language=blupf90 caption="param3.txt"}
DATAFILE
data3.txt
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
1
WEIGHT(S)

EFFECTS:
2  3 cross
3  2 cross
4  1 cov
5 15 cross
RANDOM_RESIDUAL VALUES
2.0
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
pedigree3.txt
(CO)VARIANCES
0.5
~~~~~

There are 6 modifications in this file compared with the previous one.

  1. `NUMBER_OF_EFFECTS` = Because the additive genetic effect is added, the number of effects must be increased.
        - The total number of effects is 4.
  2. `EFFECTS:` = The added effect is in the 4th row.
        - `5` for the position of the ID in the data file, `15` for the maximum animal code in the pedigree, and `cross` for the cross-classified effect.
  3. `RANDOM_GROUP` = It represents which effect is random in `EFFECTS:`
        - The 4th effect in the block is a random effect.
  4. `RANDOM_TYPE` = covariance structure
        - `add_animal` for the additive genetic effect
  5. `FILE` = pedigree file
        - `pedigree3.txt`
  6. `(CO)VARIANCES` = the variance component of this random effect
        - $\sigma_{u}^{2} = 0.5$

Now you can run `blupf90+` and obtain the solutions.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1      0.20259645
   1   1         2      2.21996461
   1   1         3      3.16680208
   1   2         1      2.27537292
   1   2         2      1.71160538
   1   3         1      0.52442755
   1   4         1     -0.03487115
   1   4         2      0.08280493
   1   4         3      0.03843921
   1   4         4      0.04492008
   1   4         5      0.04436203
   1   4         6     -0.17565609
   1   4         7      0.10794668
   1   4         8     -0.02984646
   1   4         9      0.09906236
   1   4        10     -0.25282594
   1   4        11      0.15622415
   1   4        12      0.10874296
   1   4        13      0.16426465
   1   4        14      0.34296714
   1   4        15     -0.25707431
~~~~~

The solution file contains both BLUEs for the fixed effects and BLUPs for the random effects. The first column is always 1 because this is a single-trait model; the 2nd column shows the effect number from 1 to 4; the 3rd column shows the level in each effect; and the 4th column contains the solutions. The first 3 solutions are for effect 1 ($A_i$), the next 2 solutions are for effect 2 ($S_j$), the next solution is for the regression ($\beta$), and the remaining 15 solutions are for the additive genetic effect, i.e., estimated breeding values ($u_k$). The results are: $\hat{A}_1 = 0.203$, $\hat{A}_2 = 2.220$, $\hat{A}_3 = 3.167$, $\hat{S}_1 = 2.275$, $\hat{S}_2 = 1.711$, $\hat{\beta} = 0.524$, $\hat{u}_1 = -0.035$, $\hat{u}_2 = 0.083$, $\hat{u}_3 = 0.038$, and so on.


More than 1 random effect
-------------------------

We often use a model with more than 1 random effect, such as a repeatability model (additive genetic + permanent environmental [PE] effect) or a maternal-effect model (direct + maternal + maternal PE effect). The BLUPF90 family supports such complicated models. We assume the same animal model described above, except that $S_j$ is random:
$$
y_{ijk} = A_i + S_j + \beta x_k + u_k + e_{ijk}
$$
where $S_j$ and $u_k$ are random effects. To perform the analysis, we can use the same data and pedigree files. We assume that the two random effects $S_j$ and $u_k$ are not related to each other and that $\sigma_s^2 = 1.0$, $\sigma_u^2 = 0.5$, and $\sigma_e^2 = 2.0$. We need to modify the parameter file to include an additional random effect.

~~~~~{language=blupf90 caption="param3a.txt"}
DATAFILE
data3.txt
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
1
WEIGHT(S)

EFFECTS:
2  3 cross
3  2 cross
4  1 cov
5 15 cross
RANDOM_RESIDUAL VALUES
2.0
RANDOM_GROUP   # first random effect
4
RANDOM_TYPE
add_animal
FILE
pedigree3.txt
(CO)VARIANCES
0.5
RANDOM_GROUP   # second random effect
2
RANDOM_TYPE
diagonal
FILE

(CO)VARIANCES
1.0
~~~~~

We just add a set of 4 keywords (`RANDOM_GROUP`, `RANDOM_TYPE`, `FILE`, and `(CO)VARIANCES`) to the last parameter file for the additional random effect $S_j$. In other words, these 4 keywords are optional; you can add them when you include one more random effect if the effects are unrelated to each other.

If the random effects are correlated with each other, you should use a different approach. This occurs in a maternal-effect model (direct and maternal genetic effects) and a random regression model (random regression coefficients). A multiple-trait model is another case. We do not explain such cases here, but we will consider them in later chapters.

The solutions are shown below.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1      2.27101829
   1   1         2      4.24566929
   1   1         3      5.21524118
   1   2         1      0.18586035
   1   2         2     -0.18586066
   1   3         1      0.49101139
   1   4         1     -0.01963655
   1   4         2      0.06556686
   1   4         3      0.02911205
   1   4         4      0.04793111
   1   4         5      0.03646405
   1   4         6     -0.15943505
   1   4         7      0.08747653
   1   4         8     -0.00548994
   1   4         9      0.07644881
   1   4        10     -0.24829955
   1   4        11      0.13527276
   1   4        12      0.12002543
   1   4        13      0.15461713
   1   4        14      0.33690598
   1   4        15     -0.27372367
~~~~~

The order of the solutions is the same as in the previous animal model, but effect 2 is now a random effect. The order of the solutions is determined by the `EFFECTS:` block in the parameter file. Thus, even if you add more random effects, the order will not change.


Remarks
-------

The BLUPF90 programs build the mixed model equations simply by following the parameter file. Even if your parameter file has no fixed effect (i.e., it has random effects only), the program does not add any fixed effect to the model. Without any fixed effects, the solutions may be meaningless. If you want to fit a random-effects model with a general mean, you should prepare an additional column containing 1 in the data file and describe it as a fixed effect in the parameter file.

In the animal model with `add_animal`, the BLUPF90 program does not account for inbreeding; $\mathbf{A}^{-1}$ is constructed using Henderson's method while ignoring inbreeding. The BLUPF90 program supports $\mathbf{A}^{-1}$ with inbreeding, but it does not calculate the inbreeding coefficients. The user should calculate the inbreeding coefficients and put them into the pedigree file in a special format. Using this special pedigree file and `add_an_upginb` (instead of `add_animal`), inbreeding can be incorporated. The `renumf90` program can calculate the inbreeding coefficients and prepare such a pedigree file.

Note that the number of levels for each effect in the parameter file is the maximum level or animal code found in the data or pedigree files. It is not the actual number of levels present in the files. BLUPF90+ runs correctly as long as this number is equal to or greater than the maximum level or animal code. For example, if the maximum level code is 10 but level 1 is not present in the data, you should enter `10`, not `9`, in the parameter file. In this case, BLUPF90+ will provide valid solutions, with zeros for the missing levels. Similarly, even though the maximum level code is 10, you can enter a larger number (such as `11` or even `100`) in the parameter file. This makes the computation less efficient, but BLUPF90+ will still run correctly.

Summary
-------

 - BLUPF90 supports linear mixed models with one or more random effects.
 - For analyses using pedigree information, BLUPF90 needs 3 text files: data, pedigree, and parameter files.
 - A pedigree file is needed if you perform genetic analyses.
 - The pedigree file should contain 3 columns: animal ID, sire ID, and dam ID. The ID must be an integer value.
 - To define a random effect, add 4 keywords (`RANDOM_GROUP`, `RANDOM_TYPE`, `FILE` and `(CO)VARIANCES`) and the corresponding values to the parameter file.
 - The order of the solutions depends on the `EFFECTS:` block in the parameter file.
 - You should manually add the general mean if you use a random effect model.
 - With `add_animal`, inbreeding is ignored in the calculation of $\mathbf{A}^{-1}$.
 - The number of levels in the parameter file actually means the maximum level or animal code found in the data and pedigree files.
