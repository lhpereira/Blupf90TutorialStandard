---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Genomic models for dominance effects
====================================

Model
-----

Given SNP marker data, nonadditive genetic effects can be directly integrated into a mixed-model framework. There are several ways to model nonadditive effects. The textbook deals with a simple model: the GBLUP analogue. We can construct a separate "relationship matrix" for each nonadditive effect and use these matrices in the mixed model.

The dominance effect is the most important nonadditive effect in quantitative genetic studies. A typical mixed model for the dominance effect is as follows (Equation 13.5).
$$
\mathbf{y} = \mathbf{Xb} + \mathbf{T}\mathbf{u}_a + \mathbf{V}\mathbf{u}_d + \mathbf{e} 
$$
This is an extension of the additive model (breeding-value model): $\mathbf{T}$ (often written as $\mathbf{Z}$) is a centered marker matrix associated with the additive genetic effect $\mathbf{u}_a$, and $\mathbf{V}$ is another centered marker matrix associated with the dominance effect $\mathbf{u}_d$. The additive and dominance relationship matrices are defined as follows:
$$
\mathrm{var}(\mathbf{u}_a) = \mathbf{T}\mathbf{T}'/k_a = \mathbf{G}\sigma_a^2 \quad \text{and} \quad \mathrm{var}(\mathbf{u}_d) = \mathbf{V}\mathbf{V}'/k_d = \mathbf{D}\sigma_d^2
$$
where $k_a$ and $k_d$ are functions of the allele frequencies across all marker loci for each effect, $\mathbf{G}$ ($\mathbf{D}$) is the additive (dominance) genomic relationship matrix, and $\sigma_a^2$ and $\sigma_d^2$ are the additive genetic and dominance variances, respectively. Given the variance-covariance structure, we can construct a system of mixed-model equations and predict the effects as solutions of those equations. This section uses the widely accepted GBLUP dominance model.

BLUPF90+ does not have a dedicated option for creating nonadditive relationship matrices, unlike $\mathbf{G}$ for additive effects. The user should prepare the relationship matrix and plug it into the equations, as in the previous sections.

Files
-----

The data file (`mr133.dat`) contains five columns.

1. Animal
2. Sire ID
3. Dam ID
4. Pen
5. Phenotype (WWT)

The pedigree file (`mr133.ped`) has 15 animals.

BLUPF90+ creates the genomic relationship matrix from the marker (`mr133.gen`) and cross-reference (`mr133.xrf`) files, but the inverse of the dominance relationship matrix is read from the file (`mr133.Di`). The parameter file (`mr133.par`) is as follows.

~~~~~{language=blupf90 caption="mr133.par"}
DATAFILE
mr133.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
3
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
4  2 cross
1 15 cross
1 15 cross
RANDOM_RESIDUAL VALUES
120.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr133.ped
(CO)VARIANCES
90.0
RANDOM_GROUP
3
RANDOM_TYPE
user_file
FILE
mr133.Di
(CO)VARIANCES
80.0
OPTION solv_method FSPAK
OPTION SNP_file mr133.gen mr133.xrf
OPTION no_quality_control
OPTION AlphaBeta 1.00 0.00
OPTION GammaDelta 0.01 0.00
OPTION tunedG 0
~~~~~

The setup of the additive genomic relationship matrix is the same as that shown in the GBLUP section.

Solutions
---------

The solutions are identical to those shown in the textbook (pp. 233--234).
