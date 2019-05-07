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

!!! important
    When you login it is important to be conscientious of the fact that you are on a login node. You should not be running any sort of heavy duty operation on these nodes. Instead, any analysis you perform should be submitted as a job or run using an interactive job (see below).

### Login Nodes

There are four login nodes we use: quser10-13. When you login you will be assigned to a random login node. You can switch login nodes by typing ssh and the node desired (_e.g._ `ssh quser11`).

!!! warning
    When using [screen](https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/) to submit and run jobs they will only persist on the login node you are currently on. If you log out and later log back in you may be logged in to a different login node. You will need to switch to that login node to access those screen sessions.

### Home Directory

Logging in places you in your home directory. You can install software in your home directory for use when you run jobs, or for small files/analysis.

Your home directory has a quota of 80 Gb. [More information on quotas, storage, etc](http://www.it.northwestern.edu/research/user-services/quest/file-systems.html).

More information is provided below to help install and use software.

### Projects

Quest is broadly organized into projects. Projects have associated with them storage, nodes, and users.

The Andersen lab has access to two projects.

__b1042__ - The 'Genomics' Project has 155 Tb of space and 100 nodes associated with it. This space is shared with other labs and is designed for temporary use only (covered in greater detail in the Nextflow Section). The space is available at `/projects/b1042/AndersenLab/`. By default, files are deleted after 30 days.

__b1059__ - The Andersen Lab Project. __b1059__ does not have any nodes associated with it, but it does have 10 Tb of storage. b1059 storage is located at: `/projects/b1059/`.

### Running interactive jobs on Quest

If you are running a few simple commands or want to experiment with files directly you can start an interactive session on Quest. The command below will give you access to a node where you can run your commands

```
srun -A b1042 --partition=genomicsguest -N 1 -n 24 --mem=64G --time=12:00:00 --pty bash -i 
```

!!! Important
    Do not run commands on `quser21-24`. These are login nodes and are not meant for running heavy-load workflows.

