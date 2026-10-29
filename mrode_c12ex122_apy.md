---
title: Numerical examples from Mrode and Pocrnic (2023)
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

APY approach
============

Model
-----

Please see the textbook for details about the APY approach.

This section is somewhat unclear about which data and which model (GBLUP or ssGBLUP) the author actually used. In addition, the files in the GitHub repository are inconsistent with the description in the textbook and are incomplete for carrying out the analysis. Here, we will analyze a specific data set using GBLUP.

Files and solutions
-------------------

The files in the GitHub repository are useful for seeing what the APY approach calculates. The parameter file (`mr122.par`) saves $\mathbf{G}_{APY}^{-1}$ to a text file. The SNP markers are sorted according to whether the animals belong to the core or non-core group. This is done to show the structure of $\mathbf{G}_{APY}^{-1}$, as in the textbook. You do not have to sort the genotyped animals in this way to solve the mixed model equations.

The structure of the cross-reference file (`mr122.xrf`) differs from those of the previous files. The file should have an additional column (the third column, for example) defining which animals are in the core group. In the following example, the core animals have `1` in this column, and the other animals have `0`.

~~~~~{language=text caption="mr122.xrf"}
15 15 1
20 20 1
22 22 1
26 26 1
13 13 0
14 14 0
16 16 0
17 17 0
18 18 0
19 19 0
21 21 0
23 23 0
24 24 0
25 25 0
~~~~~

With the parameter file given, BLUPF90+ generates `Gi_Orig.txt`, which contains $\mathbf{G}_{APY}^{-1}$. The option `saveGInverseOrig` puts the original animal code (corresponding to the pedigree file) in the saved file. The following shows the first several lines of this file.

~~~~~{language=text caption="Gi_Orig.txt"}
15 15 4.42519817
20 15 .37303282
22 15 -.52701071
26 15 .41157969
13 15 -.79477898
14 15 2.73078285
16 15 -.34898884
17 15 -.65833127
18 15 .95861819
19 15 .77554250
21 15 .48329251
23 15 -.44703890
24 15 -.10841053
25 15 1.82768394
...
~~~~~

The resulting solutions are not comparable with the previous models because data, pedigree, and options are not the same as before. Here, we will conduct ssGBLUP with the APY approach so that we can compare the results with those from Example 12.1 in the previous section.

We copy the previous data file (`mr121.dat`) to `mr122a.dat`, while retaining the original pedigree file (`mr122.ped`), marker file (`mr122.gen`), and cross-reference file (`mr122.xrf`). We also copy the parameter file from the previous section (`mr122.par`) to `mr122a.par` and modify it as follows.

~~~~~{language=blupf90 caption="mr122a.par"}
DATAFILE
mr122a.dat
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
mr122.ped
(CO)VARIANCES
35.241
OPTION solv_method FSPAK
OPTION SNP_file mr122.gen mr122.xrf
OPTION no_quality_control
OPTION no_tunedG_first
OPTION AlphaBeta 0.95 0.05
OPTION apy 1 3 core.eq.1
~~~~~

We should change the names of the data, marker, and cross-reference files and add one more option: `OPTION apy 1 3 core.eq.1`.

The solutions for animals 13 to 26 are shown below.

~~~~~{language=text}
 ID  Normal     APY
 13   0.048   0.042
 14   0.082   0.158
 15   0.039   0.036
 16   0.202   0.301
 17  -0.356  -0.340
 18  -0.254  -0.228
 19   0.098  -0.036
 20  -0.169  -0.167
 21   0.018   0.068
 22   0.079   0.081
 23  -0.164  -0.152
 24   0.101   0.068
 25   0.039  -0.002
 26   0.245   0.246
~~~~~

The `Normal` column is from the regular $\mathbf{G}^{-1}$ shown in Example 12.1 (p. 217). The `APY` column contains the current solutions, which are not the same as those in the textbook (p. 224). The two sets of solutions are not identical but are similar. As mentioned in the textbook, the solutions will be sufficiently close if we have more markers and enough genotyped animals.
