---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Inclusion of genomic inbreeding
===============================

Model
-----

This section considers a GBLUP dominance model with a fixed regression coefficient for genomic inbreeding values. The regression coefficient can account for inbreeding depression. In theory, this model is expected to provide a more precise estimate of the dominance variance.

Files
-----

The files are identical to those in the previous Example 13.4, except that the inbreeding coefficients are included in the model. The inbreeding coefficient is in the sixth column of the data file (`mr134.dat`). The model has an additional regression coefficient for the inbreeding coefficient as a covariate, defined in the parameter file (`6 1 cov` in `mr134.par`).

Solutions
---------

The inbreeding depression (the regression coefficient) is $-3.77$. The other solutions are similar to those calculated in Example 13.3.
