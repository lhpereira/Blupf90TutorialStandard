---
title: Introduction to BLUPF90 suite programs
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

## Revision History

- [Revision History](./history.html)

## Acknowledgment

- [Acknowledgment](./acknowledgment.html)

## License

- [License](./license.html)

## Introduction

- [Short introduction to BLUPF90 programs](./introduction_short.html)
- [Differences with other software](./introduction_difference.html)
- [About the software](./introduction_condition.html)
- [About this tutorial](./introduction_about.html)
- [New programs](./introduction_plus.html)

## Download and Installation

- [Introduction](./installation_start.html)
- [Availability](./installation_availability.html)
- [Running a program in Windows](./installation_windows.html)
- [Running a program in Linux and MacOS X](./installation_linux.html)
- [Additional settings](./installation_env.html)
- [Setup a text editor](./installation_editor.html)

## Quick tour of BLUPF90

- [Introduction](./quicktour_start.html)
- [Trivial analyses for fixed models](./quicktour_fixed.html)
- [Trivial analyses for mixed models](./quicktour_mixed.html)
- [Trivial analyses for single-step GBLUP](./quicktour_ssgblup.html)
- [Trivial analyses for multiple-trait models](./quicktour_mt.html)

## Data preparation with RENUMF90

- [Introduction](./renum_start.html)
- [Basic data preparation](./renum_basic.html)
- [Animal model with pedigree file](./renum_pedigree.html)
- [Genomic model with SNP-marker file](./renum_genomic.html)
- [Multiple-trait models](./renum_mt.html)
- [Advanced usage of RENUMF90](./renum_advanced.html)
- [What if I don't want to use RENUMF90?](./renum_norenum.html)

## Variance component estimation

- [Restricted (residual) maximum likelihood with AIREMLF90 (BLUPF90+)](./vc_aireml.html)
- [Gibbs sampling and post-Gibbs analysis](./vc_gs.html)
- [Advanced usage of AIREMLF90 (BLUPF90+)](./vc_advanced_aireml.html)
- [Advanced features for Gibbs sampling programs](./vc_advanced_gs.html)

## Practical genomic analysis

- [Introduction](./genomic_start.html)
- [Overview of genomic features](./genomic_overview.html)
- [Files used in genomic analysis](./genomic_files.html)
- [Quality control of SNP markers](./genomic_qc.html)
- [Tuning and input/output of relationship matrices](./genomic_tuning.html)
- [Fine control of single-step GBLUP](./genomic_ssgblup.html)
- [Performing GBLUP](./genomic_gblup.html)
- [GWAS using the ssGBLUP framework](./genomic_gwas.html)

## Numerical examples from Mrode and Pocrnic (2023)

- [Introduction](./mrode_start.html)
- [Chapter 4. Example 4.1. Animal model](./mrode_c04ex041_animal_model.html)
- [Chapter 4. Example 4.2. Sire model](./mrode_c04ex042_sire_model.html)
- [Chapter 4. Example 4.3. Reduced animal model](./mrode_c04ex043_reduced_animal_model.html)
- [Chapter 4. Example 4.4. Animal model with groups](./mrode_c04ex044_animal_model_with_groups.html)
- [Chapter 5. Example 5.1. Repeatability model](./mrode_c05ex051_repeatability_model.html)
- [Chapter 5. Example 5.2. Model with common environmental effects](./mrode_c05ex052_common_environment.html)
- [Chapter 6. Example 6.1. Multiple-trait model, Equal design matrices and no missing records](./mrode_c06ex061_mt_equal_design.html)
- [Chapter 6. Example 6.2. Multiple-trait model, Equal design matrices with missing records](./mrode_c06ex062_mt_missing.html)
- [Chapter 6. Example 6.3. Multiple-trait model, Unequal design matrices](./mrode_c06ex063_mt_unequal_design.html)
- [Chapter 6. Example 6.4. Multivariate models with no environmental covariance](./mrode_c06ex064_mt_no_covariance.html)
- [Chapter 7. Example 7.1. Canonical transformation](./mrode_c07ex071_canonical_transformation.html)
- [Chapter 7. Example 7.2. Cholesky transformation](./mrode_c07ex072_cholesky_transformation.html)
- [Chapter 8. Example 8.1. Animal model for maternal traits](./mrode_c08ex081_maternal.html)
- [Chapter 8. Example 8.2. Reduced animal model with maternal effects](./mrode_c08ex082_reduced_maternal.html)
- [Chapter 9. Example 9.1. Animal model with social interaction effects](./mrode_c09ex091_social_interaction.html)
- [Chapter 10. Example 10.1. Fixed regression model](./mrode_c10ex101_fixed_regression.html)
- [Chapter 10. Example 10.2. Random regression model](./mrode_c10ex102_random_regression.html)
- [Chapter 11. Example 11.1. Fixed effect model for SNP effects](./mrode_c11ex111_fixed_snp.html)
- [Chapter 11. Example 11.2. Mixed linear model for computing SNP effects](./mrode_c11ex112_mixed_snp.html)
- [Chapter 11. Example 11.3. Equivalent models: GBLUP](./mrode_c11ex113_gblup.html)
- [Chapter 11. Example 11.4. SNP solutions from GBLUP](./mrode_c11ex114_snp_solutions.html)
- [Chapter 11. Example 11.6. Base-population allele frequency](./mrode_c11ex116_allele_frequency.html)
- [Chapter 11. Example 11.7. Mixed linear models with polygenic effects](./mrode_c11ex117_polygenic.html)
- [Chapter 11. Example 11.8. Haplotype models](./mrode_c11ex118_haplotype.html)
- [Chapter 11. Example 11.13. Multivariate genomic models](./mrode_c11ex1113_mt.html)
- [Chapter 12. Example 12.1. Single-step approaches: basic principle](./mrode_c12ex121_ssgblup.html)
- [Chapter 12. Example 12.2. APY approach](./mrode_c12ex122_apy.html)
- [Chapter 13. Example 13.1. Animal model with dominance effect](./mrode_c13ex131_dominance.html)
- [Chapter 13. Example 13.2. Total genetic merit](./mrode_c13ex132_additive_and_dominance.html)
- [Chapter 13. Example 13.3. Genomic models for dominance effects](./mrode_c13ex133_genomic_dominance.html)
- [Chapter 13. Example 13.4. Inclusion of genomic inbreeding](./mrode_c13ex134_genomic_inbreeding.html)
- [Chapter 13. Example 13.5. Epistatis](./mrode_c13ex135_epistasis.html)
- [Chapter 15. Example 15.1. The threshold model](./mrode_c15ex151_threshold.html)
- [Chapter 15. Example 15.2. Joint analysis of quantitative and binary traits](./mrode_c15ex152_threshold_linear.html)
<!--
- [Chapter 10. Example 10.2. Prediction of breeding values with marker information](./mrode_c10ex102_marker_information.html)
- [Chapter 10. Example 10.3. Directly predicting the additive genetic merit at the QTL](./mrode_c10ex103_qtl.html)
- [Chapter 12. Example 12.3. Method for rapid inversion of the dominance matrix](./mrode_c12ex123_dominance_inverse.html)
-->

## Large-scale genetic evaluation
- [Introduction](./largescale_start.html)
- [Issues in a large scale analysis](./largescale_issues.html)
- [Iteration on data with preconditioned conjugate gradient (PCG)](./largescale_pcg.html)
- [Approximation of accuracy and reliability](./largescale_reliability.html)
- [REML estimation with large data](./largescale_reml.html)

## References

- [References](./references.html)
