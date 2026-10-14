---
title: Introduction
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Differences from other software
===============================

It is helpful for new users to understand how BLUPF90 programs differ from other software they may be familiar with. We briefly introduce these differences to help you understand them.

General differences
-------------------

### R and SAS ###

* BLUPF90 can handle very large datasets. It is likely to be faster than R and SAS.
* BLUPF90 accepts linear mixed models and some nonlinear models, such as threshold models. R and SAS can handle more types of models, including models other than linear mixed models.
* BLUPF90 is not interactive software. Users must prepare a *parameter file* that describes the details of the analysis (file names, model, genetic parameters, and options). The program simply reads the file and performs the defined task.
* BLUPF90 is not a scripting language. The programs do not provide any features for data manipulation. All data editing must be done before running the program. The data file must include all information required for the analysis.
* BLUPF90 can only read text files. Headers (or comments) are not allowed in the data or pedigree files. Items in a text file should be separated by one or more spaces.
* BLUPF90 shows the results on screen and writes them to text files.
* BLUPF90 accepts only integer values as effect levels in the data and pedigree files, and real values as covariates and trait values. Alphabetic characters or symbols must be replaced with integer codes before analysis. RENUMF90 can assist with this process.
* BLUPF90 has no graphical interface. It is similar to `Rscript`, a command-line utility. The program runs in Command Prompt (also called the 'DOS' window) in Windows, Terminal in macOS, and various shells in Linux.
* BLUPF90 does not provide functions for hypothesis testing. Some programs show $-2\log L$ or similar statistics, which users can use to perform likelihood ratio tests manually.

### ASREML ###

* BLUPF90 is free for research and academic purposes, but it offers limited support from the development team.
* BLUPF90 supports Gibbs sampling.
* BLUPF90 supports linear mixed models and threshold models. It does not support generalized linear models.
* BLUPF90 uses simple text file formats for data and pedigree files. Headers, comments, alphabetic characters, or symbols are not allowed in these files. The user must prepare and edit the data before analysis.
* BLUPF90 does not offer comprehensive directives for analysis. The parameter file contains minimal information, such as file names, the model, variance components, and options. BLUPF90 does not use labels to refer to effects in the model description.
* BLUPF90 produces minimal output.

<!--
Note: WOMBAT is no longer available. So, I would delete this.

### WOMBAT ###

* In BLUPF90 data files, observations recorded for an animal at the same time should be listed on the same line. Multiple observations must be saved on one line. Storing observations from the same animal on multiple lines is allowed only for repeated records or special multi-trait models (e.g., G-by-E analysis). In WOMBAT, each trait is recorded on a separate line.
* BLUPF90 does not compute inbreeding coefficients by default. The inverse of the numerator relationship matrix ($\mathbf{A}^{-1}$) is created using a non-inbred approach (i.e., Henderson's method, assuming no inbreeding). To consider inbreeding in $\mathbf{A}^{-1}$, users must supply inbreeding coefficients in a special format in the pedigree file.
* BLUPF90 does not use labels to specify the model. Instead, it refers directly to the column positions of effect identifiers.
* BLUPF90 does not create interaction effects. It does not support `*` notation in parameter files. Interactions must be prepared as cross-classified effects before analysis.
* BLUPF90 accepts both positive and negative real numbers as covariates, whereas WOMBAT accepts only integers.
* WOMBAT has more options for maximizing the likelihood (e.g., Derivative-Free, PX, reduced-rank methods), while BLUPF90 supports Bayesian estimation via Gibbs sampling.
* BLUPF90 supports the threshold model. WOMBAT does not.
-->

<!--
Note: VCE is no longer available. So, I would delete this.

### VCE ###

* BLUPF90 data and pedigree files must not contain headers or comments. The first row must be a data line.
* BLUPF90 uses a very simple parameter file format, which is not as human-readable as that used in VCE. It does not use labels for effects. The parameter order in the file is fixed.
* The pedigree file must contain animal ID, sire ID, and dam ID as the first three columns for the animal model. A fourth column can be added as a special feature.
* BLUPF90 does not compute inbreeding coefficients by default. The inverse of the numerator relationship matrix ($\mathbf{A}^{-1}$) is generated using a non-inbred approach (i.e., Henderson's method). Users who wish to consider inbreeding must provide inbreeding coefficients in a special format in the pedigree file.
-->

<!--
Note: DMU is no longer available. So, I would delete this.

### DMU ###

* BLUPF90 directly reads data and pedigree files; users do not need to run a DMU1-type program. Data preparation programs can optionally be used for user convenience.
* The parameter file (similar to a driver file) contains model descriptions. BLUPF90 does not use labels for effects.
* BLUPF90 does not differentiate between data types (integer or real). All data are read as real values and internally converted as needed.
* BLUPF90 prints minimal output to the screen. It does not create log files.
-->

### MiX99 ###

* BLUPF90 does not use a CLIM-style instruction file. Instead, it uses a more primitive way to specify a model and relationship matrices.
* BLUPF90 is mainly optimized for specific workflows with large datasets in animal breeding.
* BLUPF90 uses only two instruction files: a file for the data preparation (like a CLIM file) and a main parameter file for the analysis.
* BLUPF90 reads all the model specifications, initial variance components, and options from a single parameter file (which may read a few additional files).
* BLUPF90 does not create many temporary files.
* BLUPF90 needs another program to add the original IDs to the solutions (BLUE and BLUP).


Differences in purpose
----------------------

BLUPF90 is specialized for estimating BLUEs, predicting BLUPs, and estimating variance components in linear mixed models. It assumes that the user knows which fixed effects influence the phenotypes before the analysis. This is why the programs do not perform hypothesis testing for fixed effects. The goodness of fit of the model, including its random effects, can be evaluated using $-2\log L$, which is provided by the REML programs.

Difference in software design
-----------------------------

The philosophy of the BLUPF90 programs is described in the official wiki and several publications. The basic idea is to support general linear mixed models with minimal programming effort. Fortran 90 (and later) makes it easy to write and reuse code. BLUPF90 is the main software that demonstrates this idea, and many other programs have been derived from it.

The current programs support genomic analyses, especially single-step GBLUP. Computation times have been significantly reduced for REML, Gibbs sampling, and BLUP with iteration on data through parallelization and optimized libraries. The development team continues to actively update the programs to implement new ideas and improve stability.

BLUPF90 programs rely entirely on user-supplied information. This is intentional, to keep the software as simple as possible. The programs do not automatically create additional effects or covariates for convenience. For example, the general mean is not automatically added as a fixed effect when the model does not include one. In random regression models, the programs do not automatically generate covariates (e.g., Legendre polynomials or spline functions). This reminds users to provide all the information required by the model.
