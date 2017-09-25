## Introduction

[TOC]

The Andersen Lab makes use of Quest, the supercomputer at Northwestern. Take some time to read over the overview of what Quest is, what it does, how to use it, and how to sign up:

__[Quest Documentation](http://www.it.northwestern.edu/research/user-services/quest/index.html)__

### Signing into Quest

After you gain access to the cluster you can login using:

```
ssh <netid>@quest.it.northwestern.edu
```

I recommend setting an alias in your `.bash_profile` to make logging in quicker:

```
alias quest="ssh <netid>@quest.it.northwestern.edu"
```

The above line makes it so you simply type `quest` and the login process is initiated. 

If you are not familiar with what a bash profile is, [take a look at this](https://www.quora.com/What-is-bash_profile-and-what-is-its-use).

When you login it is important to be conscientious of the fact that you are on a login node. You should not be running any sort of heavy duty operation on these nodes. Instead, any analysis you perform should be submitted as a job. 

### Home Directory

Logging in places you in your home directory. You can install software in your home directory for use when you run jobs, or for small files/analysis.

Your home directory has a quota of 80 Gb. [More information on quotas, storage, etc.](http://www.it.northwestern.edu/research/user-services/quest/file-systems.html).

More information is provided below to help install and use software.

### Projects

Quest is broadly organized into projects. Projects have associated with them storage, nodes, and users.

The Andersen lab has access to two projects.

__b1042__ - The 'Genomics' Project has 155 Tb of space and 100 nodes associated with it. This space is shared with other labs and is designed for temporary use only (covered in greater detail in the Nextflow Section). The space is available at `/projects/b1042/AndersenLab/`. By default, files are deleted after 30 days.

__b1059__ - The Andersen Lab Project. __b1059__ does not have any nodes associated with it, but it does have 10 Tb of storage. b1059 storage is located at: `/projects/b1059/`.

## Installing and using software

### Using `module`

Quest comes with a command called `module` that allows you to load software for use. The following commands can be used to see/load software.

* `module avail` - List available software.
* `module load software/version` - Load a software package (e.g. `module load R/3.3.1`).

### Using linuxbrew

Linuxbrew is a fork of Homebrew - which is a package manager for MacOSX. Linuxbrew should be installed in your home directory using the following command:

A lot of the software you use can be installed using a `bottle` format. 


The Andersen Lab has access to the 'Genomics' project

### Starting interactive jobs

```
msub -I -A b1042
```

#### Using Screen

<kbd>Ctrl</kbd> + <kbd>A</kbd>
