---
title: Introduction
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Short introduction to BLUPF90 programs
======================================

What is BLUPF90?
----------------

BLUPF90 is the name of the software. It is also the name of a collection of programs derived from BLUPF90. In the latter case, we will refer to it as the *BLUPF90 family*, *BLUPF90 programs*, or simply *BLUPF90* if obvious. A concise description can also be found on the official wiki at the University of Georgia (<http://nce.ads.uga.edu/wiki/doku.php>).

>    The BLUPF90 family of programs is a collection of software in Fortran 90/95 for mixed  
>    model computations in animal breeding. The goal of the software is to be as simple as  
>    a matrix package and as efficient as a programming language.

The BLUPF90 program creates and solves mixed model equations. It supports various models, including the animal model, maternal model, and random regression model with multiple traits. The BLUPF90 family includes several programs for variance component estimation using REML and Gibbs sampling with various models. The set of programs is available on the official website (<http://nce.ads.uga.edu>) and can be used freely for academic or research purposes.

Since 2022, the previous set of software has been consolidated into a few computer programs.
The original BLUPF90 program was discontinued as part of the unification into BLUPF90+.
However, in this tutorial, we use BLUPF90 as an umbrella term for these programs.


Why BLUPF90?
------------

BLUPF90 offers several advantages to users: simplicity, stability, and active development. For programmers, the internal structure is documented in a course note (<https://nce.ads.uga.edu/wiki/doku.php?id=courses>), and the source code is available, so a developer can modify the program to support new ideas.

We will now look at its advantages from a user's point of view.

### Simplicity ###

The program's operation is very simple. Every BLUPF90 program reads a parameter file, which describes the names of data and pedigree files, models, and (initial) variance components to be used in the analysis. The parameter file is a short text file containing a few pairs of keywords and values that describe the necessary information. It is concise but capable of specifying general models. Once you learn how to write a parameter file, you can perform very complicated analyses with the program. When repeating statistical analyses using the same (or very similar) settings, you can automatically generate the parameter file in your script and run the program repeatedly.

Each program saves the solutions of the mixed model equations (e.g., EBV) to a file. Estimated variance components are also saved to files permanently.

### Stability ###

The programs have been tested by many researchers since their public release around 2000. They are now stable enough to be used for routine genetic evaluation at the national level. The team at the University of Georgia makes extensive use of the programs in its research.

### Active development ###

The programs are continuously maintained by the UGA team. New features are added to incorporate new methodologies, improve usability, and reduce computation time. The programs fully support single-step genomic BLUP (ssGBLUP); GWAS can be performed within the ssGBLUP framework.

### Speed ###

We have been optimizing the programs for speed, not only by using multithreaded libraries (MKL) but also through detailed optimization of fundamental subroutines. The current version is much faster than older versions, especially for REML and Gibbs sampling.

Is it easy?
-----------

Yes, it is, but the learning process is not always easy. There may be several hurdles to learning how to use the BLUPF90 programs in actual research. This is the flip side of their simplicity.

* Limited documentation. The official website hosts a manual for the programs (<http://nce.ads.uga.edu/wiki/doku.php>). The official wiki also provides various information about the software. However, documentation, especially for beginners, is incomplete in the existing resources.
* Command-line tool. BLUPF90 does not have a graphical user interface. BLUPF90 also uses simple text files for data input and output. If you use R or Python, using BLUPF90 requires some effort: saving your data to several files, preparing instruction files, calling BLUPF90 using a system function, and reading the results back from the output files. It is not convenient, but the developers focus on new features while keeping the user interface simple.
* Data manipulation. This is not a specific disadvantage of BLUPF90 but is common to similar software. The BLUPF90 family focuses on mixed model analyses and does not provide a data manipulation framework like R or SAS. Since the programs use text files as input, some text processing may be needed. You should prepare data and pedigree files in a specific format and check for erroneous records before running the programs.
* Pre-processing. Every program accepts only numerical values to maintain simplicity in programming. If your data or pedigree file contains characters (letters or symbols), you must replace them with integer codes before analysis. One of the programs, RENUMF90, can perform this task. For field or commercial data, you will need to run RENUMF90 before using BLUPF90.

The purpose of this tutorial is to provide such documentation with many examples.


Which programs?
----------------

The BLUPF90 family consists of a set of computer programs. In this tutorial, we will mainly use the following programs, which are publicly available from the official website. We may mention other programs when needed.

- `blupf90+`: to solve the mixed-model equations (calculate BLUEs and BLUPs) and estimate variance components using EM/AI REML.
- `gibbsf90+`: to perform Gibbs sampling for variance-component estimation and location-parameter prediction.
- `renumf90`: to prepare data, pedigree, and genomic marker files.

If the reader is familiar with previous versions, such as `airemlf90` and `thrgibbs1f90`, please read a [separate document](./introduction_plus.md). This document explains how the old programs have been consolidated into a few programs.

In the following sections, we will refer to specific program names using uppercase letters, such as BLUPF90+, RENUMF90, and PREGSF90. When referring to a specific computer program to be run from the command line, we will use monospaced text, such as `blupf90+`, `renumf90`, and `preGSf90`. We will use general descriptions, such as "the BLUPF90 program" or simply "BLUPF90," to refer to the set of programs in the family.
