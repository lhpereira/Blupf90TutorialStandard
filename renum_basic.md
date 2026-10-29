---
title: Data preparation with RENUMF90
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Basic data preparation
======================

Basic usage of RENUMF90
-----------------------

### Why RENUMF90? ###

The BLUPF90 programs accept data and pedigree files containing only numerical values (integers or real values). A group label should be an integer starting from 1. A raw data file usually contains characters (letters or symbols) for animal IDs or group codes. These characters should be replaced with numerical values before analysis with the BLUPF90 programs. RENUMF90 performs this task. We call this process *renumbering*.

### Trivial instruction file ###

RENUMF90 reads a parameter file, but it is quite different from the one used in BLUPF90. Do not confuse the two kinds of parameter files. To avoid confusion, we will refer to the parameter file used in RENUMF90 as an *instruction file* in this tutorial. This name is unofficial but makes it easier to distinguish the two kinds of parameter files.

To understand the behavior of RENUMF90, let us try a small example without a pedigree. We use a raw data file and an instruction file. The raw data are in a space-separated text file containing five columns.

~~~~~{language=text caption="rawdata1.txt"}
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

We will create a parameter file for the following model.
$$
    y_{ijk} = A_{i} + S_{j} + \beta x_{ijk} + e_{ijk}
$$
This model is actually the same as the one introduced in the previous chapter as a fixed-effect model. The first column is the animal ID, the next two columns are fixed cross-classified effects, and the fourth column is a fixed regression covariate. Thus, this example produces a data set similar to the one used before.

RENUMF90 will read only the necessary columns in the raw data file, as described in the instruction file. The following instruction file demonstrates the renumbering of columns 2 and 3 as fixed cross-classified effects, column 4 as a covariate, and column 5 as a phenotype. In this case, column 1 is not used.

~~~~~{language=renumf90 caption="renum1.txt"}
DATAFILE
rawdata1.txt
TRAITS
5
FIELDS_PASSED TO OUTPUT

WEIGHT(S)

RESIDUAL_VARIANCE
1.0
EFFECT          # 1st effect
2 cross alpha
EFFECT          # 2nd effect
3 cross alpha
EFFECT          # 3rd effect
4 cov
~~~~~

An instruction file can also contain comments, which start with `#`, as shown in the example above.

The instruction file looks like a parameter file for BLUPF90+. It contains several pairs of keywords and values.
The file contains six keywords. `EFFECT` can be repeated several times. The following keywords are required in a minimal instruction file.

Keyword                          possible value    description
-----------------------          --------------    ------------------------------------------------
`DATAFILE`                       characters        The name of the raw data file to be processed.
`TRAITS`                         integer           A list of the position(s) of observations in the
                                                   data file.
`FIELDS_PASSED TO OUTPUT`        integer           A list of the position(s) of unchanged
                                                   columns in the data file. An empty value is
                                                   acceptable if not needed (just keep the line empty).
                                                   The columns will be passed through to the renumbered file.
`WEIGHT(S)`                      integer           The position(s) for weight in the data file.
                                                   An empty value is acceptable if not needed.
`RESIDUAL_VARIANCE`              real value(s)     Residual (co)variances.
`EFFECT`                         (see below)       Description of an effect. Repeatable.

The `EFFECT` keyword can be repeated as many times as needed. The instruction file above contains three effects. The `EFFECT` keyword has the following values.

Keyword       Position    effect type    data type
---------     --------    -----------    ---------
`EFFECT`      integer     `cross`        `alpha`
                                         `numer`
                          `cov`          (none)

In the table above, the position means the position(s) of the group code (for a class effect) or covariate (for a regression) in the raw data file. You can choose one of two effect types (`cross` for cross-classified or `cov` for regression). For cross-classified effects, you can also choose the data type: `alpha` if the column may contain letters, or `numer` if the column contains only integer values. The following explains the `EFFECT` blocks in the instruction file above.

~~~~~{language=renumf90}
EFFECT
  2 cross alpha       # treat 2nd column as cross-classified effect; assuming letters
EFFECT
  3 cross alpha       # treat 3rd column as cross-classified effect; assuming letters
EFFECT
  4 cov               # treat 4th column as covariate
~~~~~

Note that, in this case, you can technically use `numer` instead of `alpha` for the second effect because this column contains only integer values. Nevertheless, we recommend that users always use `alpha` because it can accept any type of data, including letters and numbers.

### Resulting files ###

RENUMF90 asks you to enter the name of the instruction file. The program reads the original data and replaces the group labels with sequential integer values. Finally, it generates three files, each with a fixed name. If a file with the same name already exists, it is replaced with a new one.

- `renf90.dat`: renumbered data file
- `renf90.par`: a suggested parameter file for BLUPF90
- `renf90.tables`: a correspondence table between the original code and a new integer value

Let us look at the contents of `renf90.dat`.

~~~~~{language=text}
 3.0 1 1 1.0
 2.0 1 2 1.0
 4.0 1 1 2.0
 6.0 2 2 2.0
 3.0 2 1 1.0
 6.0 2 2 2.0
 6.0 3 1 2.0
 6.0 3 2 1.0
 8.0 3 1 1.0
 4.0 3 2 2.0
~~~~~

You can see the difference between the original and renumbered files. The order of the lines (rows) is preserved, but the order of the columns is different. Each column contains either integer or real (numeric) values. RENUMF90 changes the column order according to the following rules.

  1. First, observations: RENUMF90 does not change the values.
      - column 1: a single-trait model is assumed, and the 1st column is for observations.
  2. Second, effects: the order is determined by the order of the `EFFECT` statements in the instruction file.
      - column 2 for effect 1 (corresponding to the 2nd column in the original file).
      - column 3 for effect 2 (3rd column in the original file).
      - column 4 for effect 3 (4th column in the original file).
  3. Last, passed columns: RENUMF90 adds the columns specified by the `FIELDS_PASSED TO OUTPUT` keyword in the instruction file. It does not change the content in any way, so the passed columns may contain letters and symbols.
      - no additional columns in this case: we did not set any values for this keyword.

Class variables for effects 1 and 2 (columns 2 and 3) are successfully replaced with integer values. For effect 1, A is replaced with `1`, B with `2`, and C with `3`. For effect 2, 1 is replaced with `1` and 2 with `2`. The correspondence is saved in `renf90.tables`.

~~~~~{language=text}
 Effect group 1 of column 1 with 3 levels, effect # 1
 Value    #    consecutive number
A 3 1
B 3 2
C 4 3
 Effect group 2 of column 1 with 2 levels, effect # 2
 Value    #    consecutive number
1 5 1
2 5 2
~~~~~

In this case, each `Effect group` has three columns: the first contains the original group code, the second contains the number of times the code occurred in the input, and the third contains the replacement integer value.

RENUMF90 kindly generates a parameter file for BLUPF90.

~~~~~{language=blupf90}
# BLUPF90 parameter file created by RENUMF90
DATAFILE
 renf90.dat
NUMBER_OF_TRAITS
		   1
NUMBER_OF_EFFECTS
		   3
OBSERVATION(S)
	1
WEIGHT(S)

EFFECTS: POSITIONS_IN_DATAFILE NUMBER_OF_LEVELS TYPE_OF_EFFECT[EFFECT NESTED]
  2         3 cross
  3         2 cross
  4 1 cov
RANDOM_RESIDUAL VALUES
   1.0000
~~~~~

This parameter file is ready to use with BLUPF90, and the program will successfully run the analysis. In this parameter file, the model description is inherited from the instruction file. For example, the instruction file above contains three effects, so `renf90.par` also contains three effects corresponding to the description in the instruction file. Note that the `EFFECTS:` line is followed by extra words (`POSITIONS_IN_DATAFILE`, etc.), but this is not a problem for BLUPF90.

Running BLUPF90 with this `renf90.par` will produce the same solutions as those described earlier. You can verify this using these files.

Optional features in RENUMF90
-----------------------------

RENUMF90 specifically supports some options specified by an additional line beginning with `OPTION` at the end of the instruction file. The following options are available.

- `OPTION alpha_size` $n$: Changes the size of character fields (the maximum number of characters in a column with an alpha specification) to $n$.
- `OPTION max_string_readline` $n$: Changes the record length (the maximum number of characters in a line) to $n$.
- `OPTION max_field_readline` $n$: Changes the maximum number of fields (columns) in a line to $n$.

The $n$ above is replaced with an actual integer value. See the examples.

~~~~~{language=renumf90}
OPTION max_string_readline 2048
~~~~~

If an option is not used by RENUMF90, the program simply passes it through to `renf90.par`. Thus, you can put `OPTION` lines that BLUPF90 will use in the instruction file and obtain a parameter file with the desired options for your analysis. See the following example.

~~~~~{language=renumf90 caption="renum1a.txt"}
DATAFILE
rawdata1.txt
TRAITS
5
FIELDS_PASSED TO OUTPUT

WEIGHT(S)

RESIDUAL_VARIANCE
1.0
EFFECT
2 cross alpha   # 1st effect
EFFECT
3 cross alpha   # 2nd effect
EFFECT
4 cov           # 3rd effect
OPTION max_string_readline 2048
OPTION alpha_size 40
OPTION solv_method FSPAK
~~~~~

The first two options will be used by RENUMF90, but the last one will be passed through to the output file `renf90.par`.


A template of an instruction file
---------------------------------

A template of the instruction file is available from various online sources, such as the official DocuWiki at UGA or the author's GitHub repository. If you are working on the command line, the best way to obtain the template is to type the following command in your shell.

~~~~~{language=shell}
renumf90 --show-template
~~~~~

This displays a minimal template for RENUMF90.


What is the best practice?
--------------------------

Suppose you have to perform many analyses with the same data but different models. How many times should you run RENUMF90? Basically, there are two approaches:

1. For every analysis, rewrite the instruction file and run RENUMF90. Then use the generated `renf90.par`.
2. First, prepare an instruction file with the most comprehensive model and run RENUMF90. Then, for each analysis, copy the generated `renf90.par` to a new file and modify it to fit the model.

The better approach depends on your situation. If you have a relatively small data set and the analysis will not take a long time, the first approach may be suitable. If your data set is large and the analysis will take a long time, the second approach is the only efficient way. Also, in some cases, the generated `renf90.par` is incomplete. The second approach is more general and is the use case assumed by the development team.


Summary
-------
- RENUMF90 reads a parameter file, which is different from the one used for BLUPF90. We call it the instruction file (or simply the instruction) in this chapter.
- RENUMF90 replaces alphabetic characters or symbols with integer values that BLUPF90 can use. This process is called *renumbering*.
- RENUMF90 generates a renumbered data file, a table file, and a template parameter file for BLUPF90.
- RENUMF90 generates a template parameter file whose parameters are inherited from the instruction file.
- An instruction file consists of pairs of keywords and values. The file can contain comments using the `#` syntax.
- RENUMF90 passes option lines in the instruction file to `renf90.par`.
- Command-line option, `renumf90 --show-template`, displays a minimal template of the instruction file.
- If you use the same data for different analyses, one possible approach is to run RENUMF90 once and modify the generated `renf90.par` to fit the model you want to use.
