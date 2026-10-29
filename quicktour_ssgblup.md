---
title: Quick tour of BLUPF90
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Trivial analyses for single-step GBLUP
======================================

In a single-step genomic BLUP model (ssGBLUP), the BLUPF90 programs require 5 files: a parameter file, a data file, a pedigree file, a marker file, and a cross-reference file. The first 3 files are the same as those used in a regular animal model, so you need two additional files for ssGBLUP. In this section, we first describe the relationship matrices needed for the analysis. Then we introduce the 2 additional files and run the program.

Single-step GBLUP
-----------------

### Model and relationship matrices ###

To demonstrate ssGBLUP, we will use the same animal model introduced in the previous section.
$$
y_{ijk} = A_i + S_j + \beta x_k + u_k + e_{ijk}
$$
where $u_k$ is the additive genetic effect for animal $k$. We treat $S_j$ as a fixed effect. The mixed model equations are as follows.
$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{X}'\mathbf{R}^{-1}\mathbf{Z} \\
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{X} & \mathbf{Z}'\mathbf{R}^{-1}\mathbf{Z} + \mathbf{A}^{-1}/\sigma_u^2
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
\mathbf{Z}'\mathbf{R}^{-1}\mathbf{y}
\end{array}
\right]
$$
In ssGBLUP, $\mathbf{A}^{-1}$ is replaced with $\mathbf{H}^{-1}$, which has the following structure:
$$
\mathbf{H}^{-1}
=
\mathbf{A}^{-1}
+
\left[
\begin{array}{cc}
\mathbf{0} & \mathbf{0} \\
\mathbf{0} & \mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}
\end{array}
\right]
$$
where $\mathbf{G}$ is the genomic relationship matrix for genotyped animals and $\mathbf{A}_{22}$ is the subset of the numerator relationship matrix for genotyped animals. Thus, this method can combine both genotyped and non-genotyped animals in the same equations.

BLUPF90+ constructs $\mathbf{G}^{-1}$ and $\mathbf{A}_{22}^{-1}$ before solving the equations. The program also checks SNP markers and reports any errors it finds.

### Files ###

We can reuse the same data and pedigree files from the previous animal model. Nevertheless, we use different filenames for the data and pedigree files to avoid confusion.

~~~~~{language=text caption="data4.txt"}
  3.0  1  1  1.0   6
  2.0  1  2  1.0   9
  4.0  1  1  2.0  12
  6.0  2  2  2.0   7
  3.0  2  1  1.0  10
  6.0  2  2  2.0  13
  6.0  3  1  2.0   8
  6.0  3  2  1.0  11
  8.0  3  1  1.0  14
  4.0  3  2  2.0  15
~~~~~

The following is the pedigree file.

~~~~~{language=text caption="pedigree4.txt"}
  1   0   0
  2   0   0
  3   0   0
  4   0   0
  5   0   0
  6   0   0
  7   2   5
  8   1   4
  9   2   3
 10   7   6
 11   7   4
 12  11   8
 13  11  10
 14   9  13
 15  11  10
~~~~~

The parameter file is also the same as in the previous example, except for an option to read a marker file.

~~~~~{language=blupf90 caption="param4.txt"}
DATAFILE
data4.txt
NUMBER_OF_TRAITS
1
NUMBER_OF_EFFECTS
4
OBSERVATION(S)
1
WEIGHT(S)

EFFECTS:
2  3 cross
3  2 cross
4  1 cov
5 15 cross
RANDOM_RESIDUAL VALUES
2.0
RANDOM_GROUP
4
RANDOM_TYPE
add_animal
FILE
pedigree4.txt
(CO)VARIANCES
0.5
OPTION SNP_file snp4.txt snp4index.txt
~~~~~

The last line specifies the name of the marker file. The `SNP_file` option makes `blupf90+` read the marker file `snp4.txt` and the cross-reference file `snp4index.txt`, and invoke the genomic computations. You can omit the second argument (the name of the cross-reference file) if its name is the marker filename plus `_XrefID`. In this case, if the cross-reference file is `snp4.txt_XrefID`, you can omit the second argument in that option.

We assume that only 8 animals are genotyped (2, 4, 6, 10, 11, 12, 13, and 15). We also assume that each genotyped animal has an original ID and that the marker file contains these IDs (ID002, ID004, ID006, ID010, ID011, ID012, ID013, and ID015, respectively). The following is the marker file.

~~~~~{language=widetext caption="snp4.txt"}
ID002 1212020111001211212000102001202022201112000110020102011001012212110122100001210101112200210
ID004 2222020200110122210121001100022221102111000010011202020002002222000222000020120000020200200
ID006 2122020101100221121111212100112111112212000101012211111001201211112022100201220110022200220
ID010 2111121101111210121211112210202021211211000110021211111001101211101122100111221000022200120
ID011 2211121200121111220211011210112121111201000120011202020002002222000222000021221000021200110
ID012 2222020200110122210121001100022221102111000010011202020002002222000222000020120000020200200
ID013 2122020101210121121222101200112122102212000010012211111001101211101122100110220000021200210
ID015 2122020101210121121222101200112122102212000010012211111001101211101122100110220000021200210
~~~~~

The BLUPF90 family accepts only a marker file prepared with the following rules.

- A marker file is a text file with a fixed-width format containing 2 columns.
- The first column contains an animal's ID. You do not have to use the ID common to the data and pedigree files, so you can put the animal's original ID here. (The BLUPF90 family actually skips this column, so you can put any information here.)
- The second column contains an animal's genotypes. Each genotype can have one of 4 possible values: `0` (homozygote), `1` (heterozygote), `2` (alternative homozygote), and `5` (missing or unknown).
- The second column must start at a fixed position on each line. In the above example, the second column always starts at the 9th character.
- Each genotyped animal must have the same number of markers.
- You must not insert a space between two adjacent genotypes.
- You don't have to sort the marker file by ID.

The program will stop immediately if it finds inconsistencies in the file.

We need one more file to run `blupf90+`. The last file is a cross-reference file that relates the integer code used in `blupf90+` to the ID in the marker file. This is a text file with 2 columns separated by spaces.

~~~~~{language=text caption="snp4index.txt"}
2 ID002
4 ID004
6 ID006
10 ID010
11 ID011
12 ID012
13 ID013
15 ID015
~~~~~

The order of the animals must match that in the marker file. The first column contains the integer value (common to the pedigree file), and the second column contains the original ID found in the marker file. This file can be prepared automatically by `renumf90`.

Why do we use such a tricky example? Because the situation involving original IDs in the marker file actually occurs in real life. A farm animal usually has a unique (original) ID and phenotypes. Pedigrees and genotypes are recorded with the original ID, whereas `blupf90+` accepts only an integer value as an animal's ID. Therefore, we usually use `renumf90` to replace the original ID with integer values and create renumbered files. However, the marker file is very large, and replacing the original ID with an integer code is inefficient. RENUMF90 does not rewrite the marker file. Instead, RENUMF90 generates the cross-reference file to establish the correspondence between the IDs in the marker and pedigree files.


### Running the program ###

We can now run `blupf90+` with the 5 files. You can see a very long output from the genomic routine on the screen. We briefly explain what happens in this process.

- Read the pedigree information for genotyped animals and compute $\mathbf{A}_{22}$.
- Read SNP markers and perform quality control on the data (possibly remove unqualified markers).
- Compute $\mathbf{G}$ and compare it with $\mathbf{A}_{22}$.
- Compute $\mathbf{A}_{22}^{-1}$ and $\mathbf{G}^{-1}$.
- Finally, compute $\mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}$ as a single matrix and store it in a file.
- After the process, BLUPF90+ starts building the system of the mixed model equations with
  $\mathbf{G}^{-1} - \mathbf{A}_{22}^{-1}$ and solves it.

There are many options to precisely control the genomic routine. Each option has a default value. The user should know the default behavior of the program. This explanation is outside the scope of this introductory chapter. We will see the details in later chapters.

After running the program, we can find the solutions file.

~~~~~{language=text caption="solutions"}
trait/effect level  solution
   1   1         1      0.23340767
   1   1         2      2.19314788
   1   1         3      3.16386924
   1   2         1      2.28753998
   1   2         2      1.69948874
   1   3         1      0.52196308
   1   4         1      0.00700521
   1   4         2      0.08747780
   1   4         3     -0.01954304
   1   4         4      0.02639925
   1   4         5      0.07679027
   1   4         6     -0.18438116
   1   4         7      0.15271384
   1   4         8     -0.09012753
   1   4         9      0.01442435
   1   4        10     -0.23558188
   1   4        11      0.22278505
   1   4        12      0.10730577
   1   4        13      0.20708267
   1   4        14      0.32362808
   1   4        15     -0.21760840
~~~~~

Single-step GBLUP directly provides GEBVs (estimated breeding values enhanced with genomic information). The above solutions for the animals are GEBVs. The values are very different from those in the previous animal-model analysis. This is due to differences in the model as well as the small amount of data.


Remarks
-------

If your marker data set is relatively large, you should consider a more efficient approach. Each relationship matrix created in the genomic routine is stored fully in memory, i.e., as a dense matrix, consuming a lot of memory. The required memory is $8n^2/1024^3$ GB (gigabytes) for each relationship matrix with $n$ genotyped animals. Computation time will be longer with many genotyped animals. To determine appropriate options for obtaining stable GEBVs, you should run the program many times. You can save a relationship matrix to a file and reuse it.

The PREGSF90 program performs only the genomic computations. It can perform the same genomic computations as BLUPF90 (except for solving the mixed model equations). The best practice is to run `preGSf90` to check and clean up the markers, compute the relationship matrices, and save the matrices to files before running `blupf90+`. The program can read the files generated by PREGSF90, so you can save time.


Summary
-------

- BLUPF90 can perform a single-step GBLUP (ssGBLUP) analysis.
- With ssGBLUP, a marker file and a cross-reference file are needed in addition to the data, pedigree, and parameter files required for a regular animal model analysis.
- The parameter file must include an option for genomic analysis.
- There are many options to control genomic computations. See the manual.
- The marker file is a fixed-width text file with an animal's ID and genotypes.
- A cross-reference (XrefID) file relates the animal's ID in the marker file to the integer code in the pedigree file. This file is generated by RENUMF90.
- GEBVs are obtained directly as solutions.
- A separate program, PREGSF90, is useful for saving a relationship matrix to a file if you run the program several times using the same pedigree and genomic information. BLUPF90 reads the file, so you can save time.
