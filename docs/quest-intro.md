## Introduction

[TOC]

The Andersen Lab makes use of Quest, the supercomputer at Northwestern. Take some time to read over the overview of what Quest is, what it does, how to use it, and how to sign up:

__[Quest Documentation](http://www.it.northwestern.edu/research/user-services/quest/index.html)__

### New Users

To gain access to Quest: 

Register new user with your NetID [here](https://app.smartsheet.com/b/form?EQBCT=9b3647a8cb2145929737ab4a0540cb46).
		
Apply to be added to partition b1059 [here](https://app.smartsheet.com/b/form/797775d810274db5889b5199c4260328).
Allocation manager: Erik Andersen, erik.andersen-at-northwestern.edu
		
Apply to be added to partition b1042 [here](https://app.smartsheet.com/b/form/797775d810274db5889b5199c4260328).
Allocation manager: Janna Nugent, janna.nugent-at-northwestern.edu

Quest has its Slack channel: `genomics-rcs.slack.com` and help email: `quest-help@northwestern.edu` for users to get help. 

### Signing into Quest

After you gain access to the cluster you can login using:

```
ssh <netid>@quest.it.northwestern.edu
```

To avoid typing in the password everytime, one can [set up a ssh key](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2).

I recommend setting an alias in your `.bash_profile` to make logging in quicker:

```
alias quest="ssh <netid>@quest.it.northwestern.edu"
```

The above line makes it so you simply type `quest` and the login process is initiated. 

If you are not familiar with what a bash profile is, [take a look at this](https://www.quora.com/What-is-bash_profile-and-what-is-its-use).

!!! important
    When you login it is important to be conscientious of the fact that you are on a login node. You should not be running any sort of heavy duty operation on these nodes. Instead, any analysis you perform should be submitted as a job or run using an interactive job (see below).

### Login Nodes

There are four login nodes we use: quser21-24. When you login you will be assigned to a random login node. You can switch login nodes by typing ssh and the node desired (_e.g._ `ssh quser21`).

!!! warning
    When using [screen](https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/) to submit and run jobs they will only persist on the login node you are currently on. If you log out and later log back in you may be logged in to a different login node. You will need to switch to that login node to access those screen sessions.

### Home Directory

Logging in places you in your home directory. You can install software in your home directory for use when you run jobs, or for small files/analysis.

Your home directory has a quota of 80 Gb. [More information on quotas, storage, etc](http://www.it.northwestern.edu/research/user-services/quest/file-systems.html).

More information is provided below to help install and use software.

### Projects and Queues

Quest is broadly organized into projects (or partitions). Projects have associated with them storage, nodes, and users.

The Andersen lab has access to two projects.

__b1042__ - The 'Genomics' Project has 200 Tb of space and 100 nodes associated with it. This space is shared with other labs and is designed for temporary use only (covered in greater detail in the Nextflow Section). The space is available at `/projects/b1042/AndersenLab/`. By default, files are deleted after 30 days. One can submit jobs to `--partition=genomicsguestA` to use this partition, with a max job duration of 48hr. 

__b1059__ - The Andersen Lab Project. __b1059__ has 3 computing nodes `qnode9031`-`qnode9033` with 192Gb of RAM and 40 cores each, and has 80 Tb of storage. b1059 storage is located at: `/projects/b1059/`. One can submit jobs to `--partition=b1059` to use this partition, with no limit on job duration. 

!!! Note
    Anyone who use quest should build your own project folder under `/projects/b1059/projects` with your name. You should only write and revise files under your project folder. You can read/copy data from __b1059__ but don't write any data out of your project folder.

### Running interactive jobs on Quest

If you are running a few simple commands or want to experiment with files directly you can start an interactive session on Quest. The command below will give you access to a node where you can run your commands

```
srun -A b1042 --partition=genomicsguestA -N 1 -n 24 --mem=64G --time=12:00:00 --pty bash -i 
```

!!! Important
    Do not run commands for big data on `quser21-24`. These are login nodes and are not meant for running heavy-load workflows.
   
### Using packages already installed on Quest

Quest has a collection of packages installed. 

`module avail` to see what packages are available on Quest.

`module load bcftools/1.10.1` to load packages (`module add` does the same thing). If you do `echo $PATH` before and after loading modules, you can see what module does is simply appending paths to the packages into your `$PATH` so the packages can be found in your environment.

`module purge` will remove all loaded packages and can be helpful to try when troubleshooting.
   
### Submitting jobs to Quest

Jobs on Quest are managed by [SLURM](https://slurm.schedmd.com/). While most of our pipelines are managed by Nextflow, it is useful to know how to submit jobs directly to SLURM. Below is a template to submit an array job to SLURM that will run 10 parallel jobs. One can run it with `sbatch script.sh`. If you digs into the Nextflow working directory, you can see Nextflow actually generate such scripts and submit them to SLURM on your behave.

```
#!/bin/bash
#SBATCH -J name                # Name of job
#SBATCH -A b1042               # Allocation
#SBATCH -p genomicsguestA      # Queue
#SBATCH -t 24:00:00            # Walltime/duration of the job
#SBATCH --cpus-per-task=1      # Number of cores (= processors = cpus) for each task
#SBATCH --mem-per-cpu=3G       # Memory per core needed for a job
#SBATCH --array=0-9            # number of parallel jobs to run counting from 0. Make sure to change this to match total number of files.

# make a list of all the files you want to process
# this script will run a parallel process for each file in this list
name_list=(*.bam)

# take the nth ($SGE_TASK_ID-th) file in name_list
# don't change this line
Input=${name_list[$SLURM_ARRAY_TASK_ID]} 

# then do your operation on this file
Output=`echo $Input | sed 's/.bam/.sam/'`
```

### Monitoring SLURM Jobs on Quest

`squeue -u <netid>` to check all jobs of a user.

`scancel <jobid>` to cancel a job. `scancel  {30000..32000}` to cancel a range of jobs (job id between 30000 and 32000).

`sinfo | grep b1059` to check status of nodes. `alloc` means all cores on a node are completed engaged; `mix` means some cores on a node are engaged; `idle` means all cores on a node are available to accept jobs.
