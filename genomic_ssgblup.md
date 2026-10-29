---
title: Practical genomic analysis
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Workflow of single-step GBLUP
=================================

Recommended steps
-----------------

BLUPF90+ and GIBBSF90+ perform the preparation and statistical analysis in a single operation. If the configuration is appropriate or the analysis is routine, the one-step method is the most efficient. However, if you need exploratory analyses, you may have to repeat particular steps many times. In this case, it is generally better to break the entire operation into separate steps.

The following is the standard workflow in genomic analysis using the BLUPF90 family.

1. Prepare raw data and files that have been checked and cleaned by the user.
2. Prepare the data by RENUMF90.
3. Apply quality control to the genomic data using PREGSF90 (or QCF90) and obtain cleaned data. Stop before computing the relationship matrix. QCF90 can take non-renumbered files, so this step can be performed before renumbering.
4. Run PREGSF90 to compute relationship matrices and save them to files. Prepare the parameter file only for this purpose.
5. Run BLUPF90+ or GIBBSF90+ to perform the mixed-model analysis by reading the prepared matrices from files.
6. Check the results.
7. If needed, run POSTGSF90 or another program for post-mixed-model analysis.

The user should ensure that each step gives correct results, as expected. If there are issues, the user should return to the previous step and resolve the problems.

Model consideration
-------------------

The single-step GBLUP (ssGBLUP) framework needs more configuration in its setup in addition to the standard considerations in the mixed linear model. Such configuration may include the following points.

- Which allele frequency and method should be used to calculate $\mathbf{G}$?
- How should $\mathbf{G}$ be tuned to be compatible with $\mathbf{A}_{22}$?
- How should $\mathbf{G}$ be blended with $\mathbf{A}_{22}$?
- Which parameters can be changed to reduce bias and increase the accuracy of genomic predictions?
- What should we do if $\mathbf{G}$ (and $\mathbf{A}_{22}$) is too large to compute and store?
- Which model is most appropriate when parents are missing?
- Which approach is best for a multibreed population?

This tutorial does not answer the above questions. It also does not discuss techniques already implemented in the BLUPF90 programs. Some of the questions above are research topics, and there may be no general answer that applies to all cases.

Program behavior is largely defined by options in the parameter file. Therefore, an important caution is that we must understand the default behavior and parameters of the BLUPF90 programs. Without explicit specifications, each program uses its own default values. These values are not necessarily optimal and may be arbitrary. This applies to any model but is especially important for genomic models. Developers may change default values or even the behavior of the genomic setup if they consider a change necessary or identify a problem with the previous behavior. It is always good to specify options explicitly in the parameter file to improve reproducibility.

The best way to establish a good ssGBLUP setup is to continue following the research and learn more about the model through the course materials presented on the official UGA website or by other research teams. Alternatively, users can attend courses taught by active researchers in the field. For software-related questions, the BLUPF90 mailing list (forum) may be a useful place to ask.
