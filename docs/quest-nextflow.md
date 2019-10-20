[TOC]

# Installation

!!! Important
    If you haven't already, take a look at [Andersen-Lab-Env](pipeline-andersen-lab-env) for more information on how to setup your environment on Quest.

Nextflow can be installed with linuxbrew or homebrew. Use:

```
brew tap homebrew/science
brew install nextflow
```

# Quest cluster configuration

Configuration files allow you to define the way a pipeline is executed on Quest. 

[Read the quest documentation on configuration files](https://www.nextflow.io/docs/latest/config.html)

Configuration files are defined at a global level in `~/.nextflow/config` and on a per-pipeline basis within `<pipeline_directory>/nextflow.config`. Settings written in `<pipeline_directory>/nextflow.config` override settings written in `~/.nextflow/config`.

### Global Configuration: `~/.nextflow/config`

In order to use nextflow on quest you will need to define some global variables regarding the process. Our lab utilizies nodes and space dedicated to genomics projects. In order to access these resources your account will need to be granted access. Contact Quest and request access to the genomics nodes and project `b1042`. Once you have access you will need to modify your global configuration. Set your `~/.nextflow/config` file to be the following:

```
process {
    executor = 'slurm'
    queue = 'genomicsguestA'
    clusterOptions = '-A b1042 -t 24:00:00 -e errlog.txt'
}

workDir = "/projects/b1042/AndersenLab/work/<your folder>"
tmpDir = "/projects/b1042/AndersenLab/tmp"

```

This configuration file does the following:

* Sets the executor to `slurm` (which is what Quest uses)
* Sets the queue to `genomicsguestA` which submits jobs to genomics nodes. The `genomicsguestA` will submit jobs to our dedicated nodes first, which we have high priority. If our dedicated nodes are full, it will submit to other nodes we don't have priority. So far, our lab have 2 dedicated nodes, with 28 cores and related memory (close to 1:5) for each dedicated node. We will have more in the future.  
* `clusterOptions` - Sets the account to `b1042`; granting access to genomics-dedicated scratch space.
* `workDir` - Sets the working directory to scratch space on b1042. To better organization, Please build your own folder under `/projects/b1042/AndersenLab/work/`, and define it here.
* `tmpDir` - Creates a temporary working directory. This can be used within workflows when necessary.

# Screen

When jobs run for a very long time you should run them in screen. Screen lets you continue to run jobs in the background even if you get kicked off the cluster or log off.

* [Screen Tutorial](https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/)

Keep in mind that quest has several login nodes. We use `quser21-24`. Screen sessions only persist on __ONE__ of these login nodes. You can jump between nodes by simply typing ssh and the login node you want (_e.g._ `ssh quser 22`). 

# Resources

* [Nextflow documentation](https://www.nextflow.io/docs/latest/)
* [Awesome Nextflow pipeline examples](https://github.com/nextflow-io/awesome-nextflow) - Repository of great nextflow pipelines.
