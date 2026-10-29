---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Base-population allele frequency
================================

Model
-----

This section deals with a slightly different topic in genomic prediction: estimating allele frequencies in the base population. The characteristics of the genomic relationship matrix $\mathbf{G}$ are largely determined by the allele frequencies used in its calculation (see VanRaden 2008). In practice, the allele frequencies are generally unknown. Genotyped animals are also usually from recent generations, whereas the base population is defined many generations in the past.

Gengler et al. (2007) suggested a method for estimating the genotype (also known as "gene content" or "gene dosage") at a locus for an ancestral animal, given the genotypes of descendant animals and complete pedigree relationships among the descendants and target ancestors. Once we estimate the genotypes of nongenotyped ancestors from older generations, which are assumed to be part of the base population, we can also estimate the marker allele frequency in the base population.

In this method, the estimation follows the mixed-model framework. We treat the descendant genotypes as "phenotypes" and the genotype to be predicted as an "additive genetic effect." Then, we can apply the following simple animal model:
$$
y_{i} = \mu + g_{i} + e_{i}
$$
where $y_{i}$ is the observed genotype for animal $i$, $\mu$ is the general mean, $g_{i}$ is the predicted genotype, and $e_{i}$ is the error term. The heritability is assumed to be close to 1, say, 0.99. This model results in the usual mixed-model equations for an animal model.

Files
-----

The GitHub repository provides the data and pedigree files for this numerical example. The data file `mr116.dat` contains three columns, as shown on p. 197.

~~~~~{language=text caption="mr116.dat"}
13 1 2
14 1 1
15 1 1
16 1 0
17 1 0
18 1 1
19 1 0
20 1 0
21 1 2
22 1 0
23 1 0
24 1 1
25 1 0
26 1 1
~~~~~

1. The original animal ID
2. General mean (`1`)
3. Marker genotype (0, 1, or 2)

The pedigree file `mr116.ped` is the same as the previous numerical example.

The parameter file (`mr116.par`) is as follows.

~~~~~{language=blupf90 caption="mr116.par"}
DATAFILE
mr116.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
3
WEIGHT(S)

EFFECTS:
2 1 cross
1 26 cross
RANDOM_RESIDUAL VALUES
1.01
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr116.ped
(CO)VARIANCES
101
OPTION solv_method FSPAK
~~~~~

The author chose arbitrary variance components so that the heritability is 0.99, which corresponds to $\sigma_{e}^{2}/\sigma_{u}^{2}=0.01$.

Solutions
---------

We will obtain the same solutions as those in the textbook (p. 197). The predicted genotype is $\hat{\mu} + \hat{u}_i$, and the prediction will generally be a noninteger value. In genomic prediction, noninteger genotypes may be used. The allele frequency can also be estimated from these noninteger genotypes.
