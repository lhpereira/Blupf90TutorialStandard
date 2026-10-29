---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Reduced animal model
====================

Model
-----

The reduced animal model is an animal model that includes only parents (that is, animals with progeny). Breeding values for animals without progeny are calculated indirectly using their parents' breeding values. This model is simpler and saves computational resources, particularly when phenotypes are available only for terminal individuals (such as beef steers) that will not have progeny; therefore, their breeding values are not used in selection decisions. We omit the description of the model and its theory here. See the textbook or Quaas and Pollak (1980) for details.

BLUPF90+ does not directly support the reduced animal model, but it can still be implemented in BLUPF90+ using (very) tricky methods. Some techniques used here will also be used in random-regression and social-interaction models. Although there are several ways to implement a reduced animal model in BLUPF90+, we introduce here an educational example that is redundant but easier to understand.

Files
-----

We can use the same pedigree file (`mr43a.ped`) as in the previous animal model after removing animals without progeny (animals 7 and 8 in this case).

~~~~~{language=text caption="mr43a.ped"}
1 0 0
2 0 0
3 0 0
4 1 0
5 3 2
6 1 2
~~~~~

The data file is extended with 5 additional columns. Here, we show a few lines of the data file.

~~~~~{language=text caption="mr43a.dat"}
4 1 1 0 4.5  4 0.0 0.0 1.0  1.0
5 2 3 2 2.9  5 0.0 0.0 1.0  1.0
6 2 1 2 3.9  6 0.0 0.0 1.0  1.0
7 1 4 5 3.5  0 0.5 0.5 0.0  0.8
8 1 3 6 5.0  0 0.5 0.5 0.0  0.8
~~~~~

The 6th column (the first additional column) contains the animal ID, but is 0 if the animal has no progeny. Columns 7, 8, and 9 contain values to be added to the incidence matrix $\mathbf{W}$ (see the textbook). This idea is similar to the method used to support random regressions. Columns 7 and 8 are 0.0 if the animal is a parent and 0.5 otherwise. Column 9 is 1.0 if the animal is a parent and 0 otherwise.

The 10th column (the last column) contains a weight used to adjust the residual variance. For parent animals, the residual variance is $\sigma_e^2 = 40$, and its inverse is $1/\sigma_e^2 = 0.025$. For non-parent animals in this case, the residual variance should be $\sigma_e^{2*}=\sigma_e^2+0.5\sigma_a^2= 40 + 0.5 \times 20 = 50$, and its inverse is $1/\sigma_e^{2*} = 0.020$. If a non-parent animal has a missing parent, you should use a different weight for $\sigma_a^2$ (see the textbook). We use `WEIGHT(S)` to alter the residual variances. The program reads the weight from the data file and multiplies it by the inverse of the residual variance. In our example, the weight is 1.0 for parent animals and 0.8 for non-parent animals because $0.8 \times (1/\sigma_e^2) = 0.8 \times 0.025 = 0.020$, which is the correct value for non-parent animals.

One issue is how to add these constants to $\mathbf{W}$. For a cross-classified effect, BLUPF90 can usually add only values equal to 1 to the incidence matrix for each record. To overcome this limitation, we use a special specification in the `EFFECTS:` block of the parameter file.

~~~~~{language=blupf90 caption="mr43a.par"}
DATAFILE
mr43a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
5
WEIGHT(S)
10
EFFECTS:
2 2 cross
7 0 cov 3
8 0 cov 4
9 6 cov 6
RANDOM_RESIDUAL VALUES
40.0
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
mr43a.ped
(CO)VARIANCES
20.0
OPTION solv_method FSPAK
~~~~~

In the `EFFECTS:` block, the last three statements pass values from the data file through the incidence matrix. These three effects are in one block, and only the last statement, `9 6 cov 6`, specifies the number of levels (that is, the number of animals). For each statement, the program reads the value from the data file (the position is the first number on that line), recognizes it as a covariate because of the `cov` keyword, and places it in the location in the design matrix corresponding to an animal ID (the location is defined by the number after `cov`).

Let's see what happens when the program processes these three statements for each record. We consider the first line of the data file.

~~~~~{language=text}
4 1 1 0 4.5       4 0.0 0.0 1.0        1.0
~~~~~

- Processing statement 2 (`7 0 cov 3`): the program reads the 7th column from the data, recognizes it as a real value (0.0), and adds it to $\mathbf{W}$ at the location defined by the 3rd column in the data file (the 1st column in $\mathbf{W}$).
- Processing statement 3 (`8 0 cov 4`): the program reads the 8th column from the data, recognizes it as a real value (0.0), and adds it to $\mathbf{W}$ at the location defined by the 4th column in the data file (location 0, which is ignored).
- Processing statement 4 (`9 6 cov 6`): the program reads the 9th column from the data, recognizes it as a real value (1.0), and adds it to $\mathbf{W}$ at the location defined by the 6th column in the data file (the 4th column in $\mathbf{W}$).

After this processing, the first row in $\mathbf{W}$ should be as follows.
$$
\left[
\begin{array}{cccccc}
   0.0&0.0&0.0&1.0&0.0&0.0
\end{array}
\right]
$$

Then we see the 4th line of the data file.

~~~~~{language=text}
7 1 4 5 3.5       0 0.5 0.5 0.0        0.8
~~~~~

- Processing statement 2 (`7 0 cov 3`): the program reads the 7th column from the data, recognizes it as a real value (0.5), and adds it to $\mathbf{W}$ at the location defined by the 3rd column in the data file (the 4th column in $\mathbf{W}$).
- Processing statement 3 (`8 0 cov 4`): the program reads the 8th column from the data, recognizes it as a real value (0.5), and adds it to $\mathbf{W}$ at the location defined by the 4th column in the data file (the 5th column in $\mathbf{W}$).
- Processing statement 4 (`9 6 cov 6`): the program reads the 9th column from the data, recognizes it as a real value (0.0), and adds it to $\mathbf{W}$ at the location defined by the 6th column in the data file (location 0, which is ignored).

After this processing, the 4th row in $\mathbf{W}$ should be as follows.
$$
\left[
\begin{array}{cccccc}
0.0& 0.0& 0.0& 0.5& 0.5& 0.0
\end{array}
\right]
$$

A similar technique will be used in a competitive model (social-interaction model), which needs an incidence matrix with two or more nonzero elements in one row.


Solutions
---------

The solutions are identical to those in the textbook (p. 81).

You can find other ways to implement the reduced animal model using the same technique described above. BLUPF90+ supports heterogeneous residual variances, which may simplify the implementation of the reduced animal model. Curious readers can try this method.
