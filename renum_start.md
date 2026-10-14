---
title: Data preparation with RENUMF90
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

This chapter provides an instruction on the RENUMF90 program (`renumf90`) to prepare the files for data, pedigree, genomic markers, and parameters. 
The BLUPF90 program requires several files with a particular format which the "raw" data may not have. The RENUMF90 program can check the raw files and convert them to new files suitable for the BLUPF90 suite. We will start with a minimal example as before.

More examples are available at the authors's Github repository [https://github.com/masuday/data](https://github.com/masuday/data). You will find example files with more complicated models including random regressions, maternal effects, a model with unknown parent groups (UPGs), and GBLUP models.
