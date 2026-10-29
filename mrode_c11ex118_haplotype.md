---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Haplotype models
================

Model
-----

A haplotype model uses haplotypes (SNP blocks) as covariates. This model may capture more signal from quantitative trait loci (QTL) for some traits if the phenotype is expected to have significant QTL effects.

Depending on the method, the user may need to estimate haplotype blocks from the genotypes. Once the haplotype blocks have been prepared, they are treated in the same way as marker genotypes in the model. The prediction unit is a haplotype block rather than a marker genotype, and the prediction model is essentially the same as the SNP-BLUP model. The total breeding value is the sum of the haplotype effects, as in the SNP-BLUP model. Alternatively, the user can construct a haplotype-based genomic relationship matrix, $\mathbf{G}_{h}$, and use it in the GBLUP model to predict animals' breeding values directly.

Please see the textbook for details.

Files
-----

The data set (`mr118.dat`) is from Table 4.1 in Chapter 4 of the textbook, where it is used for the animal model.

The haplotype model here is based on SNP-BLUP or GBLUP, so we do not use pedigree information. However, we still need a dummy pedigree file to perform the analysis with BLUPF90+. The dummy pedigree contains all 8 animals, with all parents unknown and coded as `0`.

~~~~~{language=text caption="mr118.ped"}
1 0 0
2 0 0
...
8 0 0
~~~~~

For comparison, the regular SNP marker file (`mr118.gen`) is also provided. The data are the same as those in the textbook (p. 201). The cross-reference file (`mr118.xrf`) should also be prepared.

~~~~~{language=text caption="mr118.gen"}
1 201010220
2 021111021
3 211012001
4 111111120
5 100001012
~~~~~

The haplotype file contains the "haplotype alleles" for each animal. The user should prepare the haplotype file in the same format as the SNP marker file, with the animal ID followed by a sequence of "haplotype alleles" coded as `0`, `1`, and `2`. Thus, the haplotype data look like pseudo-SNP marker data.

The textbook describes how to define haplotype alleles. The haplotype alleles can be defined arbitrarily by the user. For example, in the textbook, the 9 SNP loci are divided into 3 haplotype blocks, and the first haplotype block (Hap 1) has 6 possible "alleles" (`101`, `100`, `011`, `010`, `111`, `000`). In the haplotype data file, Hap 1 is assigned 6 entries, each corresponding to one of these "alleles," and each entry is the count of that haplotype carried by the animal. The file contains the sequence of entries across all haplotype blocks.

The prepared haplotype data file (`mr118.hap`) has the same structure as the "Pseudo-SNPs" example, with 16 entries (6 for Hap 1, 5 for Hap 2, and 5 for Hap 3) in the textbook (p. 201). We can use the same cross-reference file as for the SNP-BLUP/GBLUP model.

~~~~~{language=text caption="mr118.hap"}
1 1100001100020000
2 0011000110001100
3 0100100001100011
4 0110000110011000
5 0100010100000110
~~~~~

The textbook demonstrates two models, and both are based on GBLUP. The first model is a regular GBLUP model with marker genotypes (not haplotypes) for comparison.

~~~~~{language=blupf90 caption="mr118a.par"}
# Normal GBLUP model
DATAFILE
mr118.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
6 1 cross
1 8 cross
RANDOM_RESIDUAL VALUES
40
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr118.ped
(CO)VARIANCES
20
OPTION SNP_file mr118.gen mr118.xrf
OPTION no_quality_control
OPTION AlphaBeta 1.00 0.00
OPTION GammaDelta 0.01 0.00
OPTION tunedG 0
OPTION solv_method FSPAK
~~~~~

The other model uses haplotypes to create a haplotype-based version of the genomic relationship matrix.

~~~~~{language=blupf90 caption="mr118b.par"}
# haplotype-based GBLUP model
DATAFILE
mr118.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
5
WEIGHT(S)

EFFECTS:
6 1 cross
1 8 cross
RANDOM_RESIDUAL VALUES
40
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr118.ped
(CO)VARIANCES
20
OPTION SNP_file mr118.hap mr118.xrf
OPTION no_quality_control
OPTION AlphaBeta 1.00 0.00
OPTION GammaDelta 0.01 0.00
OPTION tunedG 0
OPTION solv_method FSPAK
~~~~~


Solutions
---------

The solutions are the same as those described in the textbook (p. 202). The solutions from the two models are similar but not identical, although the ranking of predictions is the same for both models.
