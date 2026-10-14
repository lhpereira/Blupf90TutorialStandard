---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

GBLUP
=====

Model
-----

Under some assumptions, the SNP marker model is equivalent to a model called GBLUP. This model includes a genomic relationship matrix, $\mathbf{G}$, which is calculated from SNP marker data. It is analogous to the pedigree-based additive relationship matrix, although there is an essential difference between the two matrices; see VanRaden (2008) for details. The $\mathbf{G}$ matrix may be defined in several ways, but the best-known one is VanRaden's method 1, defined as
$$
\mathbf{G}=\frac{\mathbf{ZZ}'}{2\sum_j p_j(1-p_j)}
$$
where $\mathbf{Z}$ contains genotypes adjusted for allele frequency and $p_j$ is the allele frequency for marker $j$ (VanRaden 2008). When the number of genotypes is limited, $\mathbf{G}$ is likely to be singular, and its inverse does not exist. To avoid this issue, we blend $\mathbf{G}$ with a numerator relationship matrix for genotyped animals ($\mathbf{A}_{22}$). Therefore, as long as we use this blending, the pedigree file is still needed. The blending formula is
$$
\mathbf{G}_w = w\mathbf{G}+(1-w)\mathbf{A}_{22}
$$
where $\mathbf{G}_w$ is the final (weighted) genomic relationship matrix and $w$ is a weight. When $\mathbf{A}_{22}$ is not available, $\mathbf{I}$ may be useful. An appropriate weight usually ranges from 0.80 to 0.99, depending on the profile of $\mathbf{G}$. This blending is expected to make $\mathbf{G}_w$ positive definite. The blended matrix, $\mathbf{G}_w$, will be used as the genomic relationship matrix for prediction. Hereafter, we will refer to the inverse of $\mathbf{G}_w$ as $\mathbf{G}^{-1}$.

The system of mixed model equations is
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{W}\\
\mathbf{W}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{W}'\mathbf{R}^{-1}\mathbf{W}+\mathbf{G}_{w}/\sigma_u^2
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
\mathbf{W}'\mathbf{R}^{-1}\mathbf{y} \\
\end{array}
\right]
$$
and $\sigma_u^2 = 35.241$ and $\sigma_e^2 = 245.0$. In the simplest case, only the general mean ($\mu$) is considered as the fixed effect, and $\hat{\mathbf{b}}$ reduces to $\hat{\mu}$.

In contrast to the general description of the GBLUP procedure, the textbook uses the simplest case. As stated on p. 194, the blended matrix is formed as $\mathbf{G}_w = \mathbf{G} + 0.01\mathbf{I}$, and the user does not need pedigree information. This greatly simplifies the GBLUP operation in BLUPF90+. We will show a straightforward approach for performing this analysis using BLUPF90+ in this section.

In the appendix, we demonstrate how a general blended matrix, $\mathbf{G}_w = w\mathbf{G} + (1-w)\mathbf{A}$, can be obtained and how to perform GBLUP with $\mathbf{G}^{-1}$ without using the pedigree matrix, with the BLUPF90 family. This situation was explained in the previous edition of the textbook. You should use this method with care because it may seem tricky and overly complicated.

Files
-----

This model does not need a pedigree file, so you only need to prepare three files (data, SNP marker, and cross-reference files) in addition to a parameter file.

The SNP file is as follows (`mr113.gen`). The format of the SNP file is described in a separate chapter. 

~~~~~{language=text caption="mr113.gen"}
13 2011000212
...
26 1011020100
~~~~~

The accompanying cross-reference file (`mr113.xrf`) is also needed. This file relates the IDs of genotyped animals to their order in the marker file. In practice, this file will be created by RENUMF90 from the marker file. A detailed explanation is also available in the "genomics" chapter.

~~~~~{language=text caption="mr113.xrf"}
13 13
14 14
15 15
16 16
17 17
18 18
19 19
20 20
21 21
22 22
23 23
24 24
25 25
26 26
~~~~~

Contrary to what one might expect, we still need a pedigree file because the BLUPF90 family is designed to read pedigree information. Here, we prepare a _dummy_ pedigree file in which all animals have missing parents. The following shows part of the dummy pedigree file (`mr113.ped`).

~~~~~{language=text caption="mr113.ped"}
1 0 0
2 0 0
...
25 0 0
26 0 0
~~~~~

The data file (`mr113.dat`) is as follows. Note that the animals in this file are defined as the reference population. The other animals are removed because they are subject to prediction.

~~~~~{language=text caption="mr113.dat"}
13  0  0 1 558  9.0
14  0  0 1 722 13.4
15 13  4 1 300 12.7
16 15  2 1  73 15.4
17 15  5 1  52  5.9
18 14  6 1  87  7.7
19 14  9 1  64 10.2
20 14  9 1 103  4.8 
~~~~~

1. The original animal ID
2. The original sire ID
3. The original dam ID
4. general mean
5. EDC
6. Observation (Fat DYD)

The parameter file (`mr113.par`) for BLUPF90+ is as follows.

~~~~~{language=blupf90 caption="mr113.par"}
DATAFILE
mr113.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6
WEIGHT(S)
7
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
mr113.ped
(CO)VARIANCES
35.25
OPTION SNP_file mr113.gen mr113.xrf
OPTION no_quality_control
OPTION AlphaBeta 1.00 0.00
OPTION GammaDelta 0.01 0.00
OPTION tunedG 0
~~~~~

The following is a brief description of the options used in the parameter file.

- `SNP_file` = Invoking the genomic module. It has 2 values in this case: the name of the marker file and the name of the cross-reference file.
- `no_quality_control` = Turns off the quality control process to remove unqualified markers and genotyped animals. In this example, we will use all the markers and genotyped animals without checks.
- `AlphaBeta` and `GammaDelta` = Define weights $\alpha$, $\beta$, $\gamma$, and $\delta$ for $\mathbf{G}_w=\alpha\mathbf{G}+\beta\mathbf{A}_{22} + \gamma\mathbf{I} + \delta\mathbf{1}\mathbf{1}'$. In this example, we set $\alpha = 1.00$, $\beta=0.0$, $\gamma=0.01$, and $\delta=0.0$. Thus, we do not use pedigree relationships for blending.
- `tunedG` = Controls the tuning method for $\mathbf{G}_w$. It tries to adjust the scale of $\mathbf{G}$ to $\mathbf{A}_{22}$ using a method described by Chen et al. (2011). Tuning is _a different step from blending_. The argument 0 turns off the adjustment (that is, it uses $\mathbf{G}_{w}$).

Note that, in this example, you can also use `OPTION AlphaBeta 1.00 0.01` and remove `OPTION GammaDelta`. BLUPF90+ will create $\mathbf{A}_{22}$ from the dummy pedigree; it is the identity matrix, $\mathbf{I}$. Thus, the alternative option gives the same $\mathbf{G}_w$ as the parameter file shown above.

A later chapter will explain the above options in detail.

Solutions
---------

The solution file contains zero solutions for non-genotyped animals, and the breeding value solutions are available only for genotyped animals. The predictions are also available for animals with genotypes but without phenotypes. Those solutions are identical to the ones in the textbook (p.192).


Appendix: use of G-matrix blended with the pedigree relationships
-----------------------------------------------------------------

This subsection does not deal with the example in the textbook. The reader can skip this appendix and move to the next section without any loss. A similar explanation will also be given in the GBLUP section of a later chapter.

### Introduction: general GBLUP solution in the BLUPF90 family

The BLUPF90+ family supports general GBLUP, but its operation is often cumbersome. One reason is that BLUPF90+ is designed to handle a model called single-step genomic BLUP. This model assumes that two relationship matrices, $\mathbf{G}$ and $\mathbf{A}$, are available and uses their inverses in the mixed model equations. In the GBLUP model explained above, the blended matrix $\mathbf{G}_w$ uses pedigree relationships, whereas only genomic relationships are required in the equations. BLUPF90+ does not readily support this selective use of pedigree relationships across operations, so tricks are needed to handle this case.

<!-->
The BLUPF90 family is designed for the single-step GBLUP model which has the inverse of a numerator relationship matrix ($\mathbf{A}^{-1}$). It means that the final equations should include $\mathbf{A}^{-1}$, which is not usually used in GBLUP. There are several ways to perform GBLUP with BLUPF90+. 
-->

Here, I introduce a method with the following two steps: (1) preparation of $\mathbf{G}^{-1}$ and (2) construction and solution of the equations. In step 1, we can use PREGSF90. This program accepts the same parameter file as BLUPF90+, calculates $\mathbf{G}^{-1}$ with various options, and saves it to a file. BLUPF90 can read the file and use it to construct the mixed model equations. BLUPF90+ supports two file formats for $\mathbf{G}^{-1}$: a text file and a binary file. The text format is simpler, especially for a small data set, and makes it easier to understand how to run GBLUP using BLUPF90+. The binary format is more efficient, but it requires a trick that mimics single-step GBLUP while removing the pedigree relationship matrix. In this section, first, we use the text format to demonstrate how $\mathbf{G}^{-1}$ can be calculated with PREGSF90 using an educational parameter file (approach 1). Then, we move to the second approach, which uses a binary file and is fully aligned with the single-step GBLUP context.

Note that the methods introduced here may be overly complicated. One reason is the issue of handling the pedigree. Another is that we skip RENUMF90 to show the minimal operations. Using the renumbering tool, GBLUP could be performed more easily. In a later chapter, we will see how RENUMF90 prepares the files required for pure GBLUP.

In both approaches, we will use the common marker file (`mr113.gen`) and the cross-reference file (`mr113.xrf`) as used in the textbook example.

### Approach 1: a text file for G-inverse

As mentioned before, first, we calculate $\mathbf{G}^{-1}$ and store its elements in a text file. The inverse is based on the blended matrix $\mathbf{G}_w = w\mathbf{G} + (1-w)\mathbf{A}_{22}$.

We will use PREGSF90 to perform this operation using a trick. We use the data file from before, except that the last column for genotyped animals has different coding (`mr113a.dat`). Note that PREGSF90 does not actually use the data file, but we need to provide it because the program checks whether the data content is valid.

~~~~~{language=text caption="mr113a.dat"}
13  0  0 1 558  9.0  1
14  0  0 1 722 13.4  2
15 13  4 1 300 12.7  3
16 15  2 1  73 15.4  4
17 15  5 1  52  5.9  5
18 14  6 1  87  7.7  6
19 14  9 1  64 10.2  7
20 14  9 1 103  4.8  8
~~~~~

1. The original animal ID
2. The original sire ID
3. The original dam ID
4. general mean
5. EDC
6. Observation (Fat DYD)
7. new ID for genotyped animals (in the order of the marker file - _this is important_)

The key is the seventh column. Our trick needs a new ID corresponding to the order in the marker file. For example, animal 13 appears in the first row of the marker file and has 1 as its new ID. PREGSF90 does not read this column, but BLUPF90+ uses it to perform GBLUP. Later, we will discuss the new ID in detail.

The pedigree file (`mr113a.ped`) is the original one; it contains all animals, with and without genotypes, as shown before. PREGSF90 creates a subset of the relationship matrix ($\mathbf{A}_{22}$) for the genotyped animals only.

The parameter file for PREGSF90 is the same as BLUPF90+ with several options.

~~~~~{language=blupf90 caption="mr113a_pre.par"}
# For PREGSF90 in approach 1
DATAFILE
mr113a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6
WEIGHT(S)
7
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
mr113a.ped
(CO)VARIANCES
35.25
OPTION SNP_file mr113.gen mr113.xrf
OPTION no_quality_control
OPTION AlphaBeta 0.99 0.01
OPTION tunedG 0
OPTION saveAscii
OPTION saveG
OPTION saveGInverse
OPTION createGimA22i 0
~~~~~

Using the parameter file, PREGSF90 will create several files, including `Gi`, which contains the inverse of the blended matrix, $\mathbf{G}^{-1}$. PREGSF90 manipulates $\mathbf{G}_{w}$ in several ways. The options in the parameter file control PREGSF90's behavior and produce the expected $\mathbf{G}_{w}$. Note that the options are case-sensitive. The following is a brief description of the options used in the parameter file.

- `SNP_file`, `no_quality_control`, `AlphaBeta`, `tunedG` = As explained in the previous subsection.
- `saveAscii` = Saves $\mathbf{G}_{w}^{-1}$ in a text file. Without this option, the program saves the matrix to a binary file (a machine-readable file).
- `saveG` = Saves $\mathbf{G}_w$ to a file. The file name will be `G`. You do not have to specify this option in this case; we use it for educational purposes to see the actual values in $\mathbf{G}_{w}$.
- `saveGInverse` = Saves $\mathbf{G}^{-1}$ to a file. The file name will be `Gi`.
- `createGimA22i` = Turns off the calculation of $\mathbf{G}_{w}^{-1}-\mathbf{A}_{22}$ which is required in single-step GBLUP. You do not have to put this option in this small example because the computing cost is negligible. Turning off this option is important for a large-size problem in GBLUP.

PREGSF90 has more than 100 options. You can see the major ones by running `preGsf90 --help` or by consulting the manual. Many of them were implemented for experimental purposes and are not currently used. We will demonstrate some major options in a later chapter.

When running PREGSF90, you will see the basic statistics for $\mathbf{G}_{w}$ on the screen. Also, you will find that some files are created.

- `Gen_call_rate` = Empty in this case. Just ignore it.
- `freq.count` = Allele frequency for each marker. The values should be the same as the textbook (see p.181).
- `sum2pq` = $2\sum_j p_j(1-p_j)$
- `G` = Elements in $\mathbf{G}_w$. The values are slightly different from the textbook (p.186) because the textbook values are from the pure $\mathbf{G}$ (not blended with $\mathbf{A}_{22}$). You can find the same values as the textbook when you put `OPTION AlphaBeta 1.0 0.0`, but the program cannot calculate its inverse because it is singular.
- `Gi` = Elements in $\mathbf{G}^{-1}$. Only the diagonals and the upper triangular elements are stored.

Among the above files, only `Gi` is required in the analysis with BLUPF90. The first 5 rows and the last 5 rows in this file are as follows.

    1 1 36.374112147756
    1 2 -27.980861774302
    2 2 88.270926666258
    1 3 -8.255548353247
    2 3 31.300275470378
    ...
    10 14 2.733882533018
    11 14 6.555679888597
    12 14 5.120296531692
    13 14 4.504947248223
    14 14 6.133181220876

The first two integers are the row and column indices for the element. Note that the maximum row and column ID is 14 because the rank of this matrix ($\mathbf{G}^{-1}$) is 14. Thus, the row/column index does not directly refer to an animal's ID. When creating $\mathbf{G}$, PREGSF90 takes only genotyped animals and reassigns their internal IDs according to their order in the marker file. For example, row 1 and column 1 (the first line in the file above) correspond to the relationship between animals 13 and 13 (that is, the diagonal for animal 13), because animal 13 appears first in the marker file (animal 13 = new ID 1). Row 1 and column 2 (the second line) contain the relationship between animals 13 and 14 because animal 14 appears second in the marker file (animal 14 = new ID 2). Therefore, we should use the new IDs to refer to genotyped animals.

We have already prepared the data file with the new ID (the 7th column). BLUPF90+ can read the column as an animal ID.

The following shows the parameter file for BLUPF90+ using the output of PREGSF90 (file `Gi`).

~~~~~{language=blupf90 caption="mr113a_gblup.par"}
# For BLUPF90+ in approach 1
DATAFILE
mr113a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
4  1 cross
7 14 cross  # new ID (renumbered only for genotyped animals)
RANDOM_RESIDUAL VALUES
245.0
RANDOM_GROUP
2
RANDOM_TYPE
user_file
FILE
Gi
(CO)VARIANCES
35.25
OPTION solv_method FSPAK
~~~~~

We use the keyword `user_file` to read the text file containing $\mathbf{G}^{-1}$. With this keyword, we no longer need the marker file. BLUPF90+ simply reads the inverse-matrix elements from the file and inserts them into the equations. With this parameter file, the solutions look slightly different from, but are still similar to, the values in the textbook (results not shown).


### Approach 2: a binary file for G-inverse
-------------------------------------------

In the second approach using a binary file instead of the text format, we will align with the single-step GBLUP framework. The parameter file for PREGSF90 is as follows.

~~~~~{language=blupf90 caption="mr113b_pre.par"}
# For PREGSF90 in approach 2
DATAFILE
mr113a.dat
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
mr113a.ped
(CO)VARIANCES
35.25
OPTION SNP_file mr113.gen mr113.xrf
OPTION no_quality_control
OPTION AlphaBeta 0.99 0.01
OPTION tunedG 0
OPTION TauOmega 1.0 0.0
~~~~~

This parameter file is similar to the previous one but has fewer options. The program will save the matrix files in binary format (without `saveAscii`). The option `TauOmega 1.0 0.0` ignores $\mathbf{A}_{22}$ after blending. As a result, PREGSF90 will save $\mathbf{G}^{-1}$ to a binary file, `GimA22i`. The file name may sound strange, but it is meaningful in the single-step GBLUP context. When you run PREGSF90 with this parameter file, you will see output on the screen very similar to the output shown above.

<!--
This parameter file is identical to the previous one except that the `saveAscii` option has been removed (commented out). Consequently, the G-inverse file (`Gi`) is saved in a binary format that is readable only by a computer. When you run PREGSF90 with this parameter file, you will see output on the screen very similar to the output shown above.
-->

<!--
One complication is that, when using this binary `Gi`, you cannot use the previous `user_file` approach. Therefore, you must follow the full single-step GBLUP framework. This means that you must prepare a pedigree file containing no information at all. We will use a dummy pedigree file, but it should differ from the previous one: it contains `0` for all IDs. It also contains 26 lines, the same number of lines as the original pedigree, so that it is compatible with the cross-reference file.
-->

<!--
A trickier fact is, using this binary `Gi`, you may not use the previous `user_file` approach. And therefore, you must follow the full single-step GBLUP framework. It means, you must prepare a pedigree file which should contain absolutely no information. We will use a dummy pedigree file, but it should be different from the prebious one: the file contains `0` for all IDs. Also, this file contains 26 lines, which is the same number of lines as the original pedigree and compatible with the cross-reference file.
-->

One complication is that, when using this binary `GimA22i`, you cannot use the previous `user_file` method. With this approach, we make BLUPF90+ behave as if single-step GBLUP were applied without pedigree information. This means that we must prepare a pedigree file containing no information at all. We will use a dummy pedigree file, but it should differ from the previous one: it contains `0` for all IDs. It also contains 26 lines, the same number of lines as the original pedigree, so that it is compatible with the cross-reference file.

~~~~~{language=text caption="mr113b.ped"}
0 0 0
...
0 0 0
~~~~~

BLUPF90+ needs the data, dummy (empty) pedigree, marker, and cross-reference files, as well as the binary file `GimA22i`. The parameter file is slightly different from the one used for PREGSF90.

~~~~~{language=blupf90 caption="mr113b_gblup.par"}
# For BLUPF90+ in approach 2
DATAFILE
mr113a.dat
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
2
OBSERVATION(S)
6
WEIGHT(S)

EFFECTS:
4  1 cross
1 26 cross  # use the original ID
RANDOM_RESIDUAL VALUES
245.0
RANDOM_GROUP
2
RANDOM_TYPE
add_animal
FILE        # empty pedigree
mr113b.ped
(CO)VARIANCES
35.25
OPTION SNP_file mr113.gen mr113.xrf
OPTION readGimA22i
OPTION solv_method FSPAK
~~~~~

This has the same specification as single-step GBLUP. In this case, the pedigree file is empty, and $\mathbf{A}^{-1}=\mathbf{0}$. With `OPTION readGimA22i`, BLUPF90+ reads `GimA22i` ($\mathbf{G}^{-1}$ in this case). As a result, BLUPF90+ constructs and solves the GBLUP equations.

<!--
It has the same specification for single-step GBLUP. In this case, the pedigree file is empty, and $\mathbf{A}^{-1}=\mathbf{0}$. With `OPTION readGInverse`, BLUPF90+ reads `Gi` ($\mathbf{G}^{-1}$). Also, `OPTION TauOmega 1.0 0.0` ignores $\mathbf{A}_{22}$. As a result, BLUPF90+ constructs and solves the GBLUP equations.
-->

With this parameter file, you will obtain solutions identical to those from approach 1. You will see many warnings (`Animal Id equal to Sire/Dam Id - relationships removed !!`) because we are using a trick that does not occur in regular single-step GBLUP.

Again, the approaches demonstrated here require tricky, manual operations. Other tricks will be presented in a later chapter.
