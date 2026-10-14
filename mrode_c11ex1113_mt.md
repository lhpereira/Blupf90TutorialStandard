---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Multivariate genomic models
===========================

Model
-----

The genomic models can be extended to a multivariate setup. The textbook focuses on multivariate GBLUP and gives a numerical example for this model. Users can apply the same techniques and tricks discussed in the previous sections to multivariate genomic models.

Please see the textbook for details.

Files
-----

In the textbook, the author compares breeding-value solutions between the pedigree-based and genomic (GBLUP) models. Both models use the same data set (`mr1113.dat`), which is also used in Table 6.1 (Chapter 6) of the textbook (p. 210).

For the pedigree BLUP model, the pedigree file `mr1113a.ped` is used. For GBLUP, the pedigree file (`mr1113b.ped`) is a dummy file. The genotype file (`mr1113.gen`) and the cross-reference file (`mr1113.xrf`) are used exclusively for GBLUP.

Each model needs a separate parameter file. In the GitHub repository, the user can find `mr1113a.par` for the pedigree model and `mr1113b.par` for GBLUP.

Solutions
---------

The calculated solutions are identical to those in the textbook (p. 211).

Note that, as stated in the textbook, the comparison between the models (pedigree BLUP versus GBLUP) is not valid because the data contain only 10 SNP markers. Too few markers tend to result in an inaccurate $\mathbf{G}$, which may lead to unstable genomic predictions. Therefore, this numerical example should be viewed as a demonstration of multivariate analysis with GBLUP.
