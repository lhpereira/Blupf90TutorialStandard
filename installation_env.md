---
title: Download and Installation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Additional installation
=======================

You may see an error message when running a BLUPF90 program. This is usually a system error, not a software bug.

When this occurs, you should install additional software libraries provided by Intel. The BLUPF90 family is compiled with the Intel Fortran Compiler and depends on software libraries such as the Intel oneAPI Math Kernel Library (often called MKL) and the Intel Fortran Compiler runtime library. These library packages are freely available at no charge.

You may need one or both of the packages shown below. Download the packages suitable for your operating system (Windows or Linux).

- [Intel Fortran Compiler Runtime](https://www.intel.com/content/www/us/en/developer/articles/tool/compilers-redistributable-libraries-by-version.html)
- [Intel oneAPI Math Kernel Library](https://www.intel.com/content/www/us/en/developer/tools/oneapi/onemkl-download.html)

Note that Intel often changes its page addresses, so an Internet search is the best way to find the current pages. To find the library packages, simply search for "MKL" or "Fortran runtime library." For details, please check the [official DokuWiki page](https://nce.ads.uga.edu/wiki/doku.php?id=distribution) at UGA.


Additional settings
===================

The BLUPF90 programs may use a large amount of memory, especially when the data set is large and a genomic model is applied. Heavy memory consumption often leads to a program crash with a *segmentation fault* (or *bus error*). A segmentation fault occurs when a program attempts to access memory in an inappropriate way or location. Note that this error can happen even if your computer has a lot of free memory. This is because the operating system (Linux, macOS, or Windows) imposes limitations on the memory usage of each program. More specifically, the system limits the amount of memory allocated to the *stack*, and our programs use a large stack to achieve maximum performance.

This is a common issue with large data sets. You may encounter it when updating to a newer version of the program while using the same input files. Even a small code change can affect memory usage.

You can either remove the limitation or increase the stack size. The following settings are recommended even if you haven't encountered this problem yet. More details are available on our website: <http://nce.ads.uga.edu/wiki/doku.php?id=faq.segfault>.

Increase stack size
-------------------

This setting is required on Linux and macOS. First, type the following command in your shell (terminal):

~~~~~{language=shell}
ulimit -s
~~~~~

If it shows `unlimited`, the configuration is fine. If you see a number (like `8192`), it is likely to be a problem. This number represents the stack size and it should be *unlimited*. To change the stack size, type the following command before running our programs:

~~~~~{language=shell}
ulimit -s unlimited
~~~~~

You can also add this command at the end of your `.bash_profile` in your home directory. This way, the setting will be applied automatically every time you log in.

Increase stack size for OpenMP
------------------------------

There is a separate setting for defining the stack size for OpenMP. This value is independent of the system stack size explained above. You can set it using an environment variable.

### Linux and macOS

To check the current value, type the following command in your shell:

~~~~~{language=shell}
echo $OMP_STACKSIZE
~~~~~

If it shows nothing or a small value such as `4M`, it might be too small. The default value is usually `4M`, which is likely insufficient. Before running the program, set it using the following command (with no spaces around `=`):

~~~~~{language=shell}
export OMP_STACKSIZE=64M
~~~~~

This sets the stack size to 64 megabytes. If the program still fails with the same error, increase the size gradually (e.g., to 128M, 192M, etc.). However, a large value can lead to high memory usage because each thread can allocate this much stack. There is no universally appropriate value—it depends on your system, so adjust it empirically.

For a one-time setting, you can add the variable inline when you run the program (no `export`):

~~~~~{language=shell}
OMP_STACKSIZE=64M ./blupf90
~~~~~

As with `ulimit`, you can save this setting permanently in your `.bash_profile`.

### Windows

On Windows, you can set the `OMP_STACKSIZE` environment variable either temporarily through the Command Prompt or permanently through the system settings in Control Panel.

To check the current value, type:

~~~~~{language=shell}
set %OMP_STACKSIZE%
~~~~~

To set a value (e.g., 64 megabytes), type:

~~~~~{language=shell}
set OMP_STACKSIZE=64M
~~~~~

This setting lasts only for the current session, so you must run the BLUPF90 programs in the same session after setting it. To make it permanent, use the Control Panel. For instructions, search for terms such as `Windows environment variable` using a search engine.

The value should include a number and unit, such as `64M` for 64 megabytes. Start with `64M`, and if the problem persists, increase the value (e.g., `128M` or more). As with other systems, a value that is too large will result in high memory consumption because each thread may use that much memory. You will need to determine an appropriate value based on your computer's resources.


Useful command-line features
============================

Most BLUPF90 programs support command-line options. Some programs accept only a few command-line options, whereas others are controlled almost entirely through them. Several command-line options are common across many programs, although not all programs support them.

A typical command-line argument is the name of a parameter file (or instruction file) describing the operational details. For example, the following command lets BLUPF90+ read the parameter file `param.txt`.

~~~~~{language=shell}
blupf90+ param.txt
~~~~~

Major programs accept `--help` to show a list of possible command-line options.

~~~~~{language=shell}
blupf90+ --help
~~~~~

Some programs accept the option to show a minimal template of the parameter file for analysis (or the instruction file for the renumbering operation).

~~~~~{language=shell}
blupf90+ --show-template
~~~~~
