---
title: Various models presented in Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

In this chapter, we will see how BLUPF90 can be applied to a variety of models. We will examine the models presented in the standard textbook *Linear Models for the Prediction of the Genetic Merit of Animals*, 4th ed., by Mrode and Pocrnic (2023). We explain how to write a parameter file for each model in their book. The phenotypes, pedigrees, genotypes, and parameter files for the BLUPF90 family are available on the following GitHub page.

- [MrodesBook](https://github.com/MrodesBook)

In this section, we will use subsets of the phenotype, pedigree, and genotype data. We will not only solve the equations but also introduce new options and techniques useful for actual data analyses.

Note that, in most of the numerical examples, we use `OPTION solv_method FSPAK` and occasionally `OPTION sol se` or `OPTION store_accuracy`. These options are useful for calculating more precise (exact) solutions to the mixed model equations and the reliability (or accuracy) of an estimated breeding value. However, these options are computationally demanding in terms of both computation time and memory requirements. They should not cause problems with small- to medium-sized data sets, but computations may be slow or the program may even crash if the data set is too large. Please be aware that the purpose of this chapter is to demonstrate how BLUPF90+ and related tools can handle a wide variety of models. For large-scale data sets, please read the subsequent chapter.
