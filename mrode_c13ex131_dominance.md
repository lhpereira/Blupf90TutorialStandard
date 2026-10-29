---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Animal model with dominance effect
==================================

Model
-----

Linear mixed models can handle a nonadditive genetic model with dominance or epistatic relationship matrices. In a dominance model, we can consider additive and dominance genetic effects simultaneously. In this example, the author assumes an animal model with one fixed effect, an additive genetic effect, a dominance effect, and a residual effect. The additive variance is $\sigma_a^2 = 90$, the dominance variance is $\sigma_d^2 = 80$, and the residual variance is $\sigma_e^2 = 120$. BLUPF90 does not have a function for calculating $\mathbf{D}^{-1}$, so we need to supply its elements to the program in a file.

Files
-----

A data file (`mr131.dat`) should be prepared. The pedigree file (`mr131.ped`) is also required.

The inverse of the dominance relationship matrix is supplied as a text file (`mr131.dom`). See the textbook for the actual numerical values (p. 227). The prepared file contains more precise values.

~~~~~{language=text caption="mr131.dom"}
 1  1  1.00000000
 2  2  1.00000000
 ...
11 12 -0.24096386
12 12  1.09236948
~~~~~

The parameter file (`mr131.par`) defines two random effects: additive and dominance genetic effects. The inverse of the dominance relationship matrix is inserted directly into the equations.

~~~~~{language=blupf90 caption="mr131.par"}
DATAFILE
mr131.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
4  2 cross
1 12 cross
1 12 cross
RANDOM_RESIDUAL VALUES
120.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr131.ped
(CO)VARIANCES
90.0
RANDOM_GROUP
3
RANDOM_TYPE
user_file
FILE
mr131.dom
(CO)VARIANCES
80.0
OPTION solv_method FSPAK
~~~~~

Solutions and remarks
---------------------

The solutions are identical to the reference values in the textbook (p. 228).

Note that there is a rapid algorithm for calculating the inverse of the dominance relationship matrix while ignoring inbreeding, using a specialized "pedigree" file. The algorithm was suggested by Hoeschele and VanRaden (1991). They focused on the parental subclass effects $\mathbf{f}$, associated with the dominance effect, and their covariance matrix $\mathbf{F}$. They developed the algorithm to calculate $\mathbf{F}^{-1}$, while ignoring inbreeding, using the dominance pedigree file. The inverse $\mathbf{D}^{-1}$ can be derived from $\mathbf{F}^{-1}$.

The BLUPF90 family can calculate only $\mathbf{F}^{-1}$. This matrix itself cannot be used to estimate individual dominance effects, although $\mathbf{F}^{-1}$ is still useful for estimating variance components. Another drawback of this algorithm is that the dominance pedigree file has a somewhat complicated structure. This "parental dominance" method was explained in the previous edition of the textbook but has been removed from the current edition.
