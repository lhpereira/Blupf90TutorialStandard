---
title: Download and Installation
author: Yutaka Masuda
date: August 2026
subject: "Introduction to BLUPF90 suite programs"
tags: [introduction,tutorial]
...

Running a program in Linux and macOS
====================================

Command-line software
---------------------

BLUPF90 programs are command-line software. Each program reads a parameter file describing the names of the data and pedigree files, the model, and the covariance components used in the analysis. It displays log messages on the screen and writes one or more output files. These programs are designed to run in a shell environment with keyboard input.

Here, we assume a standard Bash environment. On macOS, all operations are performed in Terminal. No graphical user interface (GUI) is used.

**Note:** Currently, only Intel-based Macs are supported. Newer Macs with Apple M-series (ARM-based) CPUs are not supported.

Installation
------------

No special installation procedure is required. Simply download the program from the official website. Then, change its permissions using `chmod`, for example, `chmod 755` or `chmod u+x` (a helpful way to remember: _ch_ange the _mod_e to add execute permission _+x_ for the _u_ser). You can move the program to your preferred directory, which may already be listed in your `PATH` environment variable. If not, you can create such a directory and move the program there.

Here is an example:

~~~~~{language=shell}
# 1. Download the program. You can also use curl or another method.
# Note that, if it doesn't work, you should download it using a browser manually.
wget https://nce.ads.uga.edu/html/projects/programs/Linux/64bit/blupf90+

# 2. Change the permissions to make it executable.
chmod 755 blupf90+

# 3. Create a directory for binaries, if it doesn't exist.
mkdir ~/bin

# 4. Move the program to that directory.
mv blupf90+ ~/bin
~~~~~

You can use other tools, such as `curl`, instead of `wget` to download the file. If the directory (e.g., `~/bin`) is not listed in your `PATH` environment variable, you need to add it. Check the current `PATH` using:

~~~~~{language=shell}
echo $PATH
~~~~~

If your directory is listed, you are all set. If not, add it by typing:

~~~~~{language=shell}
export PATH=~/bin:$PATH
~~~~~

If the program is located in a different directory, replace `~/bin` with the correct path. This setting is temporary and will disappear when you log out. To make it permanent, add the command to your `.bash_profile`. If you are unsure how to do this, open `.bash_profile` with a text editor and add the line at the end. Log out and log back in for the change to take effect.

Running a program
-----------------

Navigate to the directory that contains all the required files (e.g., the pedigree, data, and parameter files). Then, type the program name to start it. If the program launches successfully, you will see a message like:

~~~~~{language=shell}
name of parameter file?
~~~~~

At this prompt, type the name of the parameter file. Some programs, especially those that perform Gibbs sampling, require additional inputs. The messages will appear on the screen (standard output), and you can save them to a file using redirection:

~~~~~{language=shell}
# Save standard output to a file.
# The program still accepts input even if nothing is shown.
blupf90+ > out.txt

# Save standard output to a file while also displaying it on screen.
blupf90+ | tee out.txt
~~~~~

You can also avoid entering the initial input manually:

~~~~~{language=shell}
# Provide the parameter file name via echo.
echo parameter.txt | blupf90+

# Alternatively, use an input file.
echo parameter.txt > input
blupf90+ < input
~~~~~

The second method is especially useful for Gibbs sampling programs that require several lines of input.

Stop a program
--------------

To stop the program immediately, press `Ctrl + C`. For convenience, you can hold down `Ctrl` and tap `C`. The program will terminate immediately, and any resulting messages can be safely ignored. In some cases, especially during multithreaded computations, the program may continue running after receiving the stop signal. If that happens, simply close the terminal window to stop the program completely.

Does the program not start?
--------------

When you try to run the program, it may stop immediately with the following or a similar message.

~~~~~{language=shell}
blupf90+: error while loading shared libraries: libmkl_intel_lp64.so.2: cannot open shared object file: No such file or directory
~~~~~

This means that you need additional software on which the BLUPF90 programs depend. Please check the next section, "Additional Settings," or the official webpage of the University of Georgia.
