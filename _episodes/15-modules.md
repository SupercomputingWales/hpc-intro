---
title: "Accessing software"
teaching: 30
exercises: 15
questions:
- "How do we load and unload software packages?"
objectives:
- "Understand how to load and use a software package."
keypoints:
- "Discover available software with `module avail`"
- "Load software with `module load softwareName`"
- "Unload software with `module purge`"
- "The module system handles software versioning and package conflicts for you automatically."
---

On a high-performance computing system, it is often the case that no software is loaded by default. If we want to use a
software package, we will need to "load" it ourselves.

Before we start using individual software packages, however, we should understand the reasoning
behind this approach. The three biggest factors are:

- software incompatibilities;
- versioning;
- dependencies.

Software incompatibility is a major headache for programmers. Sometimes the presence (or absence) of
a software package will break others that depend on it. Two of the most famous examples are Python 2
and 3 and C compiler versions. Python 3 famously provides a `python` command that conflicts with
that provided by Python 2. Software compiled against a newer version of the C libraries and then
used when they are not present will result in a nasty `'GLIBCXX_3.4.20' not found` error, for
instance.

Software versioning is another common issue. A team might depend on a certain package version for
their research project - if the software version was to change (for instance, if a package was
updated), it might affect their results. Having access to multiple software versions allow a set of
researchers to prevent software versioning issues from affecting their results.

Dependencies are where a particular software package (or even a particular version)
depends on having access to another software package (or even a particular version of
another software package). For example, the VASP materials science software may 
depend on having a particular version of the FFTW (Fastest Fourier Transform in the West)
software library available for it to work.

## Environment modules

Environment modules are the solution to these problems.
A *module* is a self-contained description of a software package - 
it contains the settings required to run a software package 
and, usually, encodes required dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one will
allow you to use whichever is installed on the system you are using. In both 
implementations the `module` command is used to interact with environment modules. An
additional subcommand is usually added to the command to specify what you want to do. For a list
of subcommands you can use `module -h` or `module help`. As for all commands, you can 
access the full help on the *man* pages with `man module`.

On login you may start out with a default set of modules loaded or you may start out
with an empty environment, this depends on the setup of the system you are using.

### Listing currently loaded modules

You can use the `module list` command to see which modules you currently have loaded
in your environment. If you have no modules loaded, you will see a message telling you
so, the ```null``` module is a special module that is automatically loaded as a placeholder if automatic loading is
performed.

```
{{ site.host_prompt }} module list
```
{: .bash}
```
Currently Loaded Modulefiles:
  1) null
```
{: .output}

### Listing available modules

To see available software modules, use `module avail`

```
{{ site.host_prompt }} module avail
```
{: .bash}
```
{% include /snippets/15/module_avail.snip %}
```
{: .output}

There is also the possibility of loading inherited modules from old systems such as Hawk but this is not
recommended and is only used to aide migration to Falcon.  Loading ```hawk``` modules will
change the available modules.

## Loading and unloading software

To load a software module, use `module load`.

In this example we will use Python 3. Initially, Falcon provides a default Python 3:
```
{{ site.host_prompt }} python --version
Python 3.9.21
```

We can also look for the location where a particular program lives with the `which` command.
`which` looks for programs the same way that Bash does, so we can use it to tell us where a particular piece of software is stored.

```
{{ site.host_prompt }} which python3
```
{: .bash}

```
/usr/bin/python
```
{: .output}

We can load the `python3` command with `module load`:

```
{% include /snippets/15/load_python.snip %}
```
{: .bash}
```
{% include /snippets/15/which_python.snip %}
```
{: .output}

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH` environment
variable. `$PATH` is a special environment variable that controls where a UNIX system looks for
software. Specifically `$PATH` is a list of directories (separated by `:`) that the OS searches
through for a command before giving up and telling us it can't find it. As with all environment
variables we can print it out using `echo`.

```
{{ site.host_prompt }} echo $PATH
```
{: .bash}
```
{% include /snippets/15/path.snip %}
```
{: .output}

You'll notice a similarity to the output of the `which` command. In this case, there's only one
difference: the different directory at the beginning. When we ran the `module load` command,
it added a directory to the beginning of our `$PATH`. Let's examine what's there:

```
{% include /snippets/15/ls_dir.snip %}
```
{: .bash}
```
{% include /snippets/15/ls_dir_output.snip %}
```
{: .output}

Taking this to it's conclusion, `module load` will add software to your `$PATH`. It "loads"
software. A special note on this - depending on which version of the `module` program that is
installed at your site, `module load` will also load required software dependencies.

{% include /snippets/15/depend_demo.snip %}

## Software versioning

So far, we've learned how to load and unload software packages. This is very useful. However, we
have not yet addressed the issue of software versioning. At some point or other, you will run into
issues where only one particular version of some software will be suitable. Perhaps a key bugfix
only happened in a certain version, or version X broke compatibility with a file format you use. In
either of these example cases, it helps to be very specific about what software is loaded.

Let's examine the output of `module avail` more closely.

```
{{ site.host_prompt }} module avail
```
{: .bash}
```
....
------------------- /shared/apps/easybuild/x86_64/amd/zen4/modules/toolchain -------------------
   foss/2021b    foss/2023a    foss/2023b    foss/2024a    foss/2025a    foss/2025b (D)
....
```
{: .output}

{% include /snippets/15/gcc_example.snip %}

> ## Using software modules in scripts
>
> Create a job that is able to run `python3 --version`. Remember, no software is loaded by default!
> Running a job is just like logging on to the system (you should not assume a module loaded on the
> login node is loaded on a compute node).  Running `module purge` as first command in the job is recommended.
{: .challenge}

> ## Loading a module by default
> 
> Adding a set of `module load` commands to all of your scripts and having to manually load modules
> every time you log on can be tiresome. Fortunately, there is a way of specifying a set of 
> "default  modules" that always get loaded, regardless of whether or not you're logged on or 
> running a job. Every user has two hidden files in their home directory: `.bashrc` and 
> `.bash_profile` (you can see these files with `ls -la ~`). These scripts are run every time you 
> log on or run a job.  To aide with changes to the system, a `.myenv` file is sourced by `.bashrc`.  Adding a `module load` command to `.myenv` means that 
> that module will always be loaded. Modify either your `.myenv` scripts to 
> load a commonly used module like Python. Does your `python3 --version` job from before still 
> need `module load` to run?
{: .challenge}

## Installing software of our own

Most HPC clusters have a pretty large set of preinstalled software. Nonetheless, it's unlikely that
all of the software we'll need will be available. Sooner or later, we'll need to install some
software of our own.

Though software installation differs from package to package, the general process is the same:
download the software, read the installation instructions (important!), install dependencies,
compile, then start using our software.

As an example we will install the bioinformatics toolkit `seqtk`. We'll first need to obtain the
source code from GitHub using `git`.

```
{{ site.host_prompt }} git clone https://github.com/lh3/seqtk.git
```
{: .bash}
```
Cloning into 'seqtk'...
remote: Counting objects: 316, done.
remote: Total 316 (delta 0), reused 0 (delta 0), pack-reused 316
Receiving objects: 100% (316/316), 141.52 KiB | 0 bytes/s, done.
Resolving deltas: 100% (181/181), done.
```
{: .output}

Now, using the instructions in the README.md file, all we need to do to complete the install is to
`cd` into the seqtk folder and run the command `make`.

```
{{ site.host_prompt }} cd seqtk
{{ site.host_prompt }} make
```
{: .bash}
```
gcc -g -Wall -O2 -Wno-unused-function seqtk.c -o seqtk -lz -lm
seqtk.c: In function ‘stk_comp’:
seqtk.c:399:16: warning: variable ‘lc’ set but not used [-Wunused-but-set-variable]
    int la, lb, lc, na, nb, nc, cnt[11];
                ^
```
{: .output}

It's done! Now all we need to do to use the program is invoke it like any other program.

```
{{ site.host_prompt }} ./seqtk
```
{: .bash}
```
Usage:   seqtk <command> <arguments>
Version: 1.2-r101-dirty

Command: seq       common transformation of FASTA/Q
         comp      get the nucleotide composition of FASTA/Q
         sample    subsample sequences
         subseq    extract subsequences from FASTA/Q
         fqchk     fastq QC (base/quality summary)
         mergepe   interleave two PE FASTA/Q files
         trimfq    trim FASTQ using the Phred algorithm

         hety      regional heterozygosity
         gc        identify high- or low-GC regions
         mutfa     point mutate FASTA at specified positions
         mergefa   merge two FASTA/Q files
         famask    apply a X-coded FASTA to a source FASTA
         dropse    drop unpaired from interleaved PE FASTA/Q
         rename    rename sequence names
         randbase  choose a random base from hets
         cutN      cut sequence at long N
         listhet   extract the position of each het
```
{: .output}

We've successfully installed our first piece of software!

{% include links.md %}
