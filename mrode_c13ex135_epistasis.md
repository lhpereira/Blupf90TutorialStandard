---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Epistatic genomic models
========================

Model
-----

Epistasis consists of higher-level interactions among genetic effects across loci. An epistatic effect can be defined as a random effect in a linear mixed model, and it is associated with a relationship matrix describing the covariance structure among individuals.

Typical epistatic effects include additive-additive, additive-dominance, and additive-additive-dominance effects. An epistatic relationship matrix can be formed as the Hadamard product of two relationship matrices associated with the epistatic effect. For example, the additive-additive relationship matrix is defined as $\mathbf{G}_{AA}=\mathbf{G}\odot\mathbf{G}/k_{AA}$, where $\odot$ is the Hadamard product operator and $k_{AA}$ is a standardization constant. Similarly, the additive-dominance relationship matrix is $\mathbf{G}_{AD}=\mathbf{G}\odot\mathbf{D}/k_{AD}$, where $k_{AD}$ is a constant.

In this section, the author applies a genomic model with additive genetic, dominance genetic, and additive-additive effects.

Files
-----

The data set is essentially the same as the one used in the GBLUP dominance model in Example 13.3. The inverse of the additive-additive relationship matrix is prepared as a file (`mr135.Gaai`). The parameter file is extended to include the additive-additive effect (`mr135.par`).

Solutions
---------

The solutions are identical to those shown in the textbook (p. 238).
