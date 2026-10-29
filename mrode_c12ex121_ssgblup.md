---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Single-step approaches: basic principle
=======================================

Model
-----

We consider the standard animal model
$$
\mathbf{y}=\mathbf{Xb}+\mathbf{Wa}+\mathbf{e}.
$$
If some animals are genotyped, their additive relationships are described with the genomic relationship matrix ($\mathbf{G}$). When the genotyped and the non-genotyped animals are simultaneously considered in the relationship matrix, the resulting matrix is $\mathbf{H}$. Its inverse falls into a simple form.
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
This $\mathbf{G}^{-1}$ is usually calculated from a "blended matrix" with the pedigree matrix ($\mathbf{G}_{w}$ shown in the previous section). The system of mixed model equations is the same as the standard animal model with $\mathbf{H}^{-1}$ instead of $\mathbf{A}^{-1}$:
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z}\\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z}+\mathbf{H}^{-1}/\sigma_a^2
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

In Example 12.1, the author applies this model with $\sigma_a^2 = 35.241$ and $\sigma_e^2 = 245.0$. BLUPF90+ fully supports ssGBLUP described in this section using a minimal description in the parameter file.


Files
-----

In this analysis, the author uses the same data as shown in Example 11.1 (for the SNP-effect model) but extended to have non-genotyped animals. The data file is the same as before in Example 11.3 (but prepared in a separate file, `mr121.dat`); animals 13 to 20 are defined as the reference population, and the rest of animals have been removed. The pedigree information is common to the previous analysis, too (`mr121.ped`).

The SNP marker file (`mr121.gen`) is as follows.

~~~~~{language=text caption="mr121.gen"}
13 2011000212
14 1000020210
...
26 1011020100
~~~~~

The corresponding cross-reference file (`mr121.xrf`) is as follows. It has two columns with the same animal code for genotyped animals.

~~~~~{language=text caption="mr121.xrf"}
13 13
14 14
...
26 26
~~~~~

The parameter file (`mr121.par`) is shown as follows.

~~~~~{language=blupf90 caption="param_mr11e.txt"}
DATAFILE
mr121.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
4  1 cross
1 26 cross
RANDOM_RESIDUAL VALUES
245.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE
mr121.ped
(CO)VARIANCES
35.241
OPTION solv_method FSPAK
OPTION SNP_file mr121.gen mr121.xrf
OPTION no_quality_control
OPTION no_tunedG_first
~~~~~

The option `OPTION no_quality_control` is needed bacause of too few SNP markers producing inaccurate $\mathbf{G}$ in this example. Without this option, BLUPF90+ (actually the embedded genomic module) may stop because of the very low correlation between diagonals from $\mathbf{G}$ and $\mathbf{A}_{22}$. The correlation should be usually high enough; otherwise, there may be a problem in the quality of the genotypes or the pedigree.

By defailt, BLUPF90+ performs the same operatons (constructing $\mathbf{G}$ and $\mathbf{A}_{22}$, blending them into a single matrix, tuning the blended matrix, and calculare the inverses) as described in the textbook. The default blending is $\mathbf{G}_w = 0.95\mathbf{G} + 0.05\mathbf{A}_{22}$ and the tuning method is the same as in the textbook. So, you don't have to specify these options in the parameter file.

Note that the user must recognize the order of blending and tuning and how these pretaions are performed given software. This causion is also given by the textbook. Such subtle operational differences may impact on genomic predictions. For example, the textbook performs the blending first, then applies tuning. This order had been the default of the BLUPF90 family. But the current BLUPF90 programs do these opposite: tuning first, then blending later. BLUPF90+ may also apply 

The option `OPTION no_tunedG_first` retrieves the previous behavior aligned with the textbook. With this option, you will see the tuning parameters in the displayed message on screen.

~~~~~{language=shell}
 Tuning: Scale G matrix according to A22 -  Method: 2
    Diagonal A:    1.000    Offdiagonal A:    0.148    All A:    0.209    Difference:   0.852
    Diagonal G:    1.125    Offdiagonal G:   -0.075    All G:    0.010    Difference:   1.201
    Diff G Diag - G OffDiag:    1.201    (da-oa)/(dg-og):    0.709
    Diff A OffDiag - G OffDiag:    0.224
    Diff A all - G all:    0.199
    New Alpha:   0.674   New Beta:   0.035
    a + G*b; a:  0.202       b:  0.709
~~~~~

Solutions
---------

When using the abobe parameter file with the latest version of BLUPF90+, we can obtain the same solutions as shown in the textbook. However, as mentioned before, the default BLUPF90+ (without `OPTION no_tunedG_first`) will give s lightly different solutions because of the order of blending and tuning.

<!--
Using the default options in the parameter file, BLUPF90+ will produce slightly different solutions to those in the textboof (p.217). This is because BLUPF90 tried to compute and use more "accurate" tuning parameters by excluding some elements from the computation, whereas the textbook uses the whole relationship matrices to get the tuning parameters. More preciselly, BLUPF90+ excludes some $\mathbf{A}_{22}$ elements close to 0 to avoid biased results. The current numerical example hits this condition, and the calculated tuning parameters will be different from the textbook. The BLUPF90+ displays the following log, and we can find the tuning parameters as $a=0.055$ and $b=0.866$.

According to the log, our data have $\bar{\mathbf{A}}_{22,diag}=1.0000$, $\bar{\mathbf{A}}_{22}=0.2089$, $\bar{\mathbf{G}}_{diag}=1.132$, $\bar{\mathbf{G}}=1.219$


In the above parameter file, we try to specify the tuning parameters ($a=0.202$ and $b=0.709$) explicitly.
-->
