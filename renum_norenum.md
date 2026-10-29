---
title: Data preparation with RENUMF90
author: Andres Legarra
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...


What if I do not want to use RENUMF90?
=====================================

Some users do not want to use RENUMF90. Common reasons for this are:

  - The data are simulated and already renumbered.
  - I do my own renumbering and want to use it.
  - I have a complex model and want better control over the assigned numbers.

In the following section, we provide instructions on how the files should be coded. We will *not* provide software, but you may take a look [here](https://github.com/alegarra/yarp).

Main requirements for recoding for BLUPF90 programs
---------------------------------------------

  - Levels of effects should be coded with consecutive numbers. The order may be "natural" or not. For instance, if the years in the data are (2010, 2011, 2013, 2014), they may be renumbered as (1, 2, 3, 4) or (2, 1, 3, 4).
  - The animal is another effect, so it must receive codes from 1 to the number of animals _in the pedigree file_ (this is usually greater than the number of animals in the data file).
  - The animal effect does not have to be renumbered in consecutive order (with parents before offspring), although that order is valid and may be used. For instance, the following renumbering is valid:

~~~{language=text caption="ped_kempthorne"}
  A 0 0
  B 0 0
  D A B
  E A D
  F B E
  Z A B
~~~

could be recoded as

~~~{language=text caption="ped_kempthorne_recoded"}
1 0 0 A 0 0
2 0 0 B 0 0
3 1 2 D A B
4 1 2 Z A B
5 1 3 E A D
6 2 5 F B E
~~~

(the last three columns are not needed but help the visualization).


Unknown parent groups
---------------------

However, if unknown parent groups (UPGs) are present in the pedigree, things get more complicated. Suppose that we have $n$ _real_ animals and $m$ UPGs. Animals have to be renumbered with codes from 1 to $n$, and UPGs have to be renumbered with codes from $n+1$ to $n+m$. In addition, because they do not have ancestors, the UPGs must not have their own lines in the pedigree file for BLUPF90. This way, BLUPF90 "knows" that a given number is a UPG and not an individual. For instance, in the previous example, assume that the UPGs are "unknown2000" and "unknown2004":

~~~{language=text caption="ped_kempthorne_unknown parent groups"}
  A unknown2000 unknown2000
  B unknown2000 unknown2004
  D A B
  E A D
  F B E
  Z A B
~~~

It should be recoded as follows.

~~~{language=text caption="ped_kempthorne_unknown parent groups_recoded"}
1 7 7 A unknown2000 unknown2000
2 7 8 B unknown2000 unknown2004
3 1 2 D A B
4 1 3 E A D
5 2 4 F B E
6 1 2 Z A B
~~~

Genotypes
---------

Genotypes should be coded as 0/1/2 for {AA, Aa, aa} and 5 for missing, as described elsewhere in this tutorial, in a fixed-width text file. The first column of this file contains possibly alphanumeric identifiers for animals, for instance:

~~~{language=text caption="genotype_file"}
   A 120120202102111
   Z 121111202111010
~~~

The important requirement is the creation of the cross-reference file (usually called `_XrefID`). This file contains the original ID and the new ID in the renumbered pedigree and data files. In the previous example, it would be:

 ~~~{language=text caption="genotype_file_XrefID"}
  1  A
  6  Z
 ~~~

Overall mean
------------

BLUPF90 programs do _not_ include an overall mean by default. If you need one, add it yourself as a column of 1s in the data file.
