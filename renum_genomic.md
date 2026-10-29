---
title: Data preparation with RENUMF90
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Genomic model with SNP-marker file
==================================

Required files
--------------

The single-step GBLUP (ssGBLUP) model is an extension of the animal model. RENUMF90 can handle ssGBLUP using a parameter file similar to that for the animal model, with an additional option. Let us use the previous files. The following files are the same as those used previously, with different names.

~~~~~{language=text caption="rawdata4.txt"}
  ID006  A  1  1.0  3.0
  ID009  A  2  1.0  2.0
  ID012  A  1  2.0  4.0
  ID007  B  2  2.0  6.0
  ID010  B  1  1.0  3.0
  ID013  B  2  2.0  6.0
  ID008  C  1  2.0  6.0
  ID011  C  2  1.0  6.0
  ID014  C  1  1.0  8.0
  ID015  C  2  2.0  4.0
~~~~~

The pedigree file is as follows.

~~~~~{language=text caption="rawpedigree4.txt"}
 ID001      0      0
 ID002      0      0
 ID003      0      0
 ID004      0      0
 ID005      0      0
 ID006      0      0
 ID007  ID002  ID005
 ID008  ID001  ID004
 ID009  ID002  ID003
 ID010  ID007  ID006
 ID011  ID007  ID004
 ID012  ID011  ID008
 ID013  ID011  ID010
 ID014  ID009  ID013
 ID015  ID011  ID010
~~~~~

An SNP-marker file should be prepared as a fixed-width text file with 2 columns: the original ID in column 1 and genotypes in column 2. See the previous chapter for a detailed explanation of the marker-file format. We use the same file shown before.

~~~~~{language=widetext caption="snp4.txt"}
ID002 1212020111001211212000102001202022201112000110020102011001012212110122100001210101112200210010022220
ID004 2222020200110122210121001100022221102111000010011202020002002222000222000020120000020200200000200220
ID006 2122020101100221121111212100112111112212000101012211111001201211112022100201220110022200220020111120
ID010 2111121101111210121211112210202021211211000110021211111001101211101122100111221000022200120020101120
ID011 2211121200121111220211011210112121111201000120011202020002002222000222000021221000021200110010101220
ID012 2222020200110122210121001100022221102111000010011202020002002222000222000020120000020200200000200220
ID013 2122020101210121121222101200112122102212000010012211111001101211101122100110220000021200210010200120
ID015 2122020101210121121222101200112122102212000010012211111001101211101122100110220000021200210010200120
~~~~~

The marker file is neither altered nor duplicated by RENUMF90. The program only reads the 1st column (the original ID) and creates a table relating the renumbered animal ID to the original animal ID. The table is saved as a cross-reference file.

We assume the same animal model as before, except that genomic information is included. The following is a parameter file for handling the SNP file.

~~~~~{language=renumf90 caption="renum4.txt"}
DATAFILE
rawdata4.txt
TRAITS
5
FIELDS_PASSED TO OUTPUT

WEIGHT(S)

RESIDUAL_VARIANCE
2.0
EFFECT           # 1st effect fixed
2 cross alpha
EFFECT           # 2nd effect fixed
3 cross alpha
EFFECT           # 3rd effect fixed
4 cov
EFFECT           # 4th effect
1 cross alpha
RANDOM           ## treated as a random effect
animal
FILE             ## pedigree file
rawpedigree4.txt
FILE_POS         ## animal, sire and dam IDs with two 0s
1 2 3 0 0
SNP_FILE         ## SNP marker file
snp4.txt
(CO)VARIANCES    ## its variance component
0.5
~~~~~

A new keyword, `SNP_FILE`, followed by the name of the marker file, tells RENUMF90 how to process the marker file. This keyword should be placed just after `FILE_POS`.


Renumbered files
----------------

When running RENUMF90 with the above instruction, the program generates several files in the same folder (directory). In this example, RENUMF90 generates 5 files: `renf90.dat`, `renf90.par`, `renf90.tables`, `renadd04.ped`, and `snp4.txt_XrefID`. The last one is the cross-reference file (or simply the XrefID file), which contains 2 columns: the renumbered ID and the original ID, as follows.

~~~~~{language=text}
11 ID002
12 ID004
2 ID006
5 ID010
7 ID011
8 ID012
9 ID013
1 ID015
~~~~~

The order of genotyped animals is the same as that in the marker file. The name of this file is automatically determined and fixed as the original SNP filename plus `_XrefID`, for example, `snp.txt_XrefID` for a marker file named `snp.txt`. You cannot change the filename.

The renumbered parameter file `renf90.par` looks similar to the previous one, except for the option on the last line.

~~~~~{language=blupf90}
OPTION SNP_file snp4.txt
~~~~~

The parameter file does not refer to the cross-reference file because, by default, BLUPF90 programs read the standard XrefID file (SNP filename + `_XrefID`). Usually, you do not have to change the name of the cross-reference file, so you can keep this option line. If you rename the XrefID file, you must add its filename to the option line by hand (as shown in the quick tour).

The renumbered pedigree is different from the previous one.

~~~~~{language=text}
1 7 5 1 0 12 1 0 0 ID015
12 0 0 3 0 10 0 0 2 ID004
13 0 0 3 0 0 0 0 1 ID005
2 0 0 3 0 10 1 0 1 ID006
3 11 13 1 0 2 1 2 0 ID007
4 14 12 1 0 2 1 0 1 ID008
5 3 2 1 0 12 1 0 2 ID010
6 11 15 1 0 2 1 1 0 ID009
7 3 12 1 0 12 1 3 0 ID011
8 7 4 1 0 12 1 0 0 ID012
14 0 0 3 0 0 0 1 0 ID001
9 7 5 1 0 12 1 0 1 ID013
11 0 0 3 0 10 0 2 0 ID002
10 6 9 1 0 2 1 0 0 ID014
15 0 0 3 0 0 0 0 1 ID003
~~~~~

With genomics, RENUMF90 assigns new integer values to animals according to the following rules.

1. First, the program assigns the smallest numbers to animals with records. The order of the assigned numbers is random (that is, it does not follow the order found in the data file).
2. Second, the program assigns larger numbers to genotyped animals. The order is random (that is, it does not follow the order found in the marker file).
3. Finally, the program assigns larger numbers to animals found only in the pedigree. The order is random (that is, it is not sorted in any particular way).

You can find the genotyped animals in the renumbered pedigree file. When an animal is genotyped, the 6th column will be 10 or larger.

Practical remarks
-----------------

With genomic analysis, BLUPF90+ and similar programs (including PREGSF90) read and manipulate genomic data.
This process is complicated and controlled by many options, so you should provide some options in the parameter file.
You can put the genomic options in the instruction file to pass them to the BLUPF90 parameter file.
In a later section, we will introduce several options for genomic analyses.


Summary
-------

- RENUMF90 supports ssGBLUP.
- The instruction for ssGBLUP is the same as that used for the animal model, except for the additional `SNP_FILE` keyword and the name of the marker file.
- RENUMF90 does not change the SNP marker file. Instead, it creates a cross-reference file relating the original ID to the renumbered ID. The name of the cross-reference file ends with `_XrefID` by default.
- The suggested `renf90.par` contains an option line for the SNP file.
- The order of animals in the pedigree is determined by placing phenotyped animals first, genotyped animals second, and the other animals last.
