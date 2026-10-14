---
title: Practical genomic analysis
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Overview of genomic features
============================

Model and implementation
------------------------

### Model

BLUPF90+ is designed to be optimal for single-step genomic BLUP (ssGBLUP), considering both genotyped and nongenotyped animals simultaneously in the linear mixed model. Nongenotyped animals are assumed to have pedigree information. The ssGBLUP model combines the genomic relationship matrix $\mathbf{G}$ for genotyped animals with the additive (numerator) relationship matrix $\mathbf{A}$ for pedigree animals. Furthermore, this method requires $\mathbf{A}_{22}$, the subset of $\mathbf{A}$ for genotyped individuals that is conformable with $\mathbf{G}$. The unified relationship matrix, $\mathbf{H}$, is a function of $\mathbf{G}$ and $\mathbf{A}$.

In the mixed-model equations, we need the inverse of the unified relationship matrix, $\mathbf{H}^{-1}$. This inverse has a simple mathematical form.
$$
\mathbf{H}^{-1}
=
\mathbf{A}^{-1}
+
\left[
\begin{array}{cc}
\mathbf{0}&\mathbf{0}\\
\mathbf{0}&\mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}
\end{array}
\right]
$$
The ssGBLUP model uses $\mathbf{H}^{-1}$ instead of $\mathbf{A}^{-1}$ in the animal model. The resulting mixed-model equations are the same as those shown in the Quick Tour chapter.
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z}\\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z}+\mathbf{H}^{-1}/\sigma_u^2
\end{array}
\right]
\left[
\begin{array}{c}
\mathbf{\hat{b}}\\
\mathbf{\hat{u}}
\end{array}
\right]
=
\left[
\begin{array}{l}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{y} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{y} \\
\end{array}
\right].
$$

### Genomic features in BLUPF90

The BLUPF90 family has a module that contains a set of functions for handling and manipulating genomic data and derived matrices. The module is incorporated into almost all programs in the BLUPF90 family, including BLUPF90+ and GIBBSF90+. Therefore, most major BLUPF90 programs have the same operational features.

In addition, there is a separate program, PREGSF90 (`preGSf90`). This program is an interface to the genomic module and focuses on preparation tasks such as quality control of genomic data and pre-calculation of the genomic relationship matrix. Whereas the main BLUPF90 programs can perform both preparation and single-step analysis in a single run, PREGSF90 performs only the preparation step. PREGSF90 is useful when you want to separate preparation from mixed-model analysis, for example, when preparation takes a long time or when you want to prepare different relationship matrices.

PREGSF90 reads the same parameter file as BLUPF90+. From this file, the program reads the names of the pedigree file, the marker file, the optional cross-reference file, and some options. The rest of the parameter file (for example, the name of the data file and the model) is effectively ignored. Note that PREGSF90 refuses options that are not used by the program, such as `solv_method` and `maxrounds` for BLUPF90+. When PREGSF90 detects such options, it stops with a warning message.

The behavior of the genomic module is controlled by various options, specified as `OPTION` lines at the bottom of the parameter file. You can find many of these options in the manual. The major options are also displayed when you type the following command in your shell or Command Prompt.

~~~~~{language=shell}
preGSf90 --help
~~~~~

This tutorial explains some of the basic options useful in ssGBLUP.


Steps in ssGBLUP
----------------

A practical ssGBLUP analysis consists of at least three parts: preparation, mixed-model analysis, and post-mixed-model analysis. Here, we look at how BLUPF90 programs perform each step.

### Order of preparation steps

The stability of ssGBLUP depends on the characteristics of $\mathbf{H}^{-1}$, which contains $\mathbf{G}^{-1}$ and $\mathbf{A}_{22}^{-1}$. We have several ways to adjust $\mathbf{G}$ and $\mathbf{G}^{-1}$ to obtain reasonable results in genomic analyses. There are also several options for harmonizing $\mathbf{G}^{-1}$ and $\mathbf{A}_{22}^{-1}$. Most of these options are applied during the preparation step in the genomic module.

Currently, the preparation proceeds as follows.

1. Check and read the provided files.
2. Create $\mathbf{A}_{22}$ from the pedigree.
3. Apply quality control to the genomic data.
4. Create $\mathbf{G}$.
    - Create the raw $\mathbf{G}$ from genomic data.
    - Tune the raw $\mathbf{G}$ to be aligned with $\mathbf{A}_{22}$. (tuning)
    - Blend the tuned $\mathbf{G}$ with $\mathbf{A}_{22}$. (blending)
5. Calculate $\mathbf{A}^{-1}$.
6. Calculate $\mathbf{G}^{-1}$.
7. Calculate $\mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}$ as a single matrix by default.

Note that, in current BLUPF90 programs, the tuning operation comes first, followed by blending. In the past, the order was the opposite (blending -> tuning). This subtle change may produce differences in genomic predictions. The following option retrieves the previous order: blending first, followed by tuning.

~~~~~{language=blupf90}
OPTION no_tunedG_first
~~~~~

### Mixed-model analysis

The BLUPF90 programs can perform the preparation and mixed-model analysis in a single workflow. BLUPF90+ and GIBBSF90+ are the major programs for predicting breeding values and estimating variance components based on the ssGBLUP model.

The programs may need additional options to control the details of the model and equations. The output of ssGBLUP is similar to that of the standard pedigree-based model, as seen in the previous chapters.

### Post mixed-model analysis

Some genomic analyses are performed in addition to the mixed-model analysis, such as single-step GWAS, model cross-validation, and deregression of estimated breeding values. The BLUPF90 family provides several programs for performing these post-mixed-model analyses using the output of the mixed-model analysis. For example, POSTGSF90 (`postGSf90`) performs single-step GWAS based on the data and solution file prepared in the previous steps.


Structure of this chapter
-------------------------

In the next three sections, we briefly introduce the preparation steps: required files, quality control of genomic data, and construction of relationship matrices with tuning and blending. In these sections, we will see various options in the parameter file for controlling the genomic module, but they are only part of the full implementation. Please check the manual for details.

Then, we move to genomic analyses. We will briefly explain a typical workflow for ssGBLUP instead of explaining the complete ssGBLUP options using numerical examples. The numerical examples are available in the Quick Tour chapter and in the book by Mrode and Pocrnic (2023). Then, we examine some GBLUP options rather than ssGBLUP options because performing GBLUP may be tricky with tools designed for ssGBLUP. Finally, we look at a basic feature of POSTGSF90, a single-step GWAS tool for post-mixed-model analysis.

For a better understanding, it is helpful to run the small examples yourself. The next chapter presents numerical examples taken from Mrode and Pocrnic (2023) using BLUPF90 files. That chapter will cover many detailed features of the BLUPF90 programs. Thus, this chapter provides only the basics of the genomic options in BLUPF90.
