---
title: Practical genomic analysis
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Quality control of SNP markers
==============================

Basic quality control options
-----------------------------

The genomic module performs quality control of SNP markers. This process removes unqualified markers and genotyped animals from the original data. The genomic module supports a variety of options for marker quality control. The following options change the behavior of the genomic module. All major BLUPF90 programs accept these options.


### Save cleaned genotypes ###

The following option saves the cleaned marker information to files. By default, this option is off, and the cleaned genotypes are not saved.

~~~~~{language=blupf90}
OPTION saveCleanSNPs
~~~~~

This option generates four new files. We assume that `snpfile` is the input marker file.

- `snpfile_clean` = new SNP marker file
- `snpfile_clean_XrefID` = new cross-reference file
- `snpfile_SNPs_removed` = list of removed markers
- `snpfile_Animals_removed` = list of removed animals


### Turn off the quality control ###

By default, the program performs quality control. The following option skips the quality-control process completely. Some quality-control statistics may still be calculated.

~~~~~{language=blupf90}
OPTION no_quality_control
~~~~~

All markers and genotyped animals are retained, even if they have obvious errors or other issues. This option is especially useful when all markers have already undergone quality control.


Detailed quality control options
--------------------------------

The following table shows useful options for this purpose. Some options have additional arguments, but you can omit them. If you omit the arguments, the default values are used. Some details are omitted here; see the Default column in the following table. A detailed explanation of each option can be found in the official manual.



 |Option name                              | Default   |Possible arguments                                    |Description                                               |
 |------------------------------------------------|-----------|----------------------------------|----------------------------------|
 |`minfreq x`                              | Enable    |$0 \leq x \leq 1$ (default:x = 0.05)                  |Minimum allele frequency to retain the marker|
 |`callrate x`                             | Enable    |$0 \leq x \leq 1$ (default:x = 0.90)                  |Minimum call rate for markers|
 |`callrateAnim x`                         | Enable    |$0 \leq x \leq 1$ (default:x = 0.90)                  |Minimum call rate for animals|
 |`monomorphic x`                          | Enable    |0 or 1 (default:1)                                    |Remove monomorphic markers|
 |`hwe x`                                  | Skip      |any real values (default: x = 0.15)                   |Maximum deviation of heterozygote frequency from the expected frequency|
 |`high_correlation x y`                   | Skip      |$-1 < x$, $y < 1$ (default: x = 0.025 and y = 0.995)  |Prune highly correlated markers|
 |`verify_parentage x`                     | Enable    |0,1,2 or 3 (default:3)                                |Check the Mendelian coherence of parent and offspring|
 |`exclusion_threshold x`                  | Enable    |$0 < x < 100$ % (default:x = 1%)                      |Acceptance percentage of Mendelian coherence |
 |`exclusion_threshold_snp x`              | Enable    |$0 < x < 100$ % (default x = 20%)                     |Acceptance percentage of Mendelian coherence |
 |`number_parent_progeny_evaluation x`     | Enable    |integer value (default:100)                           |Minimum number of tests for Mendelian coherence|
 |`threshold_duplicate_samples x`          | Enable    |$-1 < x < 1$ (default: x = 0.9)                       |Check for excessively high genomic relationships (duplicate samples)|
 |`threshold_diagonal_g x`                 | Enable    |real value (default: x = 1.6)                         |Check for excessively high genomic inbreeding|
 |`thrWarnCorAG x`                         | Enable    |$-1 < x < 1$ (default:x = 0.50)                       |Warning: low resemblance $\mathbf{A}_{22}$ and $\mathbf{G}$ |
 |`thrStopCorAG x`                         | Enable    |$-1 < x < 1$ (default:x = 0.30)                       |Error: low resemblance $\mathbf{A}_{22}$ and $\mathbf{G}$ |
 |`thrCorAG x`                             | Enable    |$-1 < x < 1$ (default: x = 0.02)                      |Defines which values of $\mathbf{A}_{22}$ are considered for the above correlations|


Extra options for quality control
---------------------------------

The following options provide extra information from quality control.

 |Option name             |Default   |Possible arguments    |Description |
 |------------------------|----------|-------------------|-----------------------|
 |`chrinfo file`          |Skip      |characters            |Specify the chromosome map file|
 |`excludeCHR n1 n2` ...  |Skip      |integer values        |Specify the chromosome numbers to be excluded from quality control|
 |`sex_chr n`             |Skip      |integer value         |Specify the number for the sex chromosome in the marker file|
 |`plotpca`               |Skip      |none                  |Plot the first two principal components of $\mathbf{G}$|
 |`outcallrate`           |Skip      |none                  |Save the call rate information on SNP markers in the file|


Numerical example for quality control
-------------------------------------

### Files ###

We will examine quality control using a small example that includes unqualified markers and animals. The following files will be used in this section.

~~~~~{language=text caption="snpqc1_XrefID.txt"}
11 ID002
15 ID003
12 ID004
2 ID006
5 ID010
7 ID011
8 ID012
9 ID013
1 ID015
~~~~~

The original SNP file is also prepared.

~~~~~{language=widetext caption="snpqc1.txt"}
ID002 12120201110012112120001020012020222011120001100201020110010122121101221000012101
ID003 15120251111512112211105021012021521121110005110112111010051011111112225001100110
ID004 22220202001101222101210011000222211021110000100112020200020022220002220000201200
ID006 21220201011002211211112121001121111122120001010122111110012012111120221002012201
ID010 21111211011112101212111122102020212112110001100212111110011012111011221001112210
ID011 22111212001211112202110112101121211112010001200112020200020022220002220000212210
ID012 22220202001101222101210011000222211021110000100112020200020022220002220000201200
ID013 21220201012101211212221012001121221022120000100122111110011012111011221001102200
ID015 21220201012101211212221012001121221022120000100122111110011012111011221001102200
~~~~~

This marker file is the same as the one in the previous chapter, except that animal `ID003` has many missing genotypes. This animal is expected to be removed during quality control.

The data file is as follows.

~~~~~{language=text caption="dataqc1.txt"}
 3.0 1 1 1.0  2
 2.0 1 2 1.0  6
 4.0 1 1 2.0  8
 6.0 2 2 2.0  3
 3.0 2 1 1.0  5
 6.0 2 2 2.0  9
 6.0 3 1 2.0  4
 6.0 3 2 1.0  7
 8.0 3 1 1.0 10
 4.0 3 2 2.0  1
~~~~~

The pedigree file is as follows.

~~~~~{language=text caption="pedigreeqc1.txt"}
1 7 5 1 0 2 1 0 0 ID015
13 17 18 3 0 0 0 0 2 ID004
11 17 19 3 0 0 0 0 1 ID005
2 16 18 3 0 0 1 0 1 ID006
3 12 11 1 0 2 1 2 0 ID007
4 14 13 1 0 2 1 0 1 ID008
5 3 2 1 0 2 1 0 2 ID010
6 12 15 1 0 2 1 1 0 ID009
7 3 13 1 0 2 1 3 0 ID011
8 7 4 1 0 2 1 0 0 ID012
14 16 19 3 0 0 0 1 0 ID001
9 7 5 1 0 2 1 0 1 ID013
12 17 18 3 0 0 0 2 0 ID002
10 6 9 1 0 2 1 0 0 ID014
15 16 18 3 0 0 0 0 1 ID003
~~~~~

We will use the following parameter file for PREGSF90. The program will save the cleaned genotypes as well as call-rate information. We use loose criteria for several options because of the small data set. For example, we use a low call-rate threshold and relax the criterion for the correlation between $\mathbf{G}$ and $\mathbf{A}_{22}$. This is only a demonstration, so do not use such low values with real data sets.

~~~~~{language=blupf90 caption="paramqc1.txt"}
# BLUPF90 parameter file created by RENF90
DATAFILE
dataqc1.txt
NUMBER_OF_TRAITS
		   1
NUMBER_OF_EFFECTS
		   4
OBSERVATION(S)
	1
WEIGHT(S)

EFFECTS: POSITIONS_IN_DATAFILE NUMBER_OF_LEVELS TYPE_OF_EFFECT[EFFECT NESTED]
  2         3 cross
  3         2 cross
  4 1 cov
  5        15 cross
RANDOM_RESIDUAL VALUES
   2.0000
 RANDOM_GROUP
	 4
 RANDOM_TYPE
 add_animal
 FILE
pedigreeqc1.txt
(CO)VARIANCES
  0.50000
OPTION SNP_file snpqc1.txt snpqc1_XrefID.txt
OPTION callrate 0.80
OPTION thrStopCorAG 0.10
OPTION outcallrate
OPTION saveCleanSNPs
~~~~~

### Resulting files ###

You will obtain the following results.

- The cleaned marker file `snpqc1.txt_clean` is created. The corresponding cleaned cross-reference file is `snpqc1.txt_clean_XrefID`.
- The removed markers are listed in `snpqc1.txt_SNPs_removed`. In this example, 20 markers are removed.
- The removed animals are listed in `snpqc1.txt_Animals_removed`. The second animal (pedigree ID 15 = `ID003`) is removed because of its low call rate.
- The allele frequencies after quality control are saved in `freqdata.count.after.clean`. The frequency for a removed marker is shown as 0.0.
- The call-rate values in the files (`callrate` and `callrate_a`) are calculated from the original marker data.

You will also find the file `GimA22i`. What values does it contain? The following is a dump of this file.

~~~~~{language=output}
-0.335  0.000  -0.161 -0.421 -0.091  0.405  -0.329   0.265   0.265
 0.000  0.000   0.000  0.000  0.000  0.000   0.000   0.000   0.000
-0.161  0.000  10.624 -0.101  0.614  0.005 -11.035  -0.144  -0.144
-0.421  0.000  -0.101  0.212 -0.018  0.537  -0.154  -0.342  -0.342
-0.091  0.000   0.614 -0.018  0.773 -2.964   1.684   0.291   0.291
 0.405  0.000   0.005  0.537 -2.964  0.064  -0.796   1.421   1.421
-0.329  0.000 -11.035 -0.154  1.684 -0.796  11.503  -0.587  -0.587
 0.265  0.000  -0.144 -0.342  0.291  1.421  -0.587  20.186 -20.738
 0.265  0.000  -0.144 -0.342  0.291  1.421  -0.587 -20.738  20.186
~~~~~

You can see that this file contains a $9 \times 9$ matrix and that the second animal is not actually removed from $\mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}$. This matrix was calculated from the cleaned markers while retaining the rows and columns corresponding to removed animals. The program simply filled those rows and columns with 0. This is technically equivalent to using reconstructed relationship matrices calculated from the cleaned marker file.


When you invoke the genomic module from application programs (BLUPF90+, GIBBSF90+, etc.), the `GimA22i` file above is used directly in the analysis. When using PREGSF90, however, be careful with this file. It still corresponds to the original marker and cross-reference files, not to the cleaned files. For example, if an animal has been excluded by quality control, it is still present in `GimA22i`, with 0 in all related elements. This situation can be confusing. There is another issue: if you want the final matrix ($\mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}$) from the cleaned genotypes, it is wasteful to compute a matrix that retains unqualified objects.


### Better practice for quality control ###

We suggest that users follow the protocol below for marker quality control.

1. Perform only the quality control, and save the cleaned genotypes with PREGSF90.
2. Run PREGSF90 again to calculate and save the relationship matrices using the cleaned marker files.
3. Run an application program for your favorite analysis.

In short, we suggest creating the cleaned files as a separate step. This means that you need a different parameter file for each step (three parameter files in total). Despite the additional effort, the procedure above avoids confusion caused by the removal of markers and animals during quality control.

Another program, QCF90 (quality-control-F-90), supports non-renumbered files. This tutorial does not cover how to use QCF90. When QCF90 is used, the process is as follows:

1. Perform quality control and output the cleaned genotypes with QCF90.
2. Run PREGSF90 to calculate and save the relationship matrices using the cleaned marker files.
3. Run an application program for your favorite analysis.

If you want to create only the cleaned files—in other words, if you want to skip all subsequent steps—the following options are helpful.

~~~~~{language=blupf90}
OPTION createA22 0           # does not create A22
OPTION createA22Inverse 0    # does not compute A22-inverse
OPTION createG 0             # does not create G
OPTION createGInverse 0      # does not compute G-inverse
OPTION createGimA22i 0       # does not compute (G-inv - A22-inv)
~~~~~

With these five options, the correlation between $\mathbf{G}$ and $\mathbf{A}_{22}$ will not be calculated. You can calculate such statistics using the cleaned files.
