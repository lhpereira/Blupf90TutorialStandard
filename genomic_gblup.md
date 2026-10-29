---
title: Practical genomic analysis
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Performing GBLUP
================

GBLUP with BLUPF90+
-------------------

### Background

BLUPF90+ can perform GBLUP, but you need a workaround because BLUPF90+ always performs single-step GBLUP and does not have a "GBLUP mode." In the single-step approach, the inverse of a relationship matrix consists of three matrices, as seen in the previous chapters.

$$
\mathbf{H}^{-1}
=
\mathbf{A}^{-1}
+
\left[
\begin{array}{cc}
\mathbf{0} & \mathbf{0}\\
\mathbf{0} & \mathbf{G}^{-1}-\mathbf{A}_{22}^{-1}
\end{array}
\right]
$$

By default, the BLUPF90 programs try to create all three matrices ($\mathbf{A}^{-1}$, $\mathbf{G}^{-1}$, and $\mathbf{A}_{22}^{-1}$) when the genomic relationship matrix is used. In other words, whenever the program creates $\mathbf{G}$, it requires a pedigree file even if the pedigree information is not actually used in the analysis. You have to make the program use a dummy pedigree file that nullifies the pedigree relationships in $\mathbf{H}^{-1}$, or prevent it from creating the pedigree relationship matrices altogether.

There are several methods to perform GBLUP using BLUPF90+ (and GIBBSF90+). The user can choose one of them.

- **Use of a dummy pedigree file.** RENUMF90 prepares such a dummy file. This is the simplest method and is always the first choice.
- **Use of a user-supplied text file for $\mathbf{G}$.** This is useful only if you want to use customized genomic relationships and can prepare the matrix yourself as a file.
- **Use of a special option in the parameter file.** This skips $\mathbf{A}^{-1}$ in combination with a zero weight on $\mathbf{A}_{22}^{-1}$. It is useful only if you need to adjust $\mathbf{G}$ using pedigree relationships but do not need pedigree relationships in the final equations.

We will explain all of them in the subsequent sections. The last two methods were explained in previous chapters.

### Model

Before introducing how to apply GBLUP in BLUPF90+, we clarify the model used in GBLUP. We assume that the model has some fixed effects ($\mathbf{b}$) and an additive genomic effect ($\mathbf{u}$). Each animal is assumed to have only one phenotype. The matrix notation of the model is as follows.

$$
\mathbf{y} = \mathbf{Xb} + \mathbf{u} + \mathbf{e}
$$

The phenotype can be a pseudo-observation such as a daughter yield deviation (DYD), progeny trait deviation (PTD), deregressed proof (DRP), and so on. When using a pseudo-phenotype, each observation has a different accuracy, so we should consider a weight that reflects the accuracy of the observation. Furthermore, as in the standard animal model, we need to know the variance components ($\sigma_{u}^{2}$ and $\sigma_{e}^{2}$ in this case) or the variance ratio ($\lambda=\sigma_{e}^{2}/\sigma_{u}^{2}$). The system of mixed-model equations is easily derived.

$$
\left[
\begin{array}{ll}
\mathbf{X}'\mathbf{W}\mathbf{X}\sigma_{e}^{-2}& \mathbf{X}'\mathbf{W}\sigma_{e}^{-2}\\
\mathbf{W}\mathbf{X}\sigma_{e}^{-2} & \mathbf{W}\sigma_{e}^{-2} + \mathbf{G}^{-1}\sigma_{u}^{-2}
\end{array}
\right]
\left[
\begin{array}{l}
\mathbf{\hat{b}} \\
\mathbf{\hat{u}}
\end{array}
\right]
=
\left[
\begin{array}{l}
\mathbf{X}'\mathbf{W}\mathbf{y}\sigma_{e}^{-2}\\
\mathbf{W}\mathbf{y}\sigma_{e}^{-2}
\end{array}
\right]
$$

The diagonal matrix, $\mathbf{W}$, contains one weight for each observation. In the following section, we assume $\mathbf{X}=\mathbf{1}$, so the general mean is the only fixed effect.


The simplest way with dummy pedigree
------------------------------------

### Idea

The simplest approach is to prepare the data using RENUMF90 with a *dummy* pedigree file. RENUMF90 creates such a pedigree file containing only genotyped animals whose parents are missing (unknown). With this pedigree file, the pedigree relationship matrix is equal to $\mathbf{I}$.

If all animals are genotyped, all three matrices in $\mathbf{H}^{-1}$ ($\mathbf{A}^{-1}$, $\mathbf{G}^{-1}$, and $\mathbf{A}_{22}^{-1}$) have the same size. Also, we have the equivalence: $\mathbf{A}^{-1}=\mathbf{A}_{22}^{-1}=\mathbf{I}$. The resulting $\mathbf{H}^{-1}$ has $\mathbf{G}^{-1}$ only because the pedigree matrices are canceled out.

$$
\mathbf{H}^{-1} = \mathbf{I} + \mathbf{G}^{-1} - \mathbf{I} = \mathbf{G}^{-1}
$$

The user should know that BLUPF90+ does not avoid creating the identity matrices $\mathbf{A}^{-1}=\mathbf{A}_{22}^{-1}=\mathbf{I}$. These matrices are explicitly calculated and added to the system of equations; in the end, the two $\mathbf{I}$ terms cancel *numerically*. This method provides the same solutions as omitting $\mathbf{A}^{-1}$ and $\mathbf{A}_{22}^{-1}$ from $\mathbf{H}^{-1}$ numerically, but those matrices are not actually omitted during the operation. This is because BLUPF90+ is designed to read the pedigree file and build $\mathbf{A}^{-1}$ and $\mathbf{A}_{22}^{-1}$ whenever $\mathbf{G}$ is created.

### Numerical example

We will use an example considered in an earlier chapter. There are 15 genotyped animals, but only the first 11 animals have phenotypes. The data file has four columns: (1) animal ID, (2) a group code for the general mean, (3) observation, and (4) observation weight. This file is the same as the previous one, but it contains the original IDs corresponding to the marker file. If you do not use weights, you can omit column 4 from the data file.

~~~~~{language=text caption=rawdata6.txt}
ID001  1  -1.28  1.00
ID002  1  -1.78  0.98
ID003  1   0.54  0.90
ID004  1   1.36  1.00
ID005  1   0.47  0.93
ID006  1   0.04  1.00
ID007  1   0.68  1.00
ID008  1  -0.89  0.85
ID009  1  -0.48  0.88
ID010  1   0.89  0.99
ID011  1  -1.16  0.97
~~~~~

The marker file should have all 15 animals.

~~~{language=widetext caption=snp6.txt}
ID002  1212020111001211212000102001202022201112000110020102011001012212110122100001210101112200210010022220
ID011  2211121200121111220211011210112121111201000120011202020002002222000222000021221000021200110010101220
ID005  1201121111012200221100012210201010110111000211011112120112101221110122100012211000022200110010012220
ID014  2122020101100221112111202101202022202222000000021111102000111201211022200101210000022200210010111120
ID010  2111121101111210121211112210202021211211000110021211111001101211101122100111221000022200120020101120
ID006  2122020101100221121111212100112111112212000101012211111001201211112022100201220110022200220020111120
ID004  2222020200110122210121001100022221102111000010011202020002002222000222000020120000020200200000200220
ID015  2122020101210121121222101200112122102212000010012211111001101211101122100110220000021200210010200120
ID012  2222020200110122210121001100022221102111000010011202020002002222000222000020120000020200200000200220
ID003  2122020210000221111111112010011100011112000202002112120112201221121022100111220110111200210010111220
ID007  1201121111012200221100012110202021210101000220020202020002002222000222000011221101112200120020012220
ID009  2222020200000222211000212001112111112212000101011102011001112212121022100102210110022200210010022220
ID001  1211121200011211220201121220102021210101000220020211021011002222000222000012211001112100120020012220
ID013  2122020101210121121222101200112122102212000010012211111001101211101122100110220000021200210010200120
ID008  1211121200011211210111111110012121201011000110020211021011002222000222000011110001111100210010111220
~~~

There is no pedigree file for these data.

In this data set, some animals do not have phenotypes, which is typical in GBLUP. RENUMF90 will create the dummy pedigree file for all genotyped animals in the marker file. The number of pedigree animals will be the same as the number of genotyped animals, and BLUPF90+ will perform GBLUP correctly.

### Renumbering

An instruction file for RENUMF90 is as follows. We assume $\sigma_{u}^{2}=0.3$ and $\sigma_{e}^2=0.7$. If you do not need the weight, you can put the empty line to `WEIGHT` (this keyword is mandatory so you should keep it even if you do not use the weights).

~~~{language=renumf90 caption=renum6.txt}
DATAFILE
rawdata6.txt
TRAITS
3
FIELDS_PASSED TO OUTPUT

WEIGHT(S)
4
RESIDUAL_VARIANCE
0.7
EFFECT           # 1st effect fixed
2 cross alpha
EFFECT           # 2nd effect (animal)
1 cross alpha
RANDOM           ## additive effect without pedigree
animal
SNP_FILE         ## SNP marker file
snp6.txt
(CO)VARIANCES    ## its variance component
0.3
OPTION AlphaBeta 0.95 0.05
OPTION tunedG 0
~~~

The last two lines are the options for BLUPF90+ (passed through the resulting parameter file `renf90.par`). The first option, `AlphaBeta`, is needed to make $\mathbf{G}$ be positive definite. It performs the blending $\mathbf{G}\leftarrow 0.95\mathbf{G}+0.05\mathbf{A}_{22}=0.95\mathbf{G}+0.05\mathbf{I}$. The second option `tunedG 0` turns off the _tuning_ to scale $\mathbf{G}$ to $\mathbf{A}_{22}$; see the previous section for details. In this analysis, $\mathbf{A}_{22}$ is just a dummy (the identity matrix) so there is no reason to perform the tuning. You can also put any options you need in the parameter file.

Note that there is no `FILE` keyword for pedigree, but the program generates the dummy pedigree-file. After running `RENUMF90`, you will see the pedigree file `renadd02.ped` in the same directory.

~~~~~
13 0 0 3 0 10 0 0 0 ID015
4 0 0 3 0 10 1 0 0 ID004
5 0 0 3 0 10 1 0 0 ID005
6 0 0 3 0 10 1 0 0 ID006
7 0 0 3 0 10 1 0 0 ID007
8 0 0 3 0 10 1 0 0 ID008
10 0 0 3 0 10 1 0 0 ID010
9 0 0 3 0 10 1 0 0 ID009
11 0 0 3 0 10 1 0 0 ID011
14 0 0 3 0 10 0 0 0 ID012
1 0 0 3 0 10 1 0 0 ID001
15 0 0 3 0 10 0 0 0 ID013
2 0 0 3 0 10 1 0 0 ID002
12 0 0 3 0 10 0 0 0 ID014
3 0 0 3 0 10 1 0 0 ID003
~~~~~

The renumbering process assigns an integer code to each genotyped animal. The first column (animal ID) contains the genotyped animals, while the second and third columns (parents) contain 0, indicating missing parents. This creates the identity matrix as the pedigree relationship matrix. The tenth column contains the original ID corresponding to the integer (renumbered) code.

The cross-reference file (XrefID) clearly shows the correspondence between the integer code and the original ID. The integer code is assigned to the original ID in an arbitrary order; the user should not expect any meaningful ordering.

~~~~~
2 ID002
11 ID011
5 ID005
12 ID014
10 ID010
6 ID006
4 ID004
13 ID015
14 ID012
3 ID003
7 ID007
9 ID009
1 ID001
15 ID013
8 ID008
~~~~~

### Running GBLUP

BLUPF90 calculates the solutions to the mixed-model equations.

~~~{language=text caption=solutions}
trait/effect level  solution
   1   1         1         -0.08090772
   1   2         1         -0.42220198
   1   2         2         -0.44672494
   1   2         3          0.26450865
   1   2         4          0.24686356
   1   2         5          0.05496292
   1   2         6          0.16937536
   1   2         7         -0.17601451
   1   2         8         -0.24385662
   1   2         9         -0.25750923
   1   2        10          0.13351286
   1   2        11          0.03869522
   1   2        12         -0.10858569
   1   2        13          0.26284875
   1   2        14          0.22127690
   1   2        15          0.26284875
~~~

The genomic-effect solutions are in the rows with effect 2 (rows with `2` in the second column). Each individual solution is labeled with the integer code, not the original ID, so the user must combine the solutions with the original IDs using the pedigree file or the XrefID file. If Bash is available, the following command immediately combines the solutions with the original IDs and sorts them by original ID.

~~~{language=shell}
paste <(sort -n snp6.txt_XrefID) <(awk '$2==2{print $3,$4}' solutions) | sort -k2,2
~~~

### Remarks

This method will work correctly unless nongenotyped animals have phenotypes. What happens when nongenotyped animals have observations? The nongenotyped animals will be included in the equations and will have solutions. However, these solutions are meaningless because those animals are not related to the other animals; the estimates are merely artifacts. The user is responsible for ensuring that the data are appropriate for GBLUP.


Genomic relationship matrix as an external text file
---------------------------------------------------

As explained in the previous chapter, the user can supply $\mathbf{G}^{-1}$ as a text file. Users should prepare the file themselves using external software such as R. In this case, BLUPF90 does not need the marker file.

The text file contains three columns: (1) row index, (2) column index, and (3) the value of the element in $\mathbf{G}^{-1}$. Only the lower or upper triangular part is needed; the program stops if the file contains the entire matrix. The data file should be renumbered. Only numerical values are allowed for the IDs of genotyped animals, and each ID should correspond to an index in the $\mathbf{G}^{-1}$ file.

### Numerical example

Here, we will use the same example as in the previous section with an external text file. There are 15 genotyped animals, but only the first 11 animals have phenotypes. The external file of $\mathbf{G}^{-1}$ (`ginverse6.txt`) is too large to show here, so only the first few rows are presented. The whole file is available at <https://github.com/masuday/data> on GitHub.

~~~~~
1 1 8.227580098846
1 2 3.438271310942
2 2 8.954010413251
1 3 1.176962440315
2 3 .432327333335
...
~~~~~

This file is obtained from the marker file shown above (`snp6.txt`) so the index of a matrix is the same as the order of animals in the marker file.

Index in $\mathbf{G}$ ID in marker file
--------------------- ---------------------
 1                     ID002
 2                     ID011
 3                     ID005
 4                     ID014
 5                     ID010
 6                     ID006
 7                     ID004
 8                     ID015
 9                     ID012
10                     ID003
11                     ID007
12                     ID009
13                     ID001
14                     ID013
15                     ID008

The data file has five columns: (1) animal ID, (2) a group code for the general mean, (3) observation, (4) observation weight, and (5) the original ID for reference. The fifth column contains characters, but this is not a problem because the program never reads it. The animal ID should be compatible with the external $\mathbf{G}^{-1}$ file.

~~~~~{language=text caption=data6.txt}
13  1  -1.28  1.00  ID001
 1  1  -1.78  0.98  ID002
10  1   0.54  0.90  ID003
 7  1   1.36  1.00  ID004
 3  1   0.47  0.93  ID005
 6  1   0.04  1.00  ID006
11  1   0.68  1.00  ID007
15  1  -0.89  0.85  ID008
12  1  -0.48  0.88  ID009
 5  1   0.89  0.99  ID010
 2  1  -1.16  0.97  ID011
~~~~~

The parameter file is as follows. It does not have any genomic options because the matrix has already been prepared and no additional operations are allowed for the supplied matrix.

~~~~~{language=blupf90 caption=param6.txt}
DATAFILE
 data6.txt
NUMBER_OF_TRAITS
 1
NUMBER_OF_EFFECTS
 2
OBSERVATION(S)
 3
WEIGHT(S)
 4
EFFECTS:
 2   1 cross
 1  15 cross
RANDOM_RESIDUAL VALUES
 0.7
RANDOM_GROUP
 2
RANDOM_TYPE
 user_file
FILE
 ginverse6.txt
(CO)VARIANCES
  0.3
~~~~~

Be careful when reading the results: the order in `solutions` is different from that in the previous example. The user must combine the solutions with the original IDs. The following command correctly sorts the solutions by original ID.

~~~{language=shell}
paste <(awk '{print $1}' snp6.txt) <(awk '$2==2{print $3,$4}' solutions) | sort
~~~

### Remarks

This method does not require RENUMF90. It is useful for simulated data with renumbered IDs. One challenge is computing $\mathbf{G}^{-1}$ with external programs. Also, reading the file may be slow when the matrix is too large.


GBLUP with partial use of pedigree
----------------------------------

There is a case in which a pedigree file is available but the user wants to perform GBLUP. The pedigree information can be used to form $\mathbf{G}$, but the final equations contain only $\mathbf{G}^{-1}$. Alternatively, the user can perform GBLUP with a (residual) polygenic effect represented by the additive relationship matrix. The method introduced here is general and flexible but somewhat complicated. We will use two programs, PREGSF90 for preparation and BLUPF90+ for prediction, according to the following protocol.

1. Run RENUMF90 to generate renumbered data and `renf90.par`.
2. Run PREGSF90 with `renf90.par`.
3. Make a copy of `renf90.par`, modify the copy, and run BLUPF90+ with the modified parameter file.


### Renumbering

In this method, we will use the same marker and data files (`snp6.txt` and `rawdata6.txt`) as the previous section. The pedigree file is shown below.

~~~{language=text caption=rawpedigree6.txt}
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
~~~

First, we have to process the files with RENUMF90. The following instruction file is used.

~~~{language=blupf90 caption=renum6a.txt}
DATAFILE
rawdata6.txt
TRAITS
3
FIELDS_PASSED TO OUTPUT

WEIGHT(S)
4
RESIDUAL_VARIANCE
0.7
EFFECT           # 1st effect fixed
2 cross alpha
EFFECT           # 2nd effect (animal)
1 cross alpha
RANDOM           ## additive effect without pedigree
animal
FILE             ## pedigree file
rawpedigree6.txt
SNP_FILE         ## SNP marker file
snp6.txt
(CO)VARIANCES    ## its variance component
0.3
OPTION TauOmega  1.00 0.00
~~~

The user can use any options for genomic setup. The option `TauOmega` is needed to remove $\mathbf{A}_{22}^{-1}$ from $\mathbf{H}^{-1}$. The user can add more options to suit the data.

Run RENUMF90, and several files including `renf90.par` will be created.

### Run PREGSF90

First, look at `renf90.par` to see what we are going to do with it.

~~~{language=blupf90 caption=renf90.par}
# BLUPF90 parameter file created by RENUMF90
DATAFILE
 renf90.dat
NUMBER_OF_TRAITS
           1
NUMBER_OF_EFFECTS
           2
OBSERVATION(S)
    1
WEIGHT(S)
           2
EFFECTS: POSITIONS_IN_DATAFILE NUMBER_OF_LEVELS TYPE_OF_EFFECT[EFFECT NESTED]
  3         1 cross
  4        15 cross
RANDOM_RESIDUAL VALUES
  0.70000
 RANDOM_GROUP
     2
 RANDOM_TYPE
 add_animal
 FILE
renadd02.ped
(CO)VARIANCES
  0.30000
OPTION SNP_file snp6.txt
OPTION TauOmega  1.00 0.00
~~~

PREGSF90 calculates the relationship matrices as follows:

1. Compute $\mathbf{A}_{22}$.
2. Compute $\mathbf{G}$.
3. Blend two matrices as $\mathbf{G}\leftarrow \alpha\mathbf{G}+\beta\mathbf{A}_{22}$.
4. Scale $\mathbf{G}$ to $\mathbf{A}_{22}$ (_tuning_).
5. Compute $\mathbf{A}_{22}^{-1}$.
6. Compute $\mathbf{G}^{-1}$.
7. Merge two inverse matrices to $\Delta=\tau\mathbf{G}^{-1}-\omega\mathbf{A}_{22}^{-1}$ and save it to a file `GimA22i`.

Note that PREGSF90 does not compute $\mathbf{A}^{-1}$ itself; BLUPF90 computes it from the pedigree file. Using the parameter file above, the default values are $\alpha=0.95$ and $\beta=0.05$, the pedigree matrix is integrated into $\mathbf{G}$, tuning is performed, and `GimA22i` contains only $\mathbf{G}^{-1}$ because $\omega=0$. Again, these configurations are intended only to demonstrate the capabilities of PREGSF90 and BLUPF90 for GBLUP. Use appropriate options for your data.

After running PREGSF90, several files will be created in the working directory. There should be a file `GimA22i`.

### Run BLUPF90

BLUPF90+ needs a separate parameter file that is slightly different from `renf90.par` used by PREGSF90. Make a copy of `renf90.par` named `blup.par`, and then modify the options in the copy as follows (only the options are shown).

~~~{language=blupf90 caption=blup.par}
OPTION SNP_file snp6.txt
OPTION readGimA22i
OPTION omit_ainv
~~~

The first option, `SNP_file`, should be retained in the file. The remaining two options are crucial: `OPTION readGimA22i` reads the file `GimA22i` as $\Delta$ instead of computing it from the data, and `OPTION omit_ainv` suppresses the creation of $\mathbf{A}^{-1}$. In the end, $\mathbf{H}^{-1}$ contains only $\mathbf{G}^{-1}$.

Run BLUPF90+ with `blup.par` to obtain the solution file. The solutions look similar to those from the previous analysis for some animals (especially younger animals) but are very different for other animals (especially older animals). This is because, in this study, $\mathbf{G}$ was blended with $\mathbf{A}_{22}$ and can capture the residual polygenic effect.

### Remarks

#### Dummy pedigree to remove A-inverse

The option `omit_ainv` is effective only in BLUPF90+. If this option does not work (or you are not sure whether the program supports it), use the alternative approach to remove $\mathbf{A}^{-1}$. First, create a _dummy_ pedigree file containing only 0s in all 3 (or 4) columns. The pedigree file should have the same number of lines as, or more lines than, the number of genotyped animals. The _awk_ program can easily create the dummy pedigree. The following command assumes that the renumbered pedigree is `renadd02.ped` and writes four columns in the case of unknown parent groups.

~~~{language=shell}
awk '{print 0,0,0,0}' renadd02.ped > dummy.ped
~~~

Then, modify `blup.par` to use `dummy.ped` instead of `renadd02.ped`. With this trick, $\mathbf{A}^{-1}$ becomes $\mathbf{0}$ and makes no contribution to the equations.

#### Pure GBLUP without pedigree effect

Even with the pedigree file, the user can run GBLUP without any contribution from pedigree information (that is, $\mathbf{A}_{22}$). In this case, the solutions should be identical to those obtained with the previous two methods.

To do this, the user can put the following options in the renum-instruction file or `renf90.par` for PREGSF90.

~~~~~
OPTION AlphaBeta 0.95 0.00
OPTION GammaDelta 0.05 0.00
OPTION tunedG 0
OPTION TauOmega 1.00 0.00
~~~~~

The first two options are used for blending: $\mathbf{G}\leftarrow \alpha\mathbf{G}+\beta\mathbf{A}_{22}+\gamma\mathbf{I}+\delta\mathbf{1}\mathbf{1}'$. The options above specify $\alpha=0.95$, $\beta=0$, $\gamma=0.05$, and $\delta=0$, so the blending is $\mathbf{G}\leftarrow 0.95\mathbf{G}+0.05\mathbf{I}$. The option `tunedG 0` prevents the program from scaling $\mathbf{G}$ to $\mathbf{A}_{22}$. These three options are equivalent to the first method introduced in this chapter. The last option removes $\mathbf{A}_{22}^{-1}$ from $\mathbf{H}^{-1}$.

Finally, the user should create a separate parameter file that uses `GimA22i` and omits $\mathbf{A}^{-1}$ in BLUPF90+, either with `omit_ainv` or with the dummy pedigree file.
